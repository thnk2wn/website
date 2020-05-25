---
title: "System.Threading.Tasks and updating the UI thread"
date: "2011-08-15"
---

This is something that I couldn't remember parts of earlier today so I figured I would add it to my "knowledge base". I wanted to retrieve a TFS work item on a background thread in my ViewModel and have the UI update when complete. Specifically I wanted to retrieve the title of a TFS work item (project here) and have that display after the user entered a work item id. While this is not a terribly expensive server hit, it can be noticeable enough to want it performed on another thread.  
  

In the end the method associated with my RelayCommand looked like the below:  

\[csharp\] private void CheckWorkItemId() { if (!this.WorkItemIdInvalidated || this.ProjectId <= 0 || string.IsNullOrWhiteSpace(Settings.Default.TfsServerName)) return; this.WorkItemTitle = WORK\_ITEM\_FETCH; Task.Factory.StartNew(() => WorkItemInfoRetriever.Get( Settings.Default.TfsServerName, this.ProjectId)).ContinueWith(t => { if (null == t.Exception) { this.WorkItemTitle = t.Result.Title; this.WorkItemIdInvalidated = false; } else { this.WorkItemTitle = null; var ex = t.Exception.InnerException; if (ex is InvalidWorkItemException) \_messageBoxService.ShowOKDispatch(ex.Message, "Invalid Work Item"); else \_messageBoxService.ShowErrorDispatch(ex.Message, "Unexpected Error"); }

ReCalculateCommands(); }, TaskScheduler.FromCurrentSynchronizationContext()); } \[/csharp\]

  

The salient points are:

- Grabbing the current task scheduler while on the UI thread with [TaskScheduler.FromCurrentSynchronizationContext](http://msdn.microsoft.com/en-us/library/system.threading.tasks.taskscheduler.fromcurrentsynchronizationcontext.aspx), done in the ContinueWith call

- Chaining in a continuation to be run when the task completes via [Task.ContinueWith](http://msdn.microsoft.com/en-us/library/dd270696.aspx)

- Checking for exception when completed. I bypassed explicit use of AggregateException and InnerExceptions as in my case there'd be a single error or none

  

ReCalculateCommands calls CanExecuteChanged on different commands that will re-enable them once the work item has been validated and retrieved. This code ensures that a cross-thread exception is not generated when accessing the commands created on the UI thread and bound to in the view.  
  

Updates:  

- 08/16/2011 - Replaced use of ContinueWithAll to ContinueWith off task created and other refactoring and simplification that resulted from it.

  

My experience using Tasks is pretty minimal at this point so perhaps there are more improvements to come in this area.
