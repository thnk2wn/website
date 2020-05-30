---
title: "Uploading SSRS Reports with PowerShell"
date: "2011-10-13"
---

The existing SSRS report deployment process I have been working with uses SQL Server's [RS Utility](http://msdn.microsoft.com/en-us/library/ms162839.aspx) along with a custom app that creates RSS input files that rs.exe reads. Both applications are used by a PowerShell script that serves as the driver.  
  

In setting up a new machine at work I had neither tool installed initially and the dependencies started to bug me. Looking at the SSRS web service I noticed a [CreateReport](http://msdn.microsoft.com/en-us/library/aa225813(v=sql.80).aspx) method so I created the below function to deploy the reports using PowerShell alone:  

\[powershell\] function UploadReports ($reportServerName = $(throw "reportServerName is required."), $fromDirectory = $(throw "fromDirectory is required."), $serverPath = $(throw "serverPath is required.")) { Write-Output "Connecting to $reportServerName" $reportServerUri = "http://{0}/ReportServer/ReportService2005.asmx" -f $reportServerName $proxy = New-WebServiceProxy -Uri $reportServerUri -Namespace SSRS.ReportingService2005 -UseDefaultCredential Write-Output "Inspecting $fromDirectory" # coerce the return to be an array with the @ operator in case only one file $files = @(get-childitem $fromDirectory \*.rdl -rec|where-object {!($\_.psiscontainer)}) $uploadedCount = 0 foreach ($fileInfo in $files) { $file = \[System.IO.Path\]::GetFileNameWithoutExtension($fileInfo.FullName) $percentDone = (($uploadedCount/$files.Count) \* 100) Write-Progress -activity "Uploading to $reportServerName$serverPath" -status $file -percentComplete $percentDone Write-Output "%$percentDone : Uploading $file to $reportServerName$serverPath" $bytes = \[System.IO.File\]::ReadAllBytes($fileInfo.FullName) $warnings = $proxy.CreateReport($file, $serverPath, $true, $bytes, $null) if ($warnings) { foreach ($warn in $warnings) { Write-Warning $warn.Message } } $uploadedCount += 1 } } \[/powershell\]  

Calling the function is straightforward:  

\[powershell\] UploadReports "report-server.domain.com" "c:\\temp" "/TEST1/AppName" \[/powershell\]  

What I am wondering now is why use the RS Utility at all if it is that easy to deploy reports using just PowerShell and the SSRS web service? My understanding is that rs.exe is just a thin wrapper around the web service, no? What does RS add? Using this method seemed to produce the same results without the extra dependencies but maybe I am missing something?

### Updates

- 10/21/2011 - Updated script to coerce get-childitem result to be an array so this works if only uploading a single report
