---
title: "Quick NLog Tip: Getting the Log Filename"
date: "2012-01-25"
---

I have found NLog to be very easy to use but one thing that was not clear was how to retrieve a log's filename. Our unhandled exception process packages up different files into a zip file for support and I wanted to resolve the log filename to include it in the package.  
  

The below code is one way to retrieve the log filename:  
  

\[csharp\] using NLog; using NLog.Layouts; using NLog.Targets;

namespace MyApp.Diagnostics { public static class LogManagement { private const string DefaultTargetName = "allTarget";

public static string GetTargetFilename(string targetName) { var target = GetTarget<FileTarget>(targetName); if (null == target) return null;

var layout = target.FileName as SimpleLayout; if (null == layout) return null; // layout.Text provides the filename "template" // LogEventInfo is required; might make sense for a log line template but not really filename var filename = layout.Render(new LogEventInfo()).Replace(@"/", @""); return filename; }

public static string GetTargetFilename() { return GetTargetFilename(DefaultTargetName); }

private static T GetTarget<T>(string targetName) where T: Target { if (null == LogManager.Configuration) return null; var target = LogManager.Configuration.FindTargetByName(targetName) as T; return target; } } } \[/csharp\]

First the code resolves a given target by name. My first thought was that Target.Filename would be a string of the filename but it is a generic Layout object. In my case I'm using a SimpleLayout and I cast that just for inspection purposes but it is not strictly needed in this sample. From there the code calls the Render method of the layout to transform the "template filename" [renderers](http://nlog-project.org/wiki/Layout_renderers) (variables such as ${processid}) into the final result with the rendered values.

  
  
Unfortunately Render requires a LogEventInfo object which might make more sense if we were rendering the template for a given line of a log file but not so much for the filename. Passing null generates an object reference not set exception but passing in a new LogEventInfo object worked and still allowed proper variable substitution; at least it did in my case but it might depend on what renderers you use in your filename configuration. Finally the code replaces any forward slashes in the filename with backslashes.  
  

If you use a different type of target or layout the code would need adjusting. NLog's LogManager is sealed so you cannot inherit from it but you can wrap it or change the code to be use extension methods though that might increase the direct dependency on NLog.  
  

So given the below NLog.config, layout.Text is "${environment:LOCALAPPDATA}/MyApp/logs/MyApp-${processid}.log.txt" and that renders to something along the lines of "C:\\Users\\MyUsername\\AppData\\Local\\MyApp\\logs\\MyApp-9176.log.txt"

  
  

\[xml\] <?xml version="1.0" encoding="utf-8" ?> <nlog xmlns="http://www.nlog-project.org/schemas/NLog.xsd" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">

<!-- make sure to set 'Copy To Output Directory' option for this file --> <!-- go to http://nlog-project.org/wiki/Configuration\_file for more information -->

<targets async="false"> <target name="allTarget" xsi:type="File" deleteOldFileOnStartup="true" archiveEvery="Day" maxArchiveFiles="1" fileName="${environment:LOCALAPPDATA}/MyApp/logs/MyApp-${processid}.log.txt" layout="${time}|${threadid}|${level:uppercase=true}|${logger}|${message}" autoFlush="true" /> <target name="debugTarget" xsi:type="Debugger" layout="${time}|${level:uppercase=true}|${logger}|${message}" /> </targets>

<rules> <!-- levels: Debug, Error, Fatal, Info, Off, Trace, Warn --> <logger name="\*" minLevel="Debug" writeTo="debugTarget,allTarget" /> </rules> </nlog> \[/xml\]
