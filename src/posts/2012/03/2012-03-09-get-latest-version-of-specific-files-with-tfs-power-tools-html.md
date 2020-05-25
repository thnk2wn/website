---
title: "Get Latest Version of Specific Files With TFS Power Tools"
date: "2012-03-09"
---

Sometimes it is handy to get the latest version of certain types of files in TFS source control. These files may reside across many folders in a large solution so it can be painful to either download a large root branch or cherry-pick through a number of folders. With the TFS Power Tools and a few lines of PowerShell it is a breeze to do.  
  

\[powershell\] if ( (Get-PSSnapin -Name Microsoft.TeamFoundation.PowerShell -ErrorAction SilentlyContinue) -eq $null ) { Add-PSSnapin Microsoft.TeamFoundation.PowerShell }

$env:path += ";$env:ProgramFiles\\Microsoft Visual Studio 10.0\\Common7\\IDE"

function GetLatestFiles(\[string\]$tfsPath, \[string\]$like) { foreach ($item in Get-TfsChildItem $tfsPath -r | where {$\_.ServerItem -like $like}) { tf.exe get $item.ServerItem /version:T -Force } } \[/powershell\]  

My need for this function today was in testing my CI MSBuild script where I kept messing with AssemblyInfo.cs and AssemblyInfo.vb files and needed to revert them back before running the build script again to test it:  
  

\[powershell\] GetLatestFiles "$/My Application/Code/Dev/" "\*AssemblyInfo.\*" \[/powershell\]
