---
title: "Trying Out Jobs in PowerShell"
date: "2012-07-04"
---

An older app in our workplace stack is a webforms website project and it has a large enough directory structure to take a while to compile the site. I have to run the site a fair amount for a rewrite effort and it changes enough to make the initial build and run painfully slow.

  
  
Since I have been working on a build related PowerShell module lately, I thought precompiling the "legacy" site might be a good candidate for a background job. I have not used jobs in PowerShell before and for whatever reason I was having a hard time finding good, complete examples of using them. There were also some things that tripped me up, so here is an example for the future version of me to reference some day.  

\[powershell\] param ( $TrunkPath = "D:\\Projects\\MyApp\\trunk" )

function Start-WebsitePrecompile { $logFile = (join-path $TrunkPath "build\\output\\logs\\BackgroundCompile.log") "Build log file is $logFile" $msg = @" Starting background compile of site. Use Get-Job to check progress. You may go on about your merry way but may want to leave the host open until complete. "@ Write-Output $msg $job = Start-Job -InputObject $TrunkPath -Name MyAppPageCompile -ScriptBlock { # doesn't appear transcription is supported here $trunk = $input

Set-Alias aspnetcompile $env:windir\\Microsoft.NET\\Framework\\v4.0.30319\\aspnet\_compiler.exe # see website solution file for these values $virtualPath = "/web" $physicalPath = (join-path $trunk "web") $compilePath = $trunk + "PrecompiledWebweb" aspnetcompile -v $virtualPath -p $physicalPath -f -errorstack $compilePath } # output details $job Register-ObjectEvent $job -MessageData $logFile -EventName StateChanged \` -SourceIdentifier Compile.JobStateChanged \` -Action { $logFile = $event.MessageData Set-Content -Force -Path $logFile \` -Value $(Receive-Job -Id $($Sender.Id) -Keep:$KeepJob | Out-String) #$eventSubscriber | UnregisterEvent Unregister-event -SourceIdentifier Compile.JobStateChanged $eventSubscriber.Action | Remove-Job Write-Host "Job # $($sender.Id) ($($sender.Name)) complete. Details at $logFile." } } \[/powershell\]  

### Some Notes

- Everything inside the job's script block will be executed in another PowerShell process; anything from outside the script block that needs to be used inside must be passed into the script block with the InputObject parameter ($input). While this might be obvious it does mean potential refactoring considerations.
- It didn't appear transcription was supported inside the script block which was disappointing.
- I half expected that Start-Job would provide a parameter for a block to call when the job was complete. Register-ObjectEvent works but it's a bit verbose and isn't even mentioned in many posts talking about job management.
- Like the job script block, the event handler action script block cannot refer to anything from the outside other than anything passed into the block with the MessageData parameter and automatic variables such as $event, $eventSubscriber, $sender, $sourceEventArgs, and $sourceArgs.
- I went through some trial and error in getting the output from the job in the completed event. The code on line 36 and 37 worked fine but it was not the most obvious initial syntax.
- There are a couple of ways to unregister events such as line 38, but I found that when I called the function again I received an error that the event was already subscribed to, so it was clear the unregistration was not working for some reason. The current code is working but similar code did not work previously. I dunno man, gremlins.
- Event handler cleanup strikes me as a bit odd and this [Register-TemporaryEvent](http://poshcode.org/2205) script is worth a look.

I am tempted to refactor more of this developer build module process to use more background jobs to do more in parallel. However it is a bit tricky in that many of the functions are called both individually and chained together in driver functions and they need to work both in "foreground" and "background" modes. It would also mean a loss of rich progress reporting and things get more difficult in terms of debugging, output, code sharing, etc. Also, while multiple cores may help with parallel work, there's a law of diminishing returns to be considered as well as machine performance while attempting to do other work while PowerShell crunches away.
