---
title: "Install a Windows Service Remotely with PowerShell"
date: "2012-03-22"
---

### The Objective

Our previous application deployment process worked in a pull manner, meaning the deployment package was pulled from another computer (such as a build server) to the target server, and the install was performed locally on the target machine. It is usually easier to make changes locally on a box than remotely but this had some drawbacks I did not like.  
  

First, for continuous deployment scenarios, it made more sense to me to deploy in a push manner. Previously we had a scheduled task on the target app servers that would periodically install our latest application bits, but it had no clue whether it was reinstalling the same build or not, or if there might be a problem with the build it is trying to deploy. Additionally it is more convenient to be able to deploy an app from any given location to any other location. With that in mind I set out to change our deployment process to a push model.  
  

### The Problem

Going into this I knew that we would no longer be able to directly use [installutil](http://msdn.microsoft.com/en-us/library/50614e95(v=vs.80).aspx) to install our Windows Service on our app server as that only works locally. What I was not sure of was what to replace that with. My deployment script is in PowerShell and I was not finding much Googling a solution for remotely installing services with PowerShell or otherwise.  
  

### Some Solutions

#### WMI

I did come across [this Fervent Coder post](http://geekswithblogs.net/robz/archive/2008/09/21/how-to-programmatically-install-a-windows-service-.net-on-a.aspx) on tackling this problem using WMI and C# and I started to base my solution on this. However WMI felt a bit low-level, raw, and dirty to me and initially I did not want to create a C# DLL for this or translate that code into equivalent PowerShell script.

#### PS Remoting

The PowerShell ninja [@beefarino](https://twitter.com/#!/beefarino) suggested PS Remoting. That might be a valid option to explore but I was a little leery of it; maybe if I had Jim's skills and was feeling more daring. [PSExec](http://technet.microsoft.com/en-us/sysinternals/bb897553) is another similar option but probably less desirable.  
  

![](images/PSRemoting.jpg)  

#### SC.exe

[SC.exe](http://support.microsoft.com/kb/251192) in the Windows Resource Kit is an option. I did not really evaluate this much as I did not run across it until late in the process.

#### Other

There are likely external tools that could be used or hackery like remote registry changes.

#### Pick Something Quick

Somehow things fell such that I did not start this until the day before a new release was due for testing. With little time to evaluate options I went the PowerShell WMI approach. Despite the cons mentioned it is fairly lightweight, allowed complete control, and worked reliably in my testing.  
  

With a basis on the [Fervent Coder WMI post](http://geekswithblogs.net/robz/archive/2008/09/21/how-to-programmatically-install-a-windows-service-.net-on-a.aspx) I began to come up with a PowerShell version of that C# implementation. Loading a .net 4 dll in PowerShell [was a bit of a pain](http://stackoverflow.com/questions/2094694/how-can-i-run-powershell-with-the-net-4-runtime) though [@beefarino](https://twitter.com/#!/beefarino) had a [solution for that too](http://poshcode.org/3294).  
  

### Enough, On With the Code Already

So I'm no PowerShell expert and this was done under the gun but...  
  

First a function to get a handle to the desired remote service.

\[powershell\] function Get-Service( \[string\]$serviceName = $(throw "serviceName is required"), \[string\]$targetServer = $(throw "targetServer is required")) { $service = Get-WmiObject -Namespace "root\\cimv2" -Class "Win32\_Service" \` -ComputerName $targetServer -Filter "Name='$serviceName'" -Impersonation 3 return $service } \[/powershell\]

An Impersonation value of 3 indicates "Impersonate (Allows objects to use the credentials of the caller)." I believe if not specified the default value is read from the registry. In my case the default did not appear to be Impersonate.  
  

Next a function to stop and uninstall the service if it exists already.

\[powershell\] function Uninstall-Service( \[string\]$serviceName = $(throw "serviceName is required"), \[string\]$targetServer = $(throw "targetServer is required")) { $service = Get-Service $serviceName $targetServer if (!($service)) { Write-Warning "Failed to find service $serviceName on $targetServer. Nothing to uninstall." return } "Found service $serviceName on $targetServer; checking status" if ($service.Started) { "Stopping service $serviceName on $targetServer" #could also use Set-Service, net stop, SC, psservice, psexec etc. $result = $service.StopService() Test-ServiceResult -operation "Stop service $serviceName on $targetServer" -result $result } "Attempting to uninstall service $serviceName on $targetServer" $result = $service.Delete() Test-ServiceResult -operation "Delete service $serviceName on $targetServer" -result $result } \[/powershell\]

Test-ServiceResult inspects the return code of a service operation; if it represents an error it maps the code to an error message and writes an error or warning.

\[powershell\] function Test-ServiceResult( \[string\]$operation = $(throw "operation is required"), \[object\]$result = $(throw "result is required"), \[switch\]$continueOnError = $false) { $retVal = -1 if ($result.GetType().Name -eq "UInt32") { $retVal = $result } else {$retVal = $result.ReturnValue} if ($retVal -eq 0) {return} $errorcode = 'Success,Not Supported,Access Denied,Dependent Services Running,Invalid Service Control' $errorcode += ',Service Cannot Accept Control, Service Not Active, Service Request Timeout' $errorcode += ',Unknown Failure, Path Not Found, Service Already Running, Service Database Locked' $errorcode += ',Service Dependency Deleted, Service Dependency Failure, Service Disabled' $errorcode += ',Service Logon Failure, Service Marked for Deletion, Service No Thread' $errorcode += ',Status Circular Dependency, Status Duplicate Name, Status Invalid Name' $errorcode += ',Status Invalid Parameter, Status Invalid Service Account, Status Service Exists' $errorcode += ',Service Already Paused' $desc = $errorcode.Split(',')\[$retVal\] $msg = ("{0} failed with code {1}:{2}" -f $operation, $retVal, $desc) if (!$continueOnError) { Write-Error $msg } else { Write-Warning $msg } } \[/powershell\]

The code to install a service remotely took more trial and error. The below worked for my needs but there are some hard-coded values that may be desirable to change depending upon the service and need.

\[powershell\] function Install-Service( \[string\]$serviceName = $(throw "serviceName is required"), \[string\]$targetServer = $(throw "targetServer is required"), \[string\]$displayName = $(throw "displayName is required"), \[string\]$physicalPath = $(throw "physicalPath is required"), \[string\]$userName = $(throw "userName is required"), \[string\]$password = "", \[string\]$startMode = "Automatic", \[string\]$description = "", \[bool\]$interactWithDesktop = $false ) { # can't use installutil; only for installing services locally #\[wmiclass\]"Win32\_Service" | Get-Member -memberType Method | format-list -property:\* #\[wmiclass\]"Win32\_Service"::Create( ... ) # todo: cleanup this section $serviceType = 16 # OwnProcess $serviceErrorControl = 1 # UserNotified $loadOrderGroup = $null $loadOrderGroupDepend = $null $dependencies = $null # description? $params = \` $serviceName, \` $displayName, \` $physicalPath, \` $serviceType, \` $serviceErrorControl, \` $startMode, \` $interactWithDesktop, \` $userName, \` $password, \` $loadOrderGroup, \` $loadOrderGroupDepend, \` $dependencies \` $scope = new-object System.Management.ManagementScope("\\\\$targetServer\\root\\cimv2", \` (new-object System.Management.ConnectionOptions)) "Connecting to $targetServer" $scope.Connect() $mgt = new-object System.Management.ManagementClass($scope, \` (new-object System.Management.ManagementPath("Win32\_Service")), \` (new-object System.Management.ObjectGetOptions)) $op = "service $serviceName ($physicalPath) on $targetServer" "Installing $op" $result = $mgt.InvokeMethod("Create", $params) Test-ServiceResult -operation "Install $op" -result $result "Installed $op" "Setting $serviceName description to '$description'" Set-Service -ComputerName $targetServer -Name $serviceName -Description $description "Service install complete" } \[/powershell\]

Starting the service could be done a variety of ways including net start, Set-Service and other means but since I was already in WMI land...

\[powershell\] function Start-Service( \[string\]$serviceName = $(throw "serviceName is required"), \[string\]$targetServer = $(throw "targetServer is required")) { "Getting service $serviceName on server $targetServer" $service = Get-Service $serviceName $targetServer if (!($service.Started)) { "Starting service $serviceName on server $targetServer" $result = $service.StartService() Test-ServiceResult -operation "Starting service $serviceName on $targetServer" -result $result } } \[/powershell\]

An example of calling these functions together might look like the below. The various helper functions this script calls are of little import; Set-Activity and Write-Log deal with writing to both output and progress and Copy-Files captures and writes copy output and optionally makes additional attempts on error.

\[powershell\] function Publish-Service { Set-Activity "Deploying Service files" $serviceName = "MyAppDataService" Write-Log "Stopping, uninstalling service $serviceName on $\_targetServer" Uninstall-Service $serviceName $\_targetServer "Pausing to avoid potential temporary access denied" Start-Sleep -s 5 # Yeah I know, don't beat me up over this Remove-RootFilesInDir $\_targetServicesDir Copy-Files "$\_packagePath\\Services\\\*\*" $\_targetServicesDir -recurse Install-Service \` -ServiceName $serviceName \` -TargetServer $\_targetServer \` -DisplayName "MyApp Data Service" \` -PhysicalPath "D:\\Apps\\MyApp\\Services\\MyApp.DataService.exe" \` -Username "NT AUTHORITY\\NetworkService" \` -Description "Provides remote TCP/IP communication between the MyApp client application and the database tier." Start-Service $serviceName $\_targetServer } \[/powershell\]
