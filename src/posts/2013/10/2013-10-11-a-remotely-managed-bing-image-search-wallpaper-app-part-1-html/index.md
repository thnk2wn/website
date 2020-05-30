---
title: "A Remotely Managed Bing Image Search Wallpaper App - Part 1"
date: "2013-10-11"
---

[![](images/you-ve-been-biebered.jpg)](http://www.spidergroup.com/blog/2012/01/youve-been-biebered/)

At a previous job I made the mistake of leaving my computer unlocked on rare occasions and I ended up getting [Biebered](http://www.spidergroup.com/blog/2012/01/youve-been-biebered/). We were constantly trolling each other and playing pranks which kept things fun. Changing someone's wallpaper is pretty basic so I decided to take it up a notch for striking back. One day I had too much time on my hands and started a remotely controlled wallpaper changing app powered by online image searches.

### The Highlights

- Powered by [Bing Image Search API](http://go.microsoft.com/fwlink/?LinkID=272626&clcid=0x409) for variety
- Windowless Windows background app
- Remotely managed / configured / controlled
- Silently changes victim's desktop wallpaper at scheduled intervals
- Jobs scheduled with [Quartz.net](http://quartznet.sourceforge.net/)
- HTTPS remote logging using both [loggly.com](http://loggly.com/) and [loggr.net](http://loggr.net/)
- Encrypted configuration
- Release compiled code obfuscation
- Sets itself to run at Windows startup
- Allows explicitly specifying an image URL to be used as background
- Delete previously downloaded images by age according to remote config

### Initial Decisions

#### App Type

I was pretty sure I would not be able to set the desktop wallpaper if the app was a Windows service but that didn't stop me from trying it :). Yeah it didn't work. In the old days you could just set the service property "Allow service to interact with desktop" but security is tighter with modern operating systems and that's a good thing. Maybe there's still a way to make that work but even if there was it probably wouldn't be worth the hassle. It would set off security alarms, it requires more hassle to install/uninstall, and similar functionality can be achieved with a Windows app that doesn't show any UI.

#### Image Sources

I considered different options for where to get the wallpaper images from including a preselected collection bundled with the app, reading from a network share, reading a URL from a remote config file, and using Google Image Search or Bing Image Search. I started out with Google's Image Search API and quickly found that to be a dead end, at least for any free version that didn't involve web scraping. Bing's Image Search API provided a free, quality, easy to use and diverse source of images. Later I decided to compliment that with the ability to override random images from a search with a specific image URL, if so desired.

### App Lifecycle Skeleton

Rather than use any hidden form, the app spins up an [ApplicationContext](http://msdn.microsoft.com/en-us/library/system.windows.forms.applicationcontext.aspx) in the Main method of Program.cs with `Application.Run(new AppContext())`. On startup it first loads app settings from a remote source and then sets up a job schedule for work to be fired off later.

\[csharp\]internal class AppContext : ApplicationContext { private static readonly IAppLogger Logger = LoggerFactory.Create(); private readonly JobScheduler \_scheduler = new JobScheduler();

public AppContext() { AppSettings.Load().ContinueWith(AfterSettingsLoad); }

private void AfterSettingsLoad(Task<AppSettings> task) { if (AppSettings.Instance.Status == AppStatus.Disabled) { Application.Exit(); return; }

Logger.Info("Setting up scheduler"); RegisterAppForWindowsStartup(); \_scheduler.Setup(); ImageCleanup.Execute(); }

protected override void ExitThreadCore() { Logger.Info("In ExitThreadCore"); base.ExitThreadCore(); AppTeardown(); }

private void AppTeardown() { Logger.Info("Tearing down app"); if (null != \_scheduler) { Logger.Info("Disposing scheduler"); \_scheduler.Dispose(); }

ImageCleanup.Execute();

RegisterAppForWindowsStartup(); }

private static void RegisterAppForWindowsStartup() { if (!DebugMode.IsDebugging) WindowsStartup.Register(); } } \[/csharp\]

### Job Scheduling

Job scheduling is done with [Quartz.net](http://quartznet.sourceforge.net/) and JobScheduler starts it's scheduler. It then scans the app's assembly for any scheduler classes and instantiates each to setup the scheduling for the different jobs.

\[csharp\]internal class JobScheduler : DisposableObject { private static readonly IAppLogger Logger = LoggerFactory.Create();

public void Setup() { Logger.Info("Creating job scheduler"); ISchedulerFactory schedFact = new StdSchedulerFactory(); Scheduler = schedFact.GetScheduler(); Scheduler.Start();

var types = Assembly.GetExecutingAssembly().GetTypes() .Where(x => x.BaseType == typeof(ScheduleBase)).ToList(); Logger.Info("Found {0} schedules. Setting up each", types.Count); types.ForEach(t=> ((ScheduleBase) Activator.CreateInstance(t, Scheduler)).Setup()); Logger.Debug("Schedules setup"); }

private IScheduler Scheduler { get; set; } protected override void DisposeManagedResources() { if (null != this.Scheduler) this.Scheduler.Shutdown(waitForJobsToComplete: false); } } \[/csharp\]

There are jobs for refreshing remote app settings, downloading images from Bing Image Search and for changing the wallpaper. A sample job schedule builder for downloading images:

\[csharp\]internal class DownloadSchedule : ScheduleBase { private static readonly IAppLogger Logger = LoggerFactory.Create();

public DownloadSchedule(IScheduler scheduler) : base(scheduler) { }

public override void Setup() { Logger.Info("Setting up download images job");

if (!AppSettings.Instance.Search.Enabled) { Logger.Info("Search and download images isn't enabled; exiting"); return; }

var job = JobBuilder.Create<DownloadImagesJob>() .WithIdentity("downloadImages") .Build();

var trigger = TriggerBuilder.Create() .WithIdentity("downloadImagesTrigger") .StartAt(AdjustOffset(DateBuilder.EvenMinuteDateAfterNow())) .WithSimpleSchedule(x => x.WithIntervalInMinutes(AppSettings.Instance.Job.DownloadImagesIntervalMinutes) .RepeatForever()) .Build();

Scheduler.ScheduleJob(job, trigger); Logger.Info("Download job setup. Next fire time is {0}", GetNextFireTimeText(trigger)); } } \[/csharp\]

### Downloading Images

#### Download Job

In the remote app settings there's a search section that defines the phrase(s) to search for with Bing Image Search, along with supporting data such as options and API credentials. The download job enumerates each search to be run and sets up each to be executed. TaskDelayer is based on [this StackOverflow post](http://stackoverflow.com/questions/4229923/how-to-schedule-a-task-for-future-execution-in-task-parallel-library) and was added to prevent consuming too many resources at once in parallel or for too long continuously. If there is only one search to be run then it's a moot point.

\[csharp\]internal class DownloadImagesJob : IJob { //... public void Execute(IJobExecutionContext context) { try { if (!ShouldDownloadImages()) return;

var outPath = AppSettings.ImagePath.FullName; Ensure.That(outPath, "outputPath").IsNotNullOrWhiteSpace();

ImageCleanup.Execute(); Logger.Info("Downloading images"); if (AppSettings.Instance.Search.Queries.Count > 5) throw new InvalidOperationException("Please limit number of queries to 5");

for (var q = 0; q < AppSettings.Instance.Search.Queries.Count; q++) { var search = AppSettings.Instance.Search.Queries\[q\];

// try not to overwhelm system all at once, may draw too much attention var delaySeconds = q\*AppSettings.Instance.Search.DelaySecondsBetweenSearches; var q1 = q; Logger.Info("Starting batch {0} for term {1} w/delay seconds {2}", q1 + 1, search.Term, delaySeconds);

TaskDelayer.RunDelayed(delaySeconds \* 1024, () => { var fetcher = new SearchImageFetcher(outPath); Logger.Info("Fetching images for {0}", search.Term); fetcher.Fetch(search.Term, search.Options).Wait(); return fetcher; }).ContinueWith(t=> Logger.Info("Finished batch {0} for term {1} w/delay seconds {2}", q1 + 1, search.Term, delaySeconds)); } } catch (Exception ex) { Logger.Error(ex.ToString()); throw new JobExecutionException(ex); } } // ... } \[/csharp\]

#### Searching for Images

The search image fetcher class begins by initializing the bing search container, downloaded from the .NET Framework C# Service Proxy Class Library link inside the [Bing API Quick Start & Code](http://go.microsoft.com/fwlink/?LinkID=272626&clcid=0x409). The [Bing Data Search API](https://datamarket.azure.com/dataset/5ba839f1-12ce-4cce-bf57-a49d98d29a44#schema) allows 5,000 transactions per month for free; more than enough for my uses but keep that limit in mind.

The `Fetch` method takes any options specific to that search or the default if none were provided. It passes that to `RunSearches` to get the matching image URLs from Bing which are fed to `DownloadImages` for local download.

\[csharp\]internal class SearchImageFetcher { private static readonly IAppLogger Logger = LoggerFactory.Create(); private readonly BingSearchContainer \_bingSearchContainer; private readonly string \_outputPath;

public SearchImageFetcher(string outputPath) { \_outputPath = outputPath; Logger.Info("Creating Image Search client");

\_bingSearchContainer = new BingSearchContainer( new Uri(AppSettings.Instance.Search.ImageSearchUrl)) { IgnoreMissingProperties = true, Timeout = AppSettings.Instance.Search.Timeout, Credentials = new NetworkCredential(AppSettings.Instance.Search.Username, AppSettings.Instance.Search.ApiKey) }; }

public async Task Fetch(string searchTerm, SearchOptions options = null) { Logger.Info("Inspecting output directory {0}", \_outputPath); var dir = new DirectoryInfo(\_outputPath);

if (!dir.Exists) dir.Create(); try { var searchOptions = options ?? AppSettings.Instance.Search.DefaultOptions; var results = RunSearches(searchTerm, searchOptions);

Logger.Info("Image searches finished; {0} results", results.Count); await DownloadImages(results, searchTerm, searchOptions); } catch (Exception ex) { Logger.Error("Error fetching images for term '{0}' : {1}", searchTerm, ex.ToString()); throw; } } // ... } \[/csharp\]

The `RunSearches` method invokes the bing image search method multiple times for paging, which [required some customizations to BingSearchContainer.cs](http://stackoverflow.com/questions/11056628/add-paging-capabilites-to-dataservicequery-for-bing-search-api). I was unsure how to combine multiple filters from their documentation so I just used "Size:Large" in the settings.

\[csharp\]private List<ImageResult> RunSearches(string searchTerm, SearchOptions options, int take = 50) { var requests = options.Max/take; Logger.Info("Fetching images for term '{0}', options: {1}. Requests to make: {2}", searchTerm, options, requests); var results = new List<ImageResult>();

for (var i = 0; i < requests; i++) { var skip = i \* take; Logger.Info("Setting up search. Query: {0}, Options: {1}, Skip: {2}", searchTerm, options, skip); var query = \_bingSearchContainer.Image( Query: searchTerm, Options: null, Market: null, Adult: options.Adult, Latitude: null, Longitude: null, ImageFilters: options.Filters, //ImageFilters:"Size:Height:768&amp;Size:Width:1024", // how to combine multiple? top: 50, // 50 is the max we can request in one shot skip:skip); var currentResults = query.ToList(); results.AddRange(currentResults); }

return results; } \[/csharp\]

#### Downloading Search Results

`DownloadImages` post-filters the bing results to exclude images smaller than the size specified in settings/options. Again it'd be better to specify that when querying Bing but I didn't see how to combine filters at first glance. Images are saved locally with the same name returned from Bing and are not downloaded again if already there. The `MetadataManager` associates the Bing image URL with the local filename and persists that to disk for later remote logging use when the wallpaper is changed.

\[csharp\]private async Task DownloadImages(IEnumerable<ImageResult> results, string searchTerm, SearchOptions options) { var sw = Stopwatch.StartNew(); var filteredResults = results.Where(x => x.Width >= options.MinWidth &amp;&amp; x.Height >= options.MinHeight).ToList(); Logger.Info("Filtered result count: {0}", filteredResults.Count); var downloadCount = 0;

var metadataMgr = new MetadataManager(); foreach (var result in filteredResults) { var imageUrl = result.MediaUrl; var outputFilename = Path.Combine(\_outputPath, string.Format("{0}.jpg", result.ID.ToString("N")));

// don't redownload image if it already exists locally, tho' it could have changed if (!File.Exists(outputFilename)) { Logger.Debug("Image Url: {0}, destination: {1}", imageUrl, outputFilename); await ImageDownloader.DownloadImage(imageUrl, outputFilename); metadataMgr.Add(new Metadata { RemoteLocation = imageUrl, LocalLocation = outputFilename, Term = searchTerm }); downloadCount++; } else Logger.Debug("File already exists locally, not redownloading {0}", imageUrl); }

metadataMgr.Save(); sw.Stop(); Logger.Info("Saved {0} images in {1:000.0} seconds", downloadCount, sw.Elapsed.TotalSeconds); } \[/csharp\]

#### Downloading an Image

`ImageDownloader` takes care of downloading a single image using [WebClient](http://msdn.microsoft.com/en-us/library/system.net.webclient.aspx); I'd probably use [HttpClient](http://msdn.microsoft.com/en-us/library/system.net.http.httpclient.aspx) if I was writing this today. I looked into throttling image downloads using [SharpBits](http://sharpbits.codeplex.com/) to utilize idle network bandwidth with Microsoft's Background Intelligent Transfer Service. Ultimatiely that was too much fuss and took far too long to download. I also researched utilizing [this CodeProject ThrottledStream class](http://www.codeproject.com/Articles/18243/Bandwidth-throttling) but ultimiately I decided not to throttle downloads, mostly out of laziness.

\[csharp\]class ImageDownloader { private static readonly IAppLogger Logger = LoggerFactory.Create();

public static async Task DownloadImage(string url, string outputFilename) { using (var webClient = new WebClient()) { try { Logger.Debug("Downloading {0} to {1}", url, outputFilename); var sw = Stopwatch.StartNew(); var imageBytes = await webClient.DownloadDataTaskAsync(url); sw.Stop(); Logger.Debug("Downloaded {0} bytes in {1:00.0} second(s)", imageBytes.Length, sw.Elapsed.TotalSeconds);

Logger.Debug("Writing image bytes to disk"); var result = ImageWriter.Write(imageBytes, outputFilename); Logger.Debug("Image write complete with result: {0}", result); } catch (Exception ex) { Logger.Error("Error downloading image '{0}' to '{1}'. Likely corrupt " + "and will be deleted. Error: {2}", url, outputFilename, ex.ToString()); try { if (File.Exists(outputFilename)) File.Delete(outputFilename); } catch (Exception inner) { Logger.Error(string.Format("Error deleting image '{0}': {1}", outputFilename, inner)); } } } } } \[/csharp\]

#### Writing Image Bytes to Disk

I found that [Image.Save()](http://msdn.microsoft.com/en-us/library/system.drawing.image.save.aspx) threw an exception for some large images so I attempted that first and if that failed I wrote the image bytes to disk in chunks using a Stream.

\[csharp\]internal class ImageWriter { private static readonly IAppLogger Logger = LoggerFactory.Create();

public static bool Write(byte\[\] imageBytes, string outputFilename) { Logger.Debug("Creating memory stream from byte array of length {0}", imageBytes.Length); using (var stream = new MemoryStream(imageBytes)) { Logger.Debug("Creating image from stream"); using (var image = Image.FromStream(stream)) { Logger.Debug("Saving image {0} in Jpeg format", outputFilename);

try { image.Save(outputFilename, ImageFormat.Jpeg); Logger.Info("Saved image {0}", outputFilename); } catch (Exception ex) { Logger.Error("Error saving {0}. Will attempt saving in chunks. Error was : {1}", outputFilename, ex.ToString());

try { SaveImageInChunks(imageBytes, outputFilename); } catch (Exception inner) { Logger.Error("Saving image in chunks failed: {0}", inner); } } } } return true; }

private static void SaveImageInChunks(byte\[\] imageBytes, string outputFilename) { // this is to handle a large image where Image.Save croaked using (Stream source = new MemoryStream(imageBytes)) using (Stream dest = File.Create(outputFilename)) { var buffer = new byte\[1024\]; int bytes; while ((bytes = source.Read(buffer, 0, buffer.Length)) > 0) { dest.Write(buffer, 0, bytes); } } } } \[/csharp\]

### Part 2

[Part 2](/tech/2013/10/11/a-remotely-managed-bing-image-search-wallpaper-app-part-2.html) - remote app settings, setting the wallpaper, remote logging, source code and more...
