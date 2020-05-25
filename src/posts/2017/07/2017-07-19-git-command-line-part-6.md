---
title: "More Time with Git at the Command Line – Part 6"
date: "2017-07-19"
---

# Miscellaneous / Wrap-up

This last post in this series looks at miscellaneous topics including search, help, and additional resources.

## Series Outline

1. [Setup](https://geoffhudik.com/tech/2017/07/19/git-command-line-part-1/)

3. [Getting Latest and Making Changes](https://geoffhudik.com/tech/2017/07/19/git-command-line-part-2/)

5. [Pushing, Fetching, and Viewing History](https://geoffhudik.com/tech/2017/07/19/git-command-line-part-3/)

7. [Merging and Managing Branches](https://geoffhudik.com/tech/2017/07/19/git-command-line-part-4/)

9. [Stashes and Reverting Work](https://geoffhudik.com/tech/2017/07/19/git-command-line-part-5/)

11. [Miscellaneous / Wrap-up](https://geoffhudik.com/tech/2017/07/19/git-command-line-part-6/)

## Searching

### Searching Files

Once a file has been tracked by Git `[git grep](https://git-scm.com/docs/git-grep)` can be handy to search content. It's usually much faster than using Find in Files in an IDE, Windows Search (ugg), or even Search apps I like such as [Agent Ransack](https://www.mythicsoft.com/agentransack). It came in handy for me recently when searching large JSON files for test data; Notepad++ locked up for minutes trying to fully load and search the file but grep was nearly instant.

I might want to see where my name appears in code, usually TODO or other comments. Adding `-n` here shows matching line numbers in the output.

C:\\source\\myproject \[Feature/GuestId +1 ~0 -0 ~\]\> git grep \-n Geoff \-- \*.cs 
Company.MyApp.API/Data/Order/Repository/OrderRepository.cs:277: // TODO: Geoff, OrderDisplayType: Ex
pression passed to the Include operator could not be bound. This will need to come in later story. 
Company.MyApp/ViewModel/OrderViewModel.cs:249: "Geoff Hudik" 

### Searching Commit Contents ("Pickaxe")

The previous search shows the current state of things but I might want to search what commit added or removed a given string. Maybe I want to find out who added that TODO comment and when.

The below uses `git log -S'Geoff'` to find when my name was added; technically this searches commits where the number of occurrences of "Geoff" changed so it could find where my name was removed. I also want to exclude my own commits so I do that with `--perl-regexp --author='^((?!Geoff).*)$'`.

C:\\source\\myproject \[Feature/GuestId +1 ~0 -0 ~\]\> git log -S'Geoff' --perl-regexp --author='^((?!Geoff).\*)$' 
commit c6bd77e981c9bbff31a5acfe916057218b78d526 
Author: John Doe <john@doe.com> 
Date:   Tue May 16 17:09:43 2017 -0700 
 
 Code review changes. 
 
 XY-389 #comment Code review changes. 

The above commit looks like a winner so I check it out with `git show`.

C:\\source\\myproject \[Feature/GuestId +1 ~0 -0 ~\]\> git show c6bd77e981c9bbff31a5acfe916057218b78d526 
commit c6bd77e981c9bbff31a5acfe916057218b78d526 
Author: John Doe <john@doe.com> 
Date:   Tue May 16 17:09:43 2017 -0700 
 
 Code review changes. 
 
 XY-389 #comment Code review changes. 
 
diff --git a/Company.MyApp.Test/Infrastructure/Configuration/TestSettings.Registration.cs b/Company.
MyApp.Test/Infrastructure/Configuration/TestSettings.Registration.cs 
index 318d622..a1a0c68 100644 
\--- a/Company.MyApp.Test/Infrastructure/Configuration/TestSettings.Registration.cs 
+++ b/Company.MyApp.Test/Infrastructure/Configuration/TestSettings.Registration.cs 
@@ -5,7 +5,7 @@ namespace Company.MyApp.Test.Infrastructure 
 public sealed partial class TestSettings 
 { 
\- 
\+        // TODO: Geoff, does RegistrationSettings have to be an inner class? 
 /// <summary> 
 /// Returns registration settings for registration tests 
 /// </summary> 

Yup that John Doe guy stuck again, sneaking in TODO comments for me.

## Custom Help Functions

### Git SCM Website

I found myself spending a lot of time looking up git command details on the [Git SCM website](https://git-scm.com/). The below function simply builds a url given a specific git command name and then launches the url.

\[powershell\] function Get-GitCommand { \[CmdletBinding()\] \[Alias("Git-Command")\] PARAM ( \[Parameter(Mandatory=$true, Position=0)\] \[string\] $command )

$c = ($command -replace "git ", "") $url="https://git-scm.com/docs/git-$c" Start-Process $url "Launched $url" } \[/powershell\]

C:\\source\\myproject \[develop ≡\]\> git-command log   
Launched https://git-scm.com/docs/git-log 

### Custom Git Cheat Sheet

Especially when learning the commands it may be helpful to have your own Git "cheat sheet" and you might as well have that accessible at the command line too. Dumping a long list of a bunch of commands is probably overwhelming though and a single command is too granular and doesn't make sense for a cheat sheet. With that in mind I created a "Git-Cheat" function that returns examples with instructions on a category basis. If it's run with no category, or with an invalid category, it gives output like the below.

C:\\source\\myproject \[develop ≡\]\> git-cheat   
Specify a category:   
 branch   
 commit   
 diff   
 history   
 remote   
 stage   
 stash   
 undo 

Given a category it writes output like the below.

C:\\source\\myproject \[develop ≡\]\> git-cheat undo   
   
Undo   
\_\_\_\_   
   
Undo staging of specified files for next commit   
git reset -- \*Order\*.cs   
   
Revert specified file(s) to last commit (HEAD default)   
git checkout HEAD -- /path/file.ext   
git checkout -- /path/file.ext   
git checkout -- \*index.html   
   
Revert specified file(s) to specific commit   
git checkout commitId /path/file.ext   
   
Undo last N commits, keep changes staged   
git reset --soft HEAD~1   
git reset --soft HEAD~2   
   
Undo last N commits, unstage changes (mixed default)   
git reset --mixed HEAD~1   
   
Undo last N commits, delete changes (CAUTION!!!)   
git reset --hard HEAD~1   
   
Undo specified commit and apply as new commit   
git revert commitId   
git revert commitId --no-edit   
   
Favor git reset on unpushed commits, git revert on pushed commits 

### My-GitUtilities PowerShell Module

These help functions along with other custom PowerShell functions mentioned in this series are available in [My-GitUtilities.psm1](https://gist.github.com/thnk2wn/a9cddfa866272dd8aa9b6b4285a0f237).

## Additional Git Resources

- A fun, interactive web terminal Git tutorial by [try.github.io](https://try.github.io/levels/1/challenges/1)
- Git basics in a fun yet practical way - [Git basics - no deep shit](http://rogerdudler.github.io/git-guide/)
- [Atlassian's Git Tutorials](https://www.atlassian.com/git/tutorials) - generally well-written and straightforward.
- Pluralsight courses such as [How Git Works](https://www.pluralsight.com/courses/how-git-works) or [Git Fundamentals](https://www.pluralsight.com/courses/git-fundamentals)
- Presentation videos such as [Git More Done](https://vimeo.com/43659036) or [Git and GitHub for Developers on Windows](https://vimeo.com/43612883)
- [Pro Git book](https://www.amazon.com/Pro-Git-Scott-Chacon/dp/1484200772/ref=sr_1_2?ie=UTF8&qid=1500015684&sr=8-2&keywords=pro+git)

## Conclusion

Switching almost entirely to Git command line use can take some time but it can be done gradually. I've found that the initial investment has been well worth it - it's saved so much time and it has really helped me learn more about Git and what's going on "under the hood". I'm still very much a Git novice, so if you're an expert, feel free to enlighten me in the comments.

Hopefully some of these basics encourages someone to ease off that Git GUI app and spend more time at the command line. Don't be scurred, it's not that bad. There's a lot more to Git of course but the basics aren't bad, and well, one step at a time.
