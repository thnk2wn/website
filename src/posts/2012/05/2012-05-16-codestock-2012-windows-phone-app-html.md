---
title: "CodeStock 2012 Windows Phone App"
date: "2012-05-16"
---

The [CodeStock](http://codestock.org) conference is coming up soon on June 15th in downtown Knoxville. As such I have upgraded the [CodeStock 2011 Windows Phone app](/tech/2011/5/10/codestock-2011-app-for-windows-phone-7.html) for this year's conference and am pleased to announce it has been published to the marketplace.  
  

[Marketplace](http://windowsphone.com/s?appid=8b065e23-79bb-4a3c-8f65-0bc26a17ffbb)  

[Video](https://vimeo.com/42290848)

[Source Code](https://github.com/thnk2wn/codestock-winphone)

### Changes Over Last Year's Version

I have not had much time this year to enhance the application but below are highlights of what changed for the 2012 version. For more details see the [commit history](https://github.com/thnk2wn/codestock-winphone/commits/master).

- **Mango upgrade** - OS target upgrade from 7.0 to 7.1 and many fixes to address breaking changes
- **Map view** - new map view with various points of interest and directions. Points of interest are externally configured so if you have suggestions, let me know.
- **Branding** - logo, title, date, image and theme related changes
- **Source control and hosting** - switch from HG and bitbucket to git and github
- **New app, not an upgrade** - Due to various factors this year's version was submitted as a new app and last year's was hidden. If by chance you have last year's version installed, uninstall it first.

  

### Potential Future Changes

There is a chance I might make additional updates before the conference and might consider pull requests if you are so inclined. Feel free to [add an issue](https://github.com/thnk2wn/codestock-winphone/issues) for any bugs or feature requests.  
  

Some changes I'd like to see in the future:  

- **Live tile** - maybe showing next session via schedule and/or favorite
- **Better twitter integration** - i.e. more of a native twitter client instead of mobile twitter links.
- **More graceful public WiFi handling** - better error message when an HTML response (from a WiFi logon page) was received instead of expected JSON. (change pending Marketplace approval in v2.2)

On another Windows Phone project I started refactoring the Phone.Common assembly out into multiple NuGet packages that are a bit more generic. I have not had the time to finish that or incorporate it into the CodeStock app yet.  
  

### Overview Video

The below video gives an overview of most of the features minus a few miscellaneous ones.

<iframe src="http://player.vimeo.com/video/42290848?title=0&amp;byline=0&amp;portrait=0" width="398" height="728" frameborder="0"></iframe>

  
  

This page can also be accessed via [/codestock-wp7](/codestock-wp7).
