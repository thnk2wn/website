---
title: "Out of Office PowerShell Script, Part I"
date: "2010-08-20"
---

Sometimes the small things annoy me, and recently that was the overhead in the Out of Office (OOF) process at work. It's not overly bearing but enough to be annoying:

1. Create and route a record in our custom Leave Request System for approval
2. Update BMC Remedy System with planned PTO (i.e. notice for routed HelpDesk tickets)
3. Create a calendar appointment in Outlook
4. Set Outlook OOF / automatic replies
5. Send an email to the team letting them know of upcoming OOF status
6. Update Kronos web time, subtract from salary hours, add PTO hours
7. Set phone voicemail
8. Update other calendars if desired (SharePoint group calendars, personal calendars etc.)
9. and the list keeps going...

If one group talked to another at my company and the bigger picture was discussed I'm sure several of the above steps could be consolidated but until then...

### The constraints

The first problem is that I've had too much real work to do to spend much time on something like this. "Who has time? Who has time? But then if we do not ever take time, how can we ever have time?"

The next problem is that some systems I have no or limited access to from an API perspective, or accessing via API is cumbersome.

With this in mind I decided to first go after the low-hanging fruit and take care of blocking off the Outlook calendar, Outlook Out of Office / Automatic replies, and group email as they were easier, core pieces. PowerShell seemed a good fit as the last thing I wanted was another app to have to deal with.

### Settings, Config, Variables & Constants

The below variables control the behavior of the script. Some are used by multiple steps, others only for a given step. Generally most variables would be set only the first time and only the out of office start/end dates would change per use. At some point when the entire script is finished I might turn several of the variables into script parameters and use a calling script but to begin with this was more convenient.  

``` powershell
#*** VARIABLES / CONFIG SETTINGS **********************************************
# change this to be directory where script file is stored and will be run from
$myScriptPath  =  "C:\Scripts\Powershell\OutOfOffice"

# start and end times as text strings. used in calendar, auto replies, email etc. use date + time
$startTime     =  "08/03/2010 5:00 PM"    # format MM/dd/yyyy hh:mm AMPM
$endTime       =  "08/09/2010 8:00 AM"    # format MM/dd/yyyy hh:mm AMPM

# leave this be:
$startDt = [DateTime]::Parse($startTime)
$endDt = [DateTime]::Parse($endTime)

# calendar appointment subject and location
$apptCreate    =  $true
$apptSubject   =  "Out of Office"
$apptLocation  =  "Away"

# email address used for out of office auto reply and From for out of office email
$emailAddress  =  "my_email@mycompany.com"
$myName = "Geoff"

# internal and external messages for out of office automatic replies. Supports HTML
$autoReplySet  =  $true
$internalMsg   =  "I will be out of the office from &lt;font color='blue'&gt;" + $startDt.DayOfWeek.ToString() + " " + $startTime + "&lt;/font&gt; to &lt;font color='blue'&gt;" + $endDt.DayOfWeek.ToString() + " " + $endTime + "&lt;/font&gt;"
$externalMsg   =  $internalMsg

# this is who you want to email to notify ahead of time that you will be out of office
$emailSend     =  $true
# comma separate multiple addresses
$emailTo       =  "AppsTeam@mycompany.com, business_person@mycompany.com"
#$emailSubject =  [string]::Format("Out of office {0} - {1}", $startDt.ToShortDateString(), $endDt.ToShortDateString())
$emailSubject  =  "Out of office"
$emailBody     =  $internalMsg + ". Please see me before then if you need anything.&lt;br/&gt;&lt;br/&gt;Thanks,&lt;br/&gt;&lt;br/&gt;" + $myName + "&lt;br/&gt;&lt;br/&gt;Sent by OutOfOffice.ps1"

#*** CONSTANTS ****************************************************************
if (!(test-path variable:olFolderCalendar))
{
    New-Variable -Option constant -Name olFolderCalendar -Value 9
}

if (!(test-path variable:olAppointmentItem))
{
    New-Variable -Option constant -Name olAppointmentItem  -Value 1
}

if (!(test-path variable:olOutOfOffice))
{
    New-Variable -Option constant -Name olOutOfOffice  -Value 3
}
```

### Creating the Outlook Appointment

I started with this first as this was what I initially forgot last time and had to decline / propose new times for meetings others scheduled because my calendar did not reflect upcoming OOF time. Also, with the exception of email, this seemed to be the easiest thing to automate with some quick Outlook interop code. I only tested this with Outlook already open; really the code should follow [this MSDN pattern](http://msdn.microsoft.com/en-us/library/ff462097.aspx) more.

``` powershell
if ($apptCreate)
{
    $outlook = new-object -com Outlook.Application

    #*** CREATE CALENDAR APPT *****************************************************
    $calendar = $outlook.Session.GetDefaultFolder($olFolderCalendar)
    $appt = $calendar.Items.Add($olAppointmentItem)
    $appt.Start = $startDt
    $appt.End = $endDt
    $appt.Subject = $apptSubject
    $appt.Location = $apptLocation
    $appt.BusyStatus = $olOutOfOffice
    $appt.Save()
}
```

### Setting Outlook OOF / Automatic Replies

This step was a bit trickier to automate. There was nothing I could find in the Outlook object model or documentation that suggested I could set this. There were some Exchange PowerShell cmdlets but it appeared to me that they were more difficult to use client side and it appeared permission and configuration issues were likely. I looked at [Set-MailboxAutoReplyConfiguration](http://technet.microsoft.com/en-us/library/dd638217.aspx), [$outlook.Session..Stores\[1\].PropertyAccessor.SetProperty](http://social.msdn.microsoft.com/forums/en-US/vsto/thread/e935c4ec-0b3c-4230-9303-0dd69eb2ce2a), [$service.SetUserOofSettings](http://www.mikepfeiffer.net/2010/07/manage-exchange-2007-out-of-office-oof-settings-with-powershell-and-the-ews-managed-api/) in Microsoft.Exchange.WebServices.dll and more. In the end I went with this nice [EWSOofUtil.dll](http://telnetport25.wordpress.com/2008/03/16/quick-ish-tip-exchange-2007-setting-oof-for-users-via-powershell-2/).

``` powershell
if ($autoReplySet)
{
    # see http://telnetport25.wordpress.com/2008/03/16/quick-ish-tip-exchange-2007-setting-oof-for-users-via-powershell-2/
    $oofDll = [System.IO.Path]::Combine($myScriptPath, "EWSOofUtil.dll")
    [Reflection.Assembly]::LoadFile($oofDll)

    $oofutil = new-object EWSOofUtil.OofUtil

    # Enabled,Disable,Scheduled
    # there are a lot of overloads. looks like for what we want we need to call main one with all the params
    # SetOof(string EmailAddress, string OofStatus, string InternalMessage, string ExternalMessage, DateTime DurationStartTime, DateTime DurationEndTime,
    #        string InternalMessageLanguage, string ExternalMessageLanguage, string ExternalAudienceSetting, string UserName, string Password,string Domain,  string OofURL)
    $oofutil.setoof($emailAddress, "Scheduled", $internalMsg, $externalMsg, $startDt.ToUniversalTime(), $endDt.ToUniversalTime(), "", "", "", "", "", "", "")
}
```

### Sending the email notification

This step was a quick icing on the cake. There are several ways to send email from Powershell but I went with the first thing that came to mind that I knew would work.

``` powershell
if ($emailSend)
{
    $smtpServer = "companyserver.domain.com"
    $smtp = new-object Net.Mail.SmtpClient($smtpServer)
    $mailMsg = new-object Net.Mail.MailMessage($emailAddress, $emailTo)
    $mailMsg.Subject = $emailSubject
    $mailMsg.Body = $emailBody
    $mailMsg.IsBodyHtml = $true
    $smtp.Send($mailMsg)
    echo "Sent email OOF notification to " $emailTo
}
```

### Source

Download [OutOfOffice.zip](/wp-content/uploads/2017/06/OutOfOffice.zip) which includes OutOfOffice.ps1 and the dll it uses.

### Next Steps

- **Remedy** - I despise BMC Remedy so much (at least our setup of it) that I wrote a support application that handles the Incident management and Remedy->TFS integration. That app is already reading our Remedy Out of Office data. Updating is trivial but the trick is managing all the dependencies of the Remedy API, securely storing the credentials and so forth. It's really more of a deployment issue.

- **Other calendars** - We have a couple group SharePoint calendars and it would be nice to update those with the OOF time. I would assume this would be easy enough to do with SharePoint web services or cmdlets? For personal use it'd be nice to update my Google calendar, perhaps create a [CalDAV](http://en.wikipedia.org/wiki/CalDAV) file etc.

- **Leave Request System** - This is one of our custom, internal systems to officially request time off for longer PTO periods. It is Silverlight based and perhaps with a service reference this could be automated or the app changed to perform more of these steps

- **Phone Voicemail** - We use Nortel VOIP phones and have a telephony project coming up; perhaps this could be set via API with pre-recorded voice messages.

- **Kronos Web Time** - Pretty sure I won't be able to automate this as it is more of an HR thing and they'd have my head

Subscribe to this feed at: [http://feeds.feedburner.com/thnk2wn](http://feeds.feedburner.com/thnk2wn)
