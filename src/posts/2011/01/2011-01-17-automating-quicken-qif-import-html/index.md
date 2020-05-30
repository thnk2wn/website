---
title: "Automating Quicken QIF Import"
date: "2011-01-17"
---

In my [last post on receipt automation](/tech/2010/11/23/receipt-automation-with-shoeboxed-quicken-evernote-and-more.html) I rambled on about different things I was trying to automate and otherwise ease managing my finances digitally. One TODO item from that post was easing the process of importing transaction data in QIF format into Quicken. Quicken stopped supporting QIF import some time ago with a move to an OFX format. I found I could not import QIF data into Quicken Essentials for Mac, though I tried in vein with an AppleScript. I ended up switching to Quicken Deluxe 2011 for Windows which had some limited QIF import into account types such as Cash accounts. That however wasn't ideal as it was a few different manual steps and it got old quick.  
  

### Approaching automating Quicken for Windows

With that in mind I decided to try automating the QIF import process on Windows. There are different frameworks for automating Windows. For now I chose Microsoft's UI Automation library mostly because it was "baked in" for free and what I wanted to automate seemed pretty straightforward. I knew there were various limitations with the UI Automation Library, such as those described here by [Brian Genisio](http://houseofbilz.com/archives/2008/10/04/ui-automation-not-fit-for-command/). For the small process I needed to automate though I did not consider those to be a roadblock, more just bumps along the way. I considered coding against the library with Powershell script but figured it would take me longer. I already had started a receipt scanner / uploader app and it made sense for me and some non-developers to build this into that application.  
  

### Magic Pixie Dust Hacks Happen Here

![](images/then-a-miracle-happens.gif)

I will ignore the ugliness in the middle for now and show a demo of the final product. There is not much to see; the devil is in the details but the basic steps are simple:

1. Quicken app instance reused if present, created if not (some wait for pwd)
2. Import QIF menu item is clicked
3. Values from receipt app are input into QIF import dialog, import button clicked
4. Unrecognized category dialog is checked for and handled if present
5. \# items imported is determined
6. Go to Register button is clicked
7. Imported transactions are selected
8. Move transactions menu item clicked
9. Account to move to is supplied, move button clicked

  
  
You will need to view full screen with HD on for detail unless you have really good vision.  
  

<iframe src="http://player.vimeo.com/video/18898429?title=0&amp;byline=0&amp;portrait=0" width="550" height="340" frameborder="0"></iframe>

  
  

### The usual disclaimers

- Only tested with Quicken Deluxe 2011 Windows
- Assumes last used Quicken file is where data should be imported
    - Most work with single quicken file per user. Adding filename adds another step
- Assumes the "temporary" (i.e. Cash) transfer (swap) account is empty at point of import
    - May work either way depending on data and details but not tested
- Limited support for error checking, edge cases, potential timing issues

  

### The Get 'er Done Code

The code isn't pretty but it gets the job done in what spare time I had. I did not consider it worth too much effort given this is somewhat "meware" at this point, it could easily break on future versions of Quicken, and I have doubts about how long I will have need of this functionality.  
  

The main import method:  
\[csharp\] public void Import() { if (null == this.AppSettings) throw new NullReferenceException("Quicken App Settings are required"); if (null == this.ImportSettings) throw new NullReferenceException("Quicken Import Settings are required");

this.ImportSettings.EnsureValid();

var launcher = new QuickenLauncher(this.AppSettings); var mainWin = launcher.Launch();

var menuBar = mainWin.FindFirst(TreeScope.Children, new PropertyCondition(AutomationElement.ControlTypeProperty, ControlType.MenuBar)); if (null == menuBar) throw new NullReferenceException("Failed to resolve menuBar");

ClickMenuItem(menuBar, this.ImportSettings.QifImportMenuPath);

var qifWin = WaitOnWindow(mainWin, this.ImportSettings.QifWindowImportTitle);

var qifFileTextBox = qifWin.FindFirst(TreeScope.Descendants, new PropertyCondition(AutomationElement.AutomationIdProperty, "100")); qifFileTextBox.SetFocus(); SendKeys.SendWait(ImportSettings.QifFilename); Thread.Sleep(500);

var acctCombo = qifWin.FindFirst(TreeScope.Descendants, new PropertyCondition(AutomationElement.AutomationIdProperty, "2302")); acctCombo.SetFocus(); SendKeys.SendWait(this.ImportSettings.ImportToAccountName);

ClickQuickenButton(qifWin, 32764); Thread.Sleep(1500);

var categoryMsgBox = mainWin.FindFirst(TreeScope.Descendants, new PropertyCondition(AutomationElement.NameProperty, this.AppSettings.QuickenVersion)); if (categoryMsgBox != null) { // click Yes on the question about adding new categories ClickQuickenButton(categoryMsgBox, 101); Thread.Sleep(1500); }

// we have to re-get the qif window even though we had a ref to it before qifWin = WaitOnWindow(mainWin, this.ImportSettings.QifWindowImportTitle); if (null == qifWin) return;

var itemsImportedLabel = qifWin.FindFirst(TreeScope.Descendants, new PropertyCondition(AutomationElement.AutomationIdProperty, "1014")); var importedText = itemsImportedLabel.Current.Name; int pos = importedText.IndexOf(" "); var importedCount = Convert.ToInt32(importedText.Substring(0, pos));

Win32API.SetForegroundWindow(qifWin.Current.NativeWindowHandle); // click Go To Register button. the done button is id 32767 ClickQuickenButton(qifWin, 1010);

Win32API.SetForegroundWindow(mainWin.Current.NativeWindowHandle); Thread.Sleep(2000);

var txWin = mainWin.FindFirst(TreeScope.Descendants, new PropertyCondition(AutomationElement.ClassNameProperty, "QWClass\_TransactionList")); if (null == txWin) throw new NullReferenceException("Failed to find transaction list window");

// need to get the focus into one of the transaction list fields var dateEditor = txWin.FindFirst(TreeScope.Descendants, new PropertyCondition(AutomationElement.AutomationIdProperty, "3")); if (null == dateEditor) throw new NullReferenceException("Failed to find date editor window"); dateEditor.SetFocus();

// focus will be after the last transaction entered. Shift + Up Arrow to select # of transactions imported // now that we've focused into date editor and not window itself need one extra shift+Up for (var i = 1; i <= importedCount+1; i++) { Thread.Sleep(50); SendKeys.SendWait("+{UP}"); }

Thread.Sleep(100); // bring up the move transactions dialog ClickMenuItem(menuBar, ImportSettings.MoveTransactionsMenuPath); var moveWin = WaitOnWindow(mainWin, "Move Transaction(s)"); if (null == moveWin) return; var moveToAcctCombo = moveWin.FindFirst(TreeScope.Descendants, new PropertyCondition(AutomationElement.AutomationIdProperty, "100")); moveToAcctCombo.SetFocus(); SendKeys.SendWait(this.ImportSettings.MoveToAccountName); ClickQuickenButton(moveWin, 32767); // move button } \[/csharp\]  

And some utility methods:  

\[csharp\] private static AutomationElement WaitOnWindow(AutomationElement mainWin, string title) { AutomationElement win = null;

int iterations = 0; while (win == null) { Win32API.SetForegroundWindow(mainWin.Current.NativeWindowHandle); win = mainWin.FindFirst(TreeScope.Descendants, new PropertyCondition( AutomationElement.NameProperty, title)); // we have to wait a tick before the QIF Import window is shown Thread.Sleep(500);

if (++iterations >= 8) break; } return win; }

private static void ClickQuickenButton(AutomationElement window, int automationId) { var btn = window.FindFirst(TreeScope.Descendants, new PropertyCondition( AutomationElement.AutomationIdProperty, automationId.ToString())); // can't use InvokePattern - GetSupportedPatterns() returns 0 - special QC\_button type //invokePattern = (InvokePattern) btn.GetCurrentPattern(InvokePattern.Pattern); //invokePattern.Invoke();

// can't set the focus either //btn.SetFocus(); //SendKeys.SendWait("{Enter}");

if (null == btn) throw new NullReferenceException(string.Format( "Couldn't find a button with automation id of {0} on window {1}", automationId, window.Current.Name));

// welcome to hack city. we'll move the mouse cursor and click the button var point = new Point((int)btn.Current.BoundingRectangle.Left, (int)btn.Current.BoundingRectangle.Top); Cursor.Position = point; Win32API.mouse\_event(Win32API.MOUSEEVENTF\_LEFTDOWN | Win32API.MOUSEEVENTF\_LEFTUP, point.X, point.Y, 0, 0); }

private static void ClickMenuItem(AutomationElement menuBar, string path) { var items = path.Split("|".ToCharArray()).ToList();

var parent = menuBar; items.ForEach(item=> { var menu = parent.FindFirst(TreeScope.Descendants, new PropertyCondition(AutomationElement.NameProperty, item));

if (menu.GetSupportedPatterns().Contains(ExpandCollapsePattern.Pattern)) { if (!menu.Current.IsEnabled) throw new InvalidOperationException( string.Format("Menu item at path '{0}' is not enabled; cannot expand", path)); var expandPattern = (ExpandCollapsePattern) menu.GetCurrentPattern( ExpandCollapsePattern.Pattern); expandPattern.Expand(); } else if (menu.GetSupportedPatterns().Contains(InvokePattern.Pattern)) { if (!menu.Current.IsEnabled) throw new InvalidOperationException( string.Format("Menu item at path '{0}' is not enabled; cannot click", path)); var invokePattern = (InvokePattern) menu.GetCurrentPattern(InvokePattern.Pattern); invokePattern.Invoke(); } parent = menu; }); } \[/csharp\]

### Code Notes

- Automation id was the surest way of referencing controls and I used Winspector Spy to retrieve the ids. It's a little hard to find online but Spy++ or similar works as well.
- Quicken buttons had a special QC\_button type that appeared to be preventing me from using invoke pattern to click the button. Focus didn't work either; moving the mouse cursor over the button bounds and sending a click via Win32 API certainly isn't ideal.
- Thread sleep calls were required to give Quicken time to respond to the previous action. In cases they might be removable, in other cases the delay might need adjusting. Other techniques could be used to wait and more safety checks could be added to ensure Quicken is at the point the code expects it to be. With reasonable enough delays I didn't have issues regardless of how much my machine was crawling at a given moment.
- QuickenAppSettings holds quicken app version info and has filename resolution. QuickenImportSettings includes account names, window titles, menu paths and related. Less fixed variables/settings are passed along from app.config to the settings classes. The QuickenLauncher class takes care of activating or starting Quicken.

### Full Source & Runtime Bits

The application requires the .net framework 4.0. More about the Receipt Transfer Utility application is available in my [prior post](/tech/2010/11/23/receipt-automation-with-shoeboxed-quicken-evernote-and-more.html) in this topic.  
  

[Source](/wp-content/uploads/2017/06/ShoeboxedScanner_Source.zip) | [App files only](/wp-content/uploads/2017/06/ShoeboxedScanner_Runtime.zip)

  
  

### Future thoughts

In addition to addressing code cleanup / refactoring / rewriting and aforementioned limitations, tagging imported transactions could be useful. This would help identify how the transactions were entered. Some items such as flags and notes can only be set on a per-transaction basis and that could really slow down the import. Quicken's Find/Replace dialog allows bulk editing fields like tags which could work. It appeared to have secondary dialogs though and since it is modal it would require extra work on another thread.  
  

Another thought I had for the [shoeboxed.com](http://shoeboxed.com) portion of this app was generating the Quicken QIF file myself using the Shoeboxed API images and receipt data. This would provide a couple benefits. First it would keep the quicken import process within the app without having to first go to the website and export the receipts then provide the filename to the app. Second I have had some issues with the category data being incorrect for Quicken-exported receipt data from Shoeboxed.com. The file format is easy enough to write but it appears the Shoeboxed API does not yet have all the data I would need. Ultimately if Intuit and Shoeboxed.com worked together for better integration I would not be stuck with the burden of working around these data transfer problems.
