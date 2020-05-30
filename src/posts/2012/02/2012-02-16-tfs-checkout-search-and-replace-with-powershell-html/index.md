---
title: "TFS Checkout, Search and Replace with PowerShell"
date: "2012-02-16"
---

Occasionally I need to make batch edits to files that are under TFS source control, outside of Visual Studio. Usually the need is editing some 37 project files to replace some path difference after branching and merging.  
  

Some options for the source control checkout:

- Manually checkout each file - Across 37 folders? No thanks.
- [tf.exe](http://msdn.microsoft.com/en-us/library/56f7w6be.aspx) at the command line - Not bad but I prefer PowerShell.
- Microsoft.TeamFoundation\*.dll assemblies in PowerShell - Powerful and flexible but tedious work.
- [Cmdlets in TFS Power Tools](http://visualstudiogallery.msdn.microsoft.com/c255a1e4-04ba-4f68-8f4e-cd473d6b971f) - Sounds like a winner.

On the search and replace front that can be done in a batch file, PowerShell, or various text editors and tools but it makes sense to stay in PowerShell.  
  

The below TFS.ps1 script is one simple approach to this task. It is not a foolproof script that handles all edge cases in the best possible manner, but it gets the job done for the simple replacements I usually need.  

\[powershell\] # This script requires the PowerShell Cmdlets in the TFS Power Tools # http://visualstudiogallery.msdn.microsoft.com/c255a1e4-04ba-4f68-8f4e-cd473d6b971f # # Open the PowerShell console under the Team Foundation Server Power Tools start menu # Or manually add the snapin with: Add-PSSnapin Microsoft.TeamFoundation.PowerShell # # Load this script => ". .\\Tfs.ps1" # # Example: # #> CheckOutSearchAndReplace "F:\\Projects\\MyApp\\Code\\Dev" "\*.\*proj" # "..\\\\..\\\\Dev-SomeBranch\\\\" "..\\..\\Dev\\" function CheckoutSearchAndReplace(\[string\] $path, \[string\] $include, \[string\] $match, \[string\] $replace) { $files = get-childitem -Path $path -include $include -recurse $index = 1 foreach ($file in $files) { $percentDone = (($index/$files.Count) \* 100) Write-Progress -activity "Checkout and Replace $path" -status $file.Name -percentComplete $percentDone Add-TfsPendingChange -Edit $file.FullName (Get-Content -Path $file.FullName -encoding UTF8) -replace $match, $replace | Set-Content $file.FullName -encoding UTF8 $index += 1 } "Processed {0} file(s)" -f ($index-1) } \[/powershell\]

Calling this script might look like this:  

F:\\Scripts
> . .\\Tfs.ps1
F:\\Scripts
> CheckOutSearchAndReplace "F:\\Projects\\MyApp\\Code\\Dev" "\*.\*proj" 
    "..\\\\..\\\\Dev-SomeBranch\\\\" "..\\..\\Dev\\"

Version CreationDa ChangeType      ServerItem
                te
------- ---------- ---------- ----------
  31334   1/1/0001 Edit            $/My App/Code...ppGenLibrary.csproj
  31334   1/1/0001 Edit            $/My App/Code...orms.Library.vbproj
  31334   1/1/0001 Edit            $/My App/Code...Works.AppGen.csproj
  31334   1/1/0001 Edit            $/My App/Code...redentialing.csproj
  31334   1/1/0001 Edit            $/My App/Code...aMaintenance.csproj
  31334   1/1/0001 Edit            $/My App/Code...yContracting.csproj
  31334   1/1/0001 Edit            $/My App/Code...ness.Payroll.csproj
  31334   1/1/0001 Edit            $/My App/Code...rContracting.csproj
  31334   1/1/0001 Edit            $/My App/Code...erEnrollment.csproj
  31334   1/1/0001 Edit            $/My App/Code...s.Recruiting.csproj
  31334   1/1/0001 Edit            $/My App/Code...s.Scheduling.csproj
  31334   1/1/0001 Edit            $/My App/Code...ess.Security.csproj
  31334   1/1/0001 Edit            $/My App/Code...iness.Shared.csproj
  31334   1/1/0001 Edit            $/My App/Code...siness.Tools.csproj
  31334   1/1/0001 Edit            $/My App/Code...Works.Client.vbproj
  31334   1/1/0001 Edit            $/My App/Code...orks.Console.csproj
  31334   1/1/0001 Edit            $/My App/Code...redentialing.csproj
  31334   1/1/0001 Edit            $/My App/Code...rks.Database.csproj
  31334   1/1/0001 Edit            $/My App/Code...aMaintenance.csproj
  31334   1/1/0001 Edit            $/My App/Code...yContracting.csproj
  31334   1/1/0001 Edit            $/My App/Code...s.Management.csproj
  31334   1/1/0001 Edit            $/My App/Code...orks.Payroll.csproj
  31334   1/1/0001 Edit            $/My App/Code...rContracting.csproj
  31334   1/1/0001 Edit            $/My App/Code...erEnrollment.csproj
  31334   1/1/0001 Edit            $/My App/Code...s.Recruiting.csproj
  31334   1/1/0001 Edit            $/My App/Code...s.Scheduling.csproj
  31334   1/1/0001 Edit            $/My App/Code...rks.Security.csproj
  31334   1/1/0001 Edit            $/My App/Code...orks.Service.csproj
  31334   1/1/0001 Edit            $/My App/Code...ceController.csproj
  31334   1/1/0001 Edit            $/My App/Code...Works.Shared.csproj
  31334   1/1/0001 Edit            $/My App/Code...s.Shared.WPF.csproj
  31334   1/1/0001 Edit            $/My App/Code...upport.Email.vbproj
  31334   1/1/0001 Edit            $/My App/Code...mServiceHost.csproj
  31334   1/1/0001 Edit            $/My App/Code...elecomShared.csproj
  31334   1/1/0001 Edit            $/My App/Code...mWorks.Tools.csproj
  31334   1/1/0001 Edit            $/My App/Code...grationTests.csproj
  31334   1/1/0001 Edit            $/My App/Code...ks.UnitTests.csproj
Processed 37 file(s)

  

The script could go further with checking in the changes but usually I want to review the differences, add a checkin comment, and associate the changes with a work item. It could also be written in a different manner that would only checkout the file if a match was found and a change was actually made. It is also worth noting there will be a small whitespace difference at the end of the files with this approach; `File.WriteAllText` could avoid that but in my case I usually do not care.
