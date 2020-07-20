---
title: "Batch Download SSRS Reports with PowerShell"
date: "2011-10-12"
---

Lately our continuous integration process has encountered some errors with automatic SSRS report deployments to our Test environment when reports are merged to the appropriate branch. Unfortunately when this fails the process makes no attempt to re-deploy the reports on the next build.  
  

Those CI failures left me with a need to mass download the Dev and Test report definition files from the report server and compare the results. I found no way to batch download reports from Report Manager so I turned to PowerShell. I figured someone had done this before so a quick web search brought me to [this post](http://www.sqlmusings.com/2011/03/28/how-to-download-all-your-ssrs-report-definitions-rdl-files-using-powershell/) from which I based the script below. I made changes to turn the script into a function, added progress reporting and other output, and allowed specifying what server path to use and where to download the reports to. The resulting function follows.  

\[powershell\] function DownloadReports($reportServerName = $(throw "reportServerName is required."), $baseDirectory = $(throw "baseDirectory is required."), $baseServerPath = "/") { $status = "Downloading reports from {0} to {1}" -f $reportServerName, $baseDirectory Write-Output $status Write-Progress -activity "Connecting" -status $status -percentComplete -1 \[void\]\[System.Reflection.Assembly\]::LoadWithPartialName("System.Xml.XmlDocument"); \[void\]\[System.Reflection.Assembly\]::LoadWithPartialName("System.IO"); $ReportServerUri = "http://{0}/ReportServer/ReportService2005.asmx" -f $reportServerName; $Proxy = New-WebServiceProxy -Uri $ReportServerUri -Namespace SSRS.ReportingService2005 -UseDefaultCredential ; #check out all members of $Proxy #$Proxy | Get-Member #http://msdn.microsoft.com/en-us/library/aa225878(v=SQL.80).aspx #second parameter means recursive $items = $Proxy.ListChildren($baseServerPath, $true) | \` select Type, Path, ID, Name | \` Where-Object {$\_.type -eq "Report"}; if(-not(Test-Path $baseDirectory)) { \[System.IO.Directory\]::CreateDirectory($baseDirectory) | out-null } $downloadedCount = 0 foreach($item in $items) { #need to figure out if it has a folder name $subfolderName = split-path $item.Path; $reportName = split-path $item.Path -Leaf; $fullSubfolderName = $baseDirectory + $subfolderName; $percentDone = (($downloadedCount/$items.Count) \* 100) Write-Progress -activity ("Downloading from {0}{1}" -f $reportServerName, $subFolderName) -status $reportName -percentComplete $percentDone if(-not(Test-Path $fullSubfolderName)) { #note this will create the full folder hierarchy \[System.IO.Directory\]::CreateDirectory($fullSubfolderName) | out-null } $rdlFile = New-Object System.Xml.XmlDocument; \[byte\[\]\] $reportDefinition = $null; $reportDefinition = $Proxy.GetReportDefinition($item.Path); #note here we're forcing the actual definition to be #stored as a byte array #if you take out the @() from the MemoryStream constructor, you'll #get an error \[System.IO.MemoryStream\] $memStream = New-Object System.IO.MemoryStream(@(,$reportDefinition)); $rdlFile.Load($memStream); $fullReportFileName = $fullSubfolderName + "\\" + $item.Name + ".rdl"; #Write-Host $fullReportFileName; $rdlFile.Save( $fullReportFileName); "Downloaded {0}.rdl from {1}{2} to {3} ({4:###}%)" -f $reportName, $reportServerName, $subfolderName, $fullSubfolderName, $percentDone $downloadedCount += 1 } "Downloaded {0} reports from {1}{2} to {3}" -f $downloadedCount, $reportServerName, $subfolderName, $fullSubfolderName } \[/powershell\]  

Now a wrapper to download reports from 2 locations and then launch Explorer to view the results:  

\[powershell\] function DownloadAppNameDevAndTestReports { $folderName = Get-Date -format "MMM-dd-yyyy-hhmm tt"; $folder = Join-Path $env:temp $folderName DownloadReports "report-server.domain.com" $folder "/DEV1/AppName" DownloadReports "report-server.domain.com" $folder "/TEST1/AppName" ii $folder } \[/powershell\]  

At that point I can easily use a tool like [SourceGear DiffMerge](http://www.sourcegear.com/diffmerge/) to compare the two folders for differences.