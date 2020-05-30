---
title: "WP7 Orientation Based Scrolling"
date: "2010-10-09"
---

The main home view for my WP7 app has 7 main menu items in a ListBox and scrolling is not needed when the phone is in Portrait orientation. However I noticed that although a scrollbar does not appear, the ListBox was still scrollable in Portrait mode anyway. At first I set the ListBox's VerticalScrollBarVisiblity to Disabled but then quickly realized that cutoff the menu items in landscape orientation.  
  

I decided to address this via a quick custom converter for now:  

\[csharp\] using System; using System.Globalization; using System.Windows.Controls; using System.Windows.Data; using Microsoft.Phone.Controls;

namespace RiverBuddy.Converters { // ScrollViewer.VerticalScrollBarVisibility="{Binding Orientation, ElementName=uxPage, Mode=OneWay, Converter={StaticResource ScrollConverter}}"> public class NoScrollIfPortraitConverter : IValueConverter { public object Convert(object value, Type targetType, object parameter, CultureInfo culture) { //Debug.WriteLine("convert"); var orient = (PageOrientation)value; var vis = (orient == PageOrientation.Portrait || orient == PageOrientation.PortraitDown || orient == PageOrientation.PortraitUp) ? ScrollBarVisibility.Disabled : ScrollBarVisibility.Auto; return vis; }

public object ConvertBack(object value, Type targetType, object parameter, CultureInfo culture) { return null; } } } \[/csharp\]

There was no need for me to convert back the other direction. The View then consumes the converter as follows:  

\[xml highlight="12,15,19,20,21,40"\] <Views:PhoneApplicationPageBase x:Class="RiverBuddy.MainPage" xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation" xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml" xmlns:shell="clr-namespace:Microsoft.Phone.Shell;assembly=Microsoft.Phone" xmlns:d="http://schemas.microsoft.com/expression/blend/2008" xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006" d:DataContext="{d:DesignData SampleData/MainViewModelSampleData.xaml}" xmlns:i="clr-namespace:System.Windows.Interactivity;assembly=System.Windows.Interactivity" xmlns:cmd="clr-namespace:GalaSoft.MvvmLight.Command;assembly=GalaSoft.MvvmLight.Extras.WP7" xmlns:Views="clr-namespace:RiverBuddy.Views" xmlns:Converters="clr-namespace:RiverBuddy.Converters" FontFamily="{StaticResource PhoneFontFamilyNormal}" FontSize="{StaticResource PhoneFontSizeNormal}" Foreground="{StaticResource PhoneForegroundBrush}" SupportedOrientations="PortraitOrLandscape" Orientation="Portrait" mc:Ignorable="d" d:DesignWidth="480" d:DesignHeight="768" shell:SystemTray.IsVisible="True" x:Name="uxPage"> <Views:PhoneApplicationPageBase.Resources> <Converters:NoScrollIfPortraitConverter x:Key="ScrollConverter"/> </Views:PhoneApplicationPageBase.Resources>

<Grid x:Name="LayoutRoot"> <Grid.RowDefinitions> <RowDefinition Height="Auto"/> <RowDefinition Height="\*"/> </Grid.RowDefinitions> <Grid.Background> <ImageBrush ImageSource="/Images/HomeBackground.png"/> </Grid.Background> <StackPanel x:Name="TitlePanel" Grid.Row="0" Margin="24,24,0,12"> <TextBlock x:Name="ApplicationTitle" Text="River Buddy" Foreground="White" Margin="-10,0" Style="{StaticResource PhoneTextNormalStyle}"/> <TextBlock x:Name="ListTitle" Text="Home" Margin="-10,0" Foreground="White" Style="{StaticResource PhoneTextTitle1Style}"/> </StackPanel>

<Grid x:Name="ContentPanel" Grid.Row="1"> <ListBox x:Name="MainListBox" ItemsSource="{Binding Items}" SelectionChanged="MainListBox\_SelectionChanged" ScrollViewer.VerticalScrollBarVisibility="{Binding Orientation, ElementName=uxPage, Mode=OneWay, Converter={StaticResource ScrollConverter}}"> <i:Interaction.Triggers> <i:EventTrigger EventName="SelectionChanged" > <cmd:EventToCommand Command="{Binding MenuItemSelected}" PassEventArgsToCommand="True"/> </i:EventTrigger> </i:Interaction.Triggers> <ListBox.ItemTemplate> <DataTemplate> <StackPanel x:Name="DataTemplateStackPanel" Orientation="Horizontal"> <Image x:Name="ItemImage" Source="{Binding Image}" Height="43" Width="43" VerticalAlignment="Top" Margin="10,0,20,0"/> <StackPanel> <TextBlock x:Name="ItemText" Text="{Binding LineOne}" Margin="-2,-13,0,0" Foreground="White" Style="{StaticResource PhoneTextExtraLargeStyle}"/> <!-- removed PhoneTextAccent style now b/c of custom background--> <TextBlock x:Name="DetailsText" Text="{Binding LineTwo}" Opacity="0.65" Margin="0,-6,0,13" Style="{StaticResource PhoneTextAccentStyle}" Foreground="Yellow"/> </StackPanel> </StackPanel> </DataTemplate> </ListBox.ItemTemplate> </ListBox> </Grid> </Grid> </Views:PhoneApplicationPageBase> \[/xml\]

Subscribe to this feed at: [http://feeds.feedburner.com/thnk2wn](http://feeds.feedburner.com/thnk2wn)
