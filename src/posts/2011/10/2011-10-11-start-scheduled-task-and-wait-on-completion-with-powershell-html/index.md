---
title: "Start Scheduled Task and Wait On Completion with PowerShell"
date: "2011-10-11"
---

Some of our application servers have scheduled tasks to deploy a staged build of our application to the current app server. In the past when I wanted to manually run the task I would do one of the following:

- Remote desktop into the app server and run the task from Scheduled Tasks
- Use Computer Management on my local machine to connect to the remote computer and run scheduled task from there
- Use a command prompt or PowerShell console with [schtasks](http://msdn.microsoft.com/en-us/library/windows/desktop/bb736357(v=vs.85).aspx)

  

The first two GUI options took too long and the last worked but I could never remember the syntax and it was still a bit manual. So I decided I would take a quick stab at a PowerShell script that would start a remote task and wait on it to complete.  
  

The below function is my first pass attempt. I am sure you PowerShell experts can tell me a better way to do this and in 75% less code but for now this works though it is not foolproof.  
  

\[powershell\] function WaitOnScheduledTask($server = $(throw "Server is required."), $task = $(throw "Task is required."), $maxSeconds = 300) { $startTime = get-date $initialDelay = 3 $intervalDelay = 5 Write-Output "Starting task '$task' on '$server'. Please wait..." schtasks /run /s $server /TN $task

\# wait a tick before checking the first time, otherwise it may still be at ready, never transitioned to running Write-Output "One moment..." start-sleep -s $initialDelay $timeout = $false

while ($true) { $ts = New-TimeSpan $startTime $(get-date) # this whole csv thing is hacky but one workaround I found for server 2003 $tempFile = Join-Path $env:temp "SchTasksTemp.csv" schtasks /Query /FO CSV /s $server /TN $task /v > $tempFile

$taskData = Import-Csv $tempFile $status = $taskData.Status if($status.tostring() -eq "Running") { $status = ((get-date).ToString("hh:MM:ss tt") + " Still running '$task' on '$server'...") Write-Progress -activity $task -status $status -percentComplete -1 #-currentOperation "Waiting for completion status" Write-Output $status } else { break }

start-sleep -s $intervalDelay if ($ts.TotalSeconds -gt $maxSeconds) { $timeout = $true Write-Output "Taking longer than max wait time of $maxSeconds seconds, giving up all hope. Task execution continues but I'm peacing out." break } }

if (-not $timeout) { $ts = New-TimeSpan $startTime $(get-date) "Scheduled task '{0}' on '{1}' complete in {2:###} seconds" -f $task, $server, $ts.TotalSeconds } } \[/powershell\]  

With the function in place a couple convenience caller functions save a little typing:  

\[powershell\] function DeployAppToDev { WaitOnScheduledTask "dev-server" "Copy Latest Build" 120 }

function DeployAppToTest { WaitOnScheduledTask "test-server" "Deploy Main Codeline" 120 } \[/powershell\]  

Incorporating the code in my [PowerShell profile](http://technet.microsoft.com/en-us/library/ee692764.aspx), creating a [SlickRun](http://www.fiddler2.com/SlickRun/) keyword and Windows shortcuts further eased the convenience of invoking the script code.  
  

Sample output follows:  

H:\\> DeployAppToDev
Starting task 'Copy Latest Build' on 'dev-server'. Please wait...
SUCCESS: Attempted to run the scheduled task "Copy Latest Build".
One moment...
09:10:42 PM Still running 'Copy Latest Build' on 'dev-server'...
09:10:48 PM Still running 'Copy Latest Build' on 'dev-server'...
09:10:53 PM Still running 'Copy Latest Build' on 'dev-server'...
09:10:58 PM Still running 'Copy Latest Build' on 'dev-server'...
09:10:03 PM Still running 'Copy Latest Build' on 'dev-server'...
09:10:09 PM Still running 'Copy Latest Build' on 'dev-server'...
09:10:14 PM Still running 'Copy Latest Build' on 'dev-server'...
09:10:19 PM Still running 'Copy Latest Build' on 'dev-server'...
09:10:24 PM Still running 'Copy Latest Build' on 'dev-server'...
09:10:29 PM Still running 'Copy Latest Build' on 'dev-server'...
09:10:35 PM Still running 'Copy Latest Build' on 'dev-server'...
Scheduled task 'Copy Latest Build' on 'dev-server' complete in 61 seconds

  

There are various other options available for this kind of thing such as [using the Task Scheduler API](http://myitforum.com/cs2/blogs/yli628/archive/2008/07/28/powershell-script-to-retrieve-scheduled-tasks-on-a-remote-machine-task-scheduler-api.aspx).
