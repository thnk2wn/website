---
title: "Another WP7 Navigation Approach with MVVM"
date: "2010-10-10"
---

There are various ways to navigate between views in a Windows Phone 7 application. The first example you typically see is an event wired into a View's code-behind that calls _NavigationService.Navigate(uri)_. How do we navigate using an MVVM pattern though? There are various ways to do that both manually and using an existing MVVM framework, both simple and complex ones. Here I show one simple option where I use MVVM Light's messaging combined with some base ViewModel and Page functionality.  
  

Some feel that navigation should be a view-only concern and I can relate to that in ways. I don't feel that a ViewModel should be calling NavigationService directly but using a more loosely coupled messaging system to accomplish the same thing is fine by me.

### The Calling View

Here is a simple states view where the user selects a state and we want to load the next view passing in the state abbreviation and the state name. The important piece is use of MVVM Light's EventToCommand on the ListBox that will execute the ViewModel's StateSelectedCommand.  

\[xml highlight="10,11,43,44,45,46,47"\] <Views:PhoneApplicationPageBase x:Class="RiverBuddy.Views.StatesView" xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation" xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml" xmlns:shell="clr-namespace:Microsoft.Phone.Shell;assembly=Microsoft.Phone" xmlns:d="http://schemas.microsoft.com/expression/blend/2008" xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006" xmlns:Views="clr-namespace:RiverBuddy.Views" FontFamily="{StaticResource PhoneFontFamilyNormal}" xmlns:VML="clr-namespace:RiverBuddy.ViewModels.Locators" xmlns:i="clr-namespace:System.Windows.Interactivity;assembly=System.Windows.Interactivity" xmlns:cmd="clr-namespace:GalaSoft.MvvmLight.Command;assembly=GalaSoft.MvvmLight.Extras.WP7" FontSize="{StaticResource PhoneFontSizeNormal}" Foreground="{StaticResource PhoneForegroundBrush}" SupportedOrientations="PortraitOrLandscape" Orientation="Portrait" mc:Ignorable="d" d:DesignWidth="480" d:DesignHeight="696" d:DataContext="{Binding Source={StaticResource viewModelLocator}, Path=ViewModel}" shell:SystemTray.IsVisible="True"> <Views:PhoneApplicationPageBase.Resources> <VML:StatesViewModelLocator x:Key="viewModelLocator" /> </Views:PhoneApplicationPageBase.Resources>

<!--Data context is set to sample data above and first item in sample data collection below and LayoutRoot contains the root grid where all other page content is placed--> <Grid x:Name="LayoutRoot" Background="Transparent" d:DataContext="{Binding Items\[0\]}" HorizontalAlignment="Stretch"> <Grid.RowDefinitions> <RowDefinition Height="Auto"/> <RowDefinition Height="\*"/> </Grid.RowDefinitions>

<!--TitleGrid is the name of the application and page title--> <StackPanel x:Name="TitlePanel" Grid.Row="0" Margin="24,24,0,12"> <TextBlock x:Name="ApplicationTitle" Text="River Buddy" Margin="-10,0" Style="{StaticResource PhoneTextNormalStyle}"/> <TextBlock x:Name="ListTitle" Text="River Location" Margin="-10,0" Style="{StaticResource PhoneTextTitle1Style}"/> </StackPanel>

<!--ContentPanel contains details text. Place additional content here--> <Grid x:Name="ContentPanel" Grid.Row="1" HorizontalAlignment="Stretch"> <ListBox x:Name="MainListBox" Margin="0,-5,-12,0" HorizontalAlignment="Stretch" ItemsSource="{Binding States}" SelectionChanged="MainListBox\_SelectionChanged"> <ListBox.DataContext> <Binding Source="{StaticResource viewModelLocator}" Path="ViewModel"/> </ListBox.DataContext>

<i:Interaction.Triggers> <i:EventTrigger EventName="SelectionChanged" > <cmd:EventToCommand Command="{Binding StateSelectedCommand}" PassEventArgsToCommand="True"/> </i:EventTrigger> </i:Interaction.Triggers> <ListBox.ItemTemplate> <DataTemplate> <StackPanel x:Name="DataTemplateStackPanel" Margin="12,10,0,0" Orientation="Horizontal" HorizontalAlignment="Stretch" > <!-- <Image x:Name="ItemImage" Source="/RiverBuddy;component/Images/ArrowImg.png" Height="32" Width="32" VerticalAlignment="Top" Margin="10,0,20,0"/> --> <StackPanel Orientation="Horizontal"> <TextBlock x:Name="ItemText" Text="{Binding FullStateName}" HorizontalAlignment="Left" TextAlignment="Left" Margin="0,0,0,-3" Style="{StaticResource PhoneTextLargeStyle}"/> </StackPanel> </StackPanel> </DataTemplate> </ListBox.ItemTemplate> </ListBox> </Grid> </Grid> </Views:PhoneApplicationPageBase> \[/xml\]

### Calling ViewModel

Here the selected item is cast to the appropriate item type and a dictionary is built with the parameters key/value pairs that need to be passed to the view being called. The code then calls a SendNavigationMessage method located in AppViewModelBase. For now the view name must be specified in the call; this bothers me a bit and might be a place for refactoring later.  

\[csharp highlight="25,26,27"\] using System.Collections.Generic; using System.Collections.ObjectModel; using System.Windows.Controls; using System.Windows.Input; using GalaSoft.MvvmLight.Command;

namespace RiverBuddy.ViewModels { public class StatesViewModel : AppViewModelBase { /\* other code removed \*/

public ICommand StateSelectedCommand { get { return new RelayCommand<SelectionChangedEventArgs>(e => { // there is no SelectedIndex here if (e.AddedItems.Count == 0) return;

// shouldn't be more than one selected var state = (StateModel)e.AddedItems\[0\]; var dict = new Dictionary<string, object> {{"StateAbb", state.StateAbb}, {"StateName", state.StateName}}; SendNavigationMessage("StateRiversView", dict);

// selected index will have to be reset in the view //e.AddedItems.Clear(); return; } ); } } } } \[/csharp\]

### AppViewModelBase

Here there are a few different overloads for SendNavigationMessage including a direct Uri, viewName only, and viewName with a dictionary of parameters. In the case of parameters a Uri is constructed with a query string built from the key/value pairs.

\[csharp\] using System; using System.Collections.Generic; using System.Text; using GalaSoft.MvvmLight; using GalaSoft.MvvmLight.Messaging; using System.Linq;

namespace RiverBuddy.ViewModels { public class AppViewModelBase : ViewModelBase { protected void SendNavigationMessage(Uri uri) { Messenger.Default.Send(uri, "NavigationRequest"); }

protected void SendNavigationMessage(string viewName) { var uri = new Uri(string.Format("/Views/{0}.xaml", viewName), UriKind.Relative); SendNavigationMessage(uri); }

protected void SendNavigationMessage(string viewName, Dictionary<string, object> args) { var url = string.Format("/Views/{0}.xaml", viewName);

if (null != args) { var sb = new StringBuilder(); args.ToList().ForEach(kvp => { sb.Append(0 == sb.Length ? "?" : "&"); sb.AppendFormat("{0}={1}", kvp.Key, kvp.Value); });

url += sb.ToString(); }

var uri = new Uri(url, UriKind.Relative); SendNavigationMessage(uri); } } } \[/csharp\]

### PhoneApplicationPageBase

Each view ultimately inherits from this base view and first the navigation messages are registered and unregistered.  

\[csharp\] public class PhoneApplicationPageBase : PhoneApplicationPage { protected override void OnNavigatedTo(NavigationEventArgs e) { base.OnNavigatedTo(e); RegisterMessages(); } protected override void OnNavigatedFrom(NavigationEventArgs e) { base.OnNavigatedFrom(e); UnregisterAll() } private void RegisterMessages() { RegisterNavigationMessaging(); /\*...\*/ } protected void UnregisterAll() { Messenger.Default.Unregister(this); } protected void RegisterNavigationMessaging() { Messenger.Default.Register<Uri>(this, "NavigationRequest", Navigate); } private void Navigate(Uri uri) { NavigationService.Navigate(uri); } } \[/csharp\]

At this point code is in place to navigate over to the view being called. However on the view being called, I did not want to have to write code-behind code to reference the QueryString variables and then reference the ViewModel and call a Load method with those values.  
  

So the next thing I wanted to do was to automatically set properties on the ViewModel to the corresponding query string parameters. First a method was needed to go through the visual control tree from top to bottom, and look for a control with a DataContext of type AppViewModelBase and return that value.  

\[csharp\] protected AppViewModelBase GetViewModel() { var vm = FindViewModel(this.Content); return vm; }

protected AppViewModelBase FindViewModel(DependencyObject element) { var fe = element as FrameworkElement; var vm = (null != fe) ? fe.DataContext as AppViewModelBase : null; if (vm != null) { return vm; }

// recursively process nested children in the visual tree var children = VisualTreeHelper.GetChildrenCount(element); for (int i = 0; i < children; i++) { var child = VisualTreeHelper.GetChild(element, i); vm = FindViewModel(child); if (null != vm) { break; } }

return vm; } \[/csharp\]

With the ViewModel resolved, a method could then be created to enumerate the QueryString variables and set any matching property on the ViewModel. This method can then be called in OnNavigatedTo.  

\[csharp\] private void ProcessNavigationContext() { var count = this.NavigationContext.QueryString.Count; if (0 == count) return;

var vm = this.GetViewModel(); if (null == vm) return;

this.NavigationContext.QueryString.ToList().ForEach(kvp => { var propInfo = vm.GetType().GetProperty(kvp.Key); if (null != propInfo) { // NOTE: you may need other type checking and null handling depending on your datatypes if (propInfo.PropertyType == typeof(string)) propInfo.SetValue(vm, kvp.Value, null); else if (propInfo.PropertyType == typeof(long?) || propInfo.PropertyType == typeof(long)) propInfo.SetValue(vm, Convert.ToInt64(kvp.Value), null); } }); }

protected override void OnNavigatedTo(NavigationEventArgs e) { base.OnNavigatedTo(e);

RegisterMessages(); ProcessNavigationContext(); } \[/csharp\]

At this point the called view is loaded and its properties are set but I still want to automatically call a load type method on the called page's ViewModel. To accomplish that I added a "NavigatedTo" dependency property along with a modification in OnNavigatedTo to check for the presence of a NavigatedToCommand value and automatically execute it if set.  

\[csharp\] public static readonly DependencyProperty NavigatedToCommandProperty = DependencyProperty.Register("NavigatedToCommand", typeof(ICommand), typeof(PhoneApplicationPageBase), null); public ICommand NavigatedToCommand { get { return (ICommand)GetValue(NavigatedToCommandProperty); } set { SetValue(NavigatedToCommandProperty, value); } }

protected override void OnNavigatedTo(NavigationEventArgs e) { base.OnNavigatedTo(e);

RegisterMessages(); ProcessNavigationContext();

if (null != this.NavigatedToCommand && this.NavigatedToCommand.CanExecute(null)) this.NavigatedToCommand.Execute(null); } \[/csharp\]

### The called view

Now in the called view the NavigatedToCommand is set to be bound to the RefreshCommand on the ViewModel for the page.  

\[xml highlight="16"\] <Views:PhoneApplicationPageBase x:Class="RiverBuddy.Views.StateRiversView" xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation" xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml" xmlns:shell="clr-namespace:Microsoft.Phone.Shell;assembly=Microsoft.Phone" xmlns:d="http://schemas.microsoft.com/expression/blend/2008" xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006" xmlns:Views="clr-namespace:RiverBuddy.Views" FontFamily="{StaticResource PhoneFontFamilyNormal}" xmlns:VML="clr-namespace:RiverBuddy.ViewModels.Locators" FontSize="{StaticResource PhoneFontSizeNormal}" Foreground="{StaticResource PhoneForegroundBrush}" SupportedOrientations="PortraitOrLandscape" Orientation="Portrait" mc:Ignorable="d" d:DesignWidth="480" d:DesignHeight="696" d:DataContext="{Binding Source={StaticResource viewModelLocator}, Path=ViewModel}" shell:SystemTray.IsVisible="True" NavigatedToCommand="{Binding RefreshCommand}">

<Views:PhoneApplicationPageBase.Resources> <VML:StateRiversViewModelLocator x:Key="viewModelLocator" /> </Views:PhoneApplicationPageBase.Resources>

<!-- Page contents ommitted--> <Views:PhoneApplicationPageBase.DataContext> <Binding Source="{StaticResource viewModelLocator}" Path="ViewModel"/> </Views:PhoneApplicationPageBase.DataContext> </Views:PhoneApplicationPageBase> \[/xml\]

### Called View's ViewModel

Here the RefreshCommand will get called automatically and StateAbb and StateName will already be set prior to the call. No navigation code was required in either the calling or called views.  

\[csharp highlight="48,49,50,51"\] using System; using System.Collections.ObjectModel; using System.Windows.Input; using GalaSoft.MvvmLight.Command;

namespace RiverBuddy.ViewModels { public class StateRiversViewModel : AppViewModelBase { public StateRiversViewModel() { return; }

private string \_stateName; public string StateName { get { return \_stateName; } set { if (\_stateName != value) { \_stateName = value; RaisePropertyChanged("StateName"); } } }

private string \_stateAbb; public string StateAbb { get { return \_stateAbb; } set { if (\_stateAbb != value) { \_stateAbb = value; RaisePropertyChanged("StateAbb"); } } }

public void Refresh() { LoadRiversForState(this.StateAbb); }

public ICommand RefreshCommand { get { return new RelayCommand(Refresh); } }

public void LoadRiversForState(string stateAbb) { if (string.IsNullOrEmpty(stateAbb)) throw new ArgumentNullException("stateAbb", @"stateAbb is required");

/\* ...data load code... \*/ } /\* ... other code ... \*/ } } \[/csharp\]

### Why OnNavigatedTo over Loaded?

I could have used MVVM Light's EventToCommand, targeting the Loaded event at the Page level and this would not have required the NavigatedToCommand property on PhoneApplicationPageBase. I prefer to use OnNavigatedTo because I can override it and call base instead of having to wire (and optionally unwire) the Page's Loaded event. Additionally it sounds like there is more of a chance that the Loaded event could be called more than once. [This post](http://www.codebadger.com/blog/post/2010/10/05/WP7-Development-Tip-of-the-Day-Page-Startup-Loaded-event-vs-OnNavigatedTo-method.aspx) is one that talks a little about the differences between the two.  
  

### //TODO

I am not perfectly happy with this solution just yet but it is the best I've come up with so far. It needs some error handling and more testing with different cases but so far, so good. I've been considering more elegant solutions such as [Caliburn Micro for WP7](http://caliburnmicro.codeplex.com/wikipage?title=Working%20with%20Windows%20Phone%207&version=7) but I don't want to get too elaborate with frameworks on a phone device with limited resources. Of course there are simpler page navigation techniques that have their place as well. Also, while some of this is certainly phone specific in general this can all be applied just as well to regular Silverlight applications.  
  

### Updates

**10/12/2010** - Updated ProcessNavigationContext() to do basic type checking on QueryString paramters.  
  

**05/19/2011** - A while back I moved on to more of a navigation service similar to [this post by the creator of MVVM Light](http://blog.galasoft.ch/archive/2011/01/06/navigation-in-a-wp7-application-with-mvvm-light.aspx). The techniques here still work and parts of this can still compliment the navigation service approach. I just did not like the idea of the base page reliance for this and registering the navigation messages.

  
  
[Subscribe to this feed](http://feeds.feedburner.com/thnk2wn)
