---
title: "WP7 In-app Searching, Filtering Part II"
date: "2011-02-13"
---

In my [first post on WP7 in-app searching / filtering](/tech/2010/10/14/wp7-in-app-searching-filtering.html) I presented a quick, first pass solution to this task. [Justin Angel](http://twitter.com/#!/JustinAngel) left some good suggestions in his comments and I wanted to enhance this feature a bit now that I've finally had a little bit of time to return to Windows Phone development. This second pass includes enhancements such as:

- Replacing ugly search form buttons with a contextual application bar
- Animating search textbox visibility (fade-in / out)
- Small moves towards "framework-ifying" search in separate class library
- Customization of search application bar along with hide keyboard button
- Smoother search switching from keypress to timer search trigger
- Demo solution for download and hopefully reader contributions

  
The video to the right shows the search operation in action. The search works fine in landscape orientation as well as on my Samsung Focus phone, the video recording is just easier with the emulator.  
  

I am only going to touch on changes made to the 1st pass of this functionality, so refer to the [original post](/tech/2010/10/14/wp7-in-app-searching-filtering.html) for any other background information.  
  

<iframe src="http://player.vimeo.com/video/19883303?title=0&amp;byline=0&amp;portrait=0" width="255" height="498" frameborder="0"></iframe>

  

### Contextual App Bar

Removing the buttons beside the search textbox freed up space but more importantly it provides a much more polished look and is more consisent with Windows Phone usage patterns.  
  

In the search control there is now support for specifying what application bar icons / features to use when the search control is active (focused). I originally did this because I had [issues getting app bar icons working from within a class library](http://social.msdn.microsoft.com/Forums/en-US/windowsphone7series/thread/6d59a5c0-33ff-4384-a842-8c9159278283) and this allowed it to work with any images. In addition to the built-in commands of deleting, hiding the keyboard and closing search, a custom option allows executing an arbitrary command (not shown here).  
  

\[xml\] <fwCtl:SearchControl x:Name="uxSearchControl" SearchCommand="{Binding SearchCommand}" Margin="0,0,0,0" Visibility="Collapsed"> <fwCtl:SearchControl.AppBarItems> <fwCtl:SearchAppBarItem IconUri="/Images/appbar.delete.rest.png" Action="Delete" /> <fwCtl:SearchAppBarItem IconUri="/Images/MB\_0036\_keyboard.png" Action="Keyboard" /> <fwCtl:SearchAppBarItem IconUri="/Images/appbar.close.rest.png" Action="Close" /> </fwCtl:SearchControl.AppBarItems> </fwCtl:SearchControl> \[/xml\]

When the search control loads up, it sets up the actions and resolves the page the search control is on. This is used later to backup the original application bar that is temporarily replaced when the search control is active.  
\[csharp\] private void Init() { this.Loaded += SearchControl\_Loaded; this.Searcher = new Searcher(); \_actions = new Dictionary<SearchActions, Action<SearchAppBarItem>> { {SearchActions.Delete, (i) => Clear()}, {SearchActions.Close, (i) => Close()}, {SearchActions.Keyboard, (i) => ToggleKeyboard()}, {SearchActions.Custom, (i) => { if (null == i.Command || !i.Command.CanExecute(null)) return; i.Command.Execute(null); }} };

uxSearchTextBlock.GotFocus += (s, e) => {SetContextAppBar(); HasSearchFocus = true; }; uxSearchTextBlock.LostFocus += (s, e) => { RestoreAppBar(); HasSearchFocus = false; }; }

private PhoneApplicationPage Page { get; set; }

private void SearchControl\_Loaded(object sender, RoutedEventArgs e) { if (DesignerProperties.IsInDesignTool) return; LayoutRoot.Background = new SolidColorBrush(Colors.Transparent); this.Page = this.FindParent<PhoneApplicationPage>();

this.Opacity = 0; this.Visibility = Visibility.Collapsed; }

private Dictionary<SearchActions, Action<SearchAppBarItem>> \_actions; \[/csharp\]

The context app bar is setup once and the page application bar simply gets swapped out as focus enters and leaves the search textbox.  

\[csharp\] private void SetContextAppBar() { if (null != this.ContextAppBar) { this.Page.ApplicationBar = this.ContextAppBar; return; } this.OriginalAppBar = this.Page.ApplicationBar;

// might want a setting for menu support, auto-create menu items for buttons var bar = new ApplicationBar {IsVisible = true, IsMenuEnabled = false, Opacity = 1};

this.AppBarItems.ToList().ForEach(i => { var btnText = (i.Action != SearchActions.Custom ? i.Action.ToString() : i.Text); var btn = new ApplicationBarIconButton { IconUri = i.IconUri, Text = btnText, IsEnabled = true }; btn.Click += (s, e) => \_actions\[i.Action\](i); bar.Buttons.Add(btn); });

this.ContextAppBar = bar; this.Page.ApplicationBar = this.ContextAppBar; }

private IApplicationBar OriginalAppBar { get; set; } private IApplicationBar ContextAppBar { get; set; } \[/csharp\]  

I am not sure what the best way is to go about swapping out application bars. Performance might be slightly better just using one application bar and removing and adding buttons. I did not notice a real performance problem though I did notice a slightly longer transition with the initial switch then on subsequent show/hide search operations.  
  

### Animating visibility transition

I started off using VisualStateManager in the search control's XAML but in the end it was not buying me much. I also switched from a sliding to a fade in/out animation and I found the later to be rather common so I went with a UIElement extension as opposed to repeating in XAML.  

\[csharp\] public static void FadeInAndShow(this UIElement target, double seconds, Action afterComplete) { target.Opacity = 0; target.Visibility = Visibility.Visible; var da = new DoubleAnimation { From = 0.0, To = 1.0, Duration = TimeSpan.FromSeconds(seconds), AutoReverse = false };

Storyboard.SetTargetProperty(da, new PropertyPath("Opacity")); Storyboard.SetTarget(da, target);

var sb = new Storyboard(); sb.Children.Add(da);

if (null != afterComplete) { EventHandler eh = null; eh = (s, args) => { //sb.Stop(); sb.Completed -= eh; afterComplete(); };

sb.Completed += eh; }

sb.Begin(); } \[/csharp\]

On the fade-in an action is specified to automatically set focus into the search textbox to start typing once the animation has completed.  

\[csharp\] public void ShowHide() { if (this.Visibility == Visibility.Collapsed) Show(); else Close(); }

private void Show() { this.FadeInAndShow(.8, () => uxSearchTextBlock.Focus()); }

private void Close() { RestoreOriginalUI(); }

private void RestoreOriginalUI() { button.Focus(); // how else to hide keyboard? RestoreAppBar(); Clear(); this.FadeOutAndCollapse(.5); }

private void RestoreAppBar() { this.Page.ApplicationBar = this.OriginalAppBar; }

private void Clear() { this.Searcher.Clear(); } \[/csharp\]  

### Timer-based Search

Searching immediately upon a keypress / property changed event was a bit overkill and introduced some lag on larger lists. It now simply toggles an invalidated property and a timer periodically inspects that and searches when needed. The interval might be a good target for a configurable property.  

\[csharp\] public class Searcher : NotifyPropertyChangedBase { private DispatcherTimer \_timer;

private void EnsureTimerIsRunning() { if (null == \_timer) { \_timer = new DispatcherTimer() {Interval = TimeSpan.FromSeconds(.7)}; \_timer.Tick += (s, e) => { if (Invalidated) ExecuteSearch(); }; }

if (!\_timer.IsEnabled) \_timer.Start(); }

private void ExecuteSearch() { if (null != this.SearchCommand && this.SearchCommand.CanExecute(null)) { this.SearchCommand.Execute(this.SearchText); Invalidated = false; } }

private string \_searchText; public string SearchText { get { return \_searchText; } set { if (\_searchText != value) { EnsureTimerIsRunning(); \_searchText = value; Invalidated = true; OnPropertyChanged("SearchText"); } } }

private ICommand \_searchCommand; public ICommand SearchCommand { get { return \_searchCommand; } set { if (\_searchCommand != value) { \_searchCommand = value; OnPropertyChanged("SearchCommand"); } } }

private bool \_invalidated; public bool Invalidated { get { return \_invalidated; } set { if (\_invalidated != value) { \_invalidated = value; OnPropertyChanged("Invalidated"); } } }

// or Reset() public void Clear() { this.SearchText = string.Empty;

// i.e. value cleared, full list if (this.Invalidated) ExecuteSearch();

this.Invalidated = false;

if (null != \_timer && \_timer.IsEnabled) \_timer.Stop(); } } \[/csharp\]  

### Demo Solution and Final Thoughts

Feel free to download and improve upon the [demo solution](/wp-content/uploads/2017/06/WP7AppSearch.zip). It's from a stripped down, earlier version of my app and there is plenty of room for improvement. I have only begun to cleanup the 1st pass and I haven't exercised the searching / filtering functionality outside of the single use case in my current app. The search feature is the only functionality hooked up in the demo.  
  

Some things to consider for enhancing this:  

- Styling enhancements such as moving the search icon "inside the textbox"; this originally presented some oddities so I pulled it.
- Additional dependency properties to allow changing more behavior characteristics
- Watermark textbox; ignored here since I autofocus to search textbox
- More generalization / "frameworkification" of searching / filtering
- Looking at [AutoCompleteBox](http://blogs.msdn.com/b/devfish/archive/2010/11/16/autocompletebox-listpicker-longlistselector-pagetransitions-for-wp7-oh-my-november-2010-wp7-toolkit-release.aspx) perhaps; my current usage is more for filtering down a list then just for picking an item
- Performance improvements and general code cleanup
