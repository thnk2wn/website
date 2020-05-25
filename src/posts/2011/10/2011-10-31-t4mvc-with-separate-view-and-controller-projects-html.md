---
title: "T4MVC with separate view and controller projects"
date: "2011-10-31"
---

### T4MVC

A friend of mine recently informed me of [T4MVC](http://mvccontrib.codeplex.com/wikipage?title=T4MVC_doc&referringTitle=T4MVC), a great little T4 template that generates some helper classes that allow for strongly referencing ASP.NET MVC controllers, actions, and views without those nasty hard-coded magic strings I detest. ReSharper helps validate those magic strings but I prefer to eliminate them entirely wherever possible.  
  

### The Issue

I tried T4MVC this morning and it worked for my controllers but not for my views. The problem was the template makes the assumption that your views and controllers reside in the same project. That works fine with the default, out of the box setup with a new ASP.NET MVC solution. However I went with [this solution organization from Jimmy Bogard](http://lostechies.com/jimmybogard/2009/12/09/organizing-asp-net-mvc-solutions/) where my web project only has content (views, css, javascript, images etc.) and all my managed code (including controllers) resides in my "Core" assembly (see [this post](http://derans.blogspot.com/2009/12/aspnet-mvc-solution-structure.html) for sample solution). I added the NuGet package reference in my Core assembly where it could generate code for my controllers but not for my views. I will not go into the rationale for this style of organization as Jimmy does a good job of explaining that. His post is a couple of years old but even today in ASP.NET MVC 3 I still find it relevant.  
  

### Changing T4MVC.tt

I first logged a [T4MVC issue](http://mvccontrib.codeplex.com/workitem/7171) requesting this support. Perhaps it will be added someday but I am not holding my breath. In the meantime, it was not overly difficult modifying the default template to support this separation. My workaround changes were quick and only lightly tested with my limited scenario however. In other words, your mileage may vary and this is not how I would implement this support in the official T4MVC project. That said, the code is below:  
  

Declarations section:  
\[csharp\] //<Custom> static Project ViewProject; //</Custom> \[/csharp\]  

PrepareDataToRender:  
\[csharp\] void PrepareDataToRender(TextTransformation tt) { //... // Get the path of the root folder of the app //<Custom> //AppRoot = Path.GetDirectoryName(Project.FullName) + '\\\\'; //</Custom> MvcVersion = GetMvcVersion(); // Use the proper return type of render helpers HtmlStringType = MvcVersion < 2 ? "string" : "MvcHtmlString"; // <Custom> //ProcessAreas(Project); ViewProject = GetViewsProject(Dte); AppRoot = Path.GetDirectoryName(ViewProject.FullName) + '\\\\'; ProcessAreas(ViewProject); // </Custom> } \[/csharp\]  

New functions to get a flat list of all projects (handling solution folders) and locating the Views/web project within:  
\[csharp\] //<Custom> Project GetViewsProject(DTE dte) { var projects = Projects(dte);

foreach (Project proj in projects) { if (proj.Name == ViewsProject) { return proj; } } return null; }

public static IList<Project> Projects(DTE dte) { Projects projects = dte.Solution.Projects; List<Project> list = new List<Project>(); var item = projects.GetEnumerator();

while (item.MoveNext()) { var project = item.Current as Project; if (project == null) { continue; }

if (project.Kind == ProjectKinds.vsProjectKindSolutionFolder) { list.AddRange(GetSolutionFolderProjects(project)); } else { list.Add(project); } }

return list; }

private static IEnumerable<Project> GetSolutionFolderProjects(Project solutionFolder) { List<Project> list = new List<Project>(); for (var i = 1; i <= solutionFolder.ProjectItems.Count; i++) { var subProject = solutionFolder.ProjectItems.Item(i).SubProject; if (subProject == null) { continue; }

// If this is another solution folder, do a recursive call, otherwise add if (subProject.Kind == ProjectKinds.vsProjectKindSolutionFolder) { list.AddRange(GetSolutionFolderProjects(subProject)); } else { list.Add(subProject); } }

return list; } //</Custom> \[/csharp\]  

ProcessAreas:  
\[csharp\] void ProcessAreas(Project project) { // Process the default area // <Custom> ProcessDefaultArea(); //</Custom> //... } \[/csharp\]  

Modified clone of ProcessArea:  
\[csharp\] //<Custom> void ProcessDefaultArea() { string name = null; var area = new AreaInfo() { Name = name }; var viewItems = ViewProject.ProjectItems; var controllerItems = Project.ProjectItems; ProcessAreaControllers(controllerItems, area); ProcessAreaViews(viewItems, area); Areas.Add(area);

if (String.IsNullOrEmpty(name)) DefaultArea = area; } //</Custom> \[/csharp\]  

Per a reader's comment I changed static file processing to use the view project:  

\[csharp\] namespace <#=LinksNamespace #> { <# foreach (string folder in StaticFilesFolders) { ProcessStaticFiles(ViewProject, folder); } #> } \[/csharp\]  

Finally in **T4MVC.tt.settings.t4** a new constant for the web project containing the views:  
\[csharp\] // <Custom> const string ViewsProject = "MyWebProjectName"; // </Custom> \[/csharp\]  

Download: [T4MVC.tt](https://geoffhudik.com/wp-content/uploads/2017/06/T4MVC.tt_.txt) | [T4MVC.tt.settings.t4](https://geoffhudik.com/wp-content/uploads/2017/06/T4MVC.tt_.settings.t4.txt)  
  

### Wrapup

With this in place I can enjoy the benefits of eliminating the magic strings for controller, action, and view names while also being able to have my views and controllers in separate projects. One downside is this could become a maintenance burden should T4MVC.tt change in a future release without this support baked in.  
  

### Updates

1. 06/06/2012 - Changed resolution of the view project to handle solution folders; initial version just enumerated top level projects and didn't handle subprojects. Also changed static file processing to use ViewProject.
