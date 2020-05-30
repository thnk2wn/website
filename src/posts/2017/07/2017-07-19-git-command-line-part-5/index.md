---
title: "More Time with Git at the Command Line - Part 5"
date: "2017-07-19"
---

# Stashing and Reverting Work

This post in the series looks at stashing away changes, managing stashes, and reverting work.

## Series Outline

1. [Setup](https://geoffhudik.com/tech/2017/07/19/git-command-line-part-1/)

3. [Getting Latest and Making Changes](https://geoffhudik.com/tech/2017/07/19/git-command-line-part-2/)

5. [Pushing, Fetching, and Viewing History](https://geoffhudik.com/tech/2017/07/19/git-command-line-part-3/)

7. [Merging and Managing Branches](https://geoffhudik.com/tech/2017/07/19/git-command-line-part-4/)

9. [Stashes and Reverting Work](https://geoffhudik.com/tech/2017/07/19/git-command-line-part-5/)

11. [Miscellaneous / Wrap-up](https://geoffhudik.com/tech/2017/07/19/git-command-line-part-6/)

## Stashing Overview

Sometimes I might be in the middle of working on one story / feature and suddenly need to shift to another story; maybe a serious bug came up or a more important, time-sensitive story needs to be worked on first. Or, maybe I stay focused on one story only but it requires some wild discovery experiments. In that case I may need to be able to quickly reset the code and try other approaches but I don't want to lose the ability to get back to previous approaches. Also, I may just feel unsure about some pending changes and want to reset them, but still want the ability to get back to them later if I change my mind. Finally, maybe I need to pull code from the remote repo but I have uncommitted pending changes getting in my way.

In these situations, there are a few options I might take.

1. I could commit the incomplete work, perhaps with temporary code to disable some of it. This isn't ideal as it could create problems and generally commits should have a certain level of completeness - maybe code compilation, passing unit tests, meeting coding standards, and/or a logical set of working changes ("functional unit of work").

3. I could abandon the changes with `git reset --hard`. Note it might be best to first run `git reset` to see what would be reset. This also isn't ideal as the changes are abandoned and ideally I'd like them saved in some fashion.

5. I could do something wonky like manually backing up certain files or the whole repo to another directory. Alas, this stone age technique would make me feel dirty, is too manual, I might miss some changed files, and I might not even remember where I backed up the changes to later.

7. I could use `[git stash](https://git-scm.com/docs/git-stash)` to save change snapshots and revert my working directory and still get back to them later. This turns out to be a handy "pause button" for work in progress.

Git stash records the state of the working directory and index returns me back to a clean working directory. It could almost be thought of a lightweight, "off the record commit" followed by revert/reset. In some ways it's similar to a shelveset in TFS except it's local only - stashes aren't sent to the server when pushing so they should be considered more transient in nature.

## Creating Stashes (Save and Revert)

### Git Stash (No Message)

Let's say I start off with these changes:

C:\\source\\myproject \[Feature/GuestId +1 ~1 -1 !\]\> git status   
On branch Feature/GuestId   
Changes not staged for commit:   
 (use "git add/rm <file>..." to update what will be committed)   
 (use "git checkout -- <file>..." to discard changes in working directory)   
   
 modified:   CONTRIBUTING.md   
 deleted:    Readme.txt   
   
Untracked files:   
 (use "git add <file>..." to include in what will be committed)   
   
 Utilities.ps1   
   
no changes added to commit (use "git add" and/or "git commit -a") 

Now I run `git stash`:

C:\\source\\myproject \[Feature/GuestId +1 ~1 -1 !\]\> git stash   
Saved working directory and index state WIP on Feature/GuestId: aae9719 readme change   
HEAD is now at aae9719 readme change   
C:\\source\\myproject \[Feature/GuestId +1 ~0 -0 !\]\> 

After the stash I check status again:

C:\\source\\myproject \[Feature/GuestId +1 ~0 -0 !\]\> git status   
On branch Feature/GuestId   
Untracked files:   
 (use "git add <file>..." to include in what will be committed)   
   
 Utilities.ps1   
   
nothing added to commit but untracked files present (use "git add" to track) 

I wasn't expecting newly added files to still be there initially. It makes sense that Git just reverts changes to tracked files it knows about - those untracked files could be anything really. Had I done a git add on _Utilities.ps1_ before stashing then the stash would've reverted (deleted) that file. It turns out that using `git stash save -u` will stash untracked files and then remove them with `[git clean](https://git-scm.com/docs/git-clean)`. To be more explicit, `git stash save --include-untracked` does the same thing.

### Git Stash with Message

When stashing it's often helpful to include a message describing the changes in that stash. This way when the list of stashes is viewed later, it'll be easier to tell what is what. At first I thought this message was actually a name that could be used to reapply the stash by later but found that didn't appear to be the case.

C:\\source\\myproject \[Feature/GuestId +0 ~1 -0 !\]\> git stash save "Program.cs Trying Something Crazy"   
Saved working directory and index state On Feature/GuestId: Program.cs Trying Something Crazy   
HEAD is now at aae9719 readme change   
C:\\source\\myproject \[Feature/GuestId\]\> 

### Git Stash Without Reverting

If I've already updated the index (staging area) with what I want to commit (`git add _..._`) I can do `git stash save --keep-index` and it will create a stash for me but not reset/revert all the changes I already added to the index - only unstaged changes will be reset.

## Viewing Stash Details

### Viewing the Stash List

C:\\source\\myproject \[Feature/GuestId\]\> git stash list   
stash@{0}: On Feature/GuestId: Program.cs Trying Something Crazy   
stash@{1}: WIP on Feature/GuestId: aae9719 readme change   
stash@{2}: WIP on Feature/GuestId: aae9719 readme change   
stash@{3}: On Feature/LoadSession: LocationPageVMAsync 

### Viewing Stash Changes (Diffstat)

To view change overview for a specific stash I can do `git stash show 'stash@{_N_}'` where _N_ is the stash index shown in the list. Note the single quotes around stash@{_N_}; quotes won't appear in git documentation and they normally aren't needed. However, @ is an array operator in PowerShell so without the quotes I'd see "fatal: ambiguous argument 'stash@': unknown revision or path not in the working tree.".

C:\\source\\myproject \[Feature/GuestId\]\> git stash show 'stash@{2}'   
 CONTRIBUTING.md | 2 +\- 
 Readme.txt      | 5 \----- 
 2 files changed, 1 insertion(+), 6 deletions(-) 

### Viewing Stash Diffs

If I add `-p` into `git stash show` I'll see the stash in patch form where I can view the diff details.

C:\\source\\myproject \[Feature/GuestId\]\> git stash show 'stash@{2}' \-p   
diff --git a/CONTRIBUTING.md b/CONTRIBUTING.md   
index dc9065d..bdf2528 100644   
\--- a/CONTRIBUTING.md   
+++ b/CONTRIBUTING.md   
@@ -19,4 +19,4 @@   
   
 3. At least one developer must approve the pull request   
   
\-Finally, hit the Merge button and take a coffee or tea break :)   
+Finally, hit the Merge button and take a coffee or tea break. :)   
\\ No newline at end of file 

## Applying (Restoring) Stashes

After stashing away my changes and resetting my workspace, there are a few ways to get back to those stashed changes later.

### Applying a Stash by Index

Stashes can be applied in any order. When a stash is applied changes in that stash are replayed over the current working tree. Effectively this is a merge so applying a stash can lead to conflicts. One way to apply a stash is by specifying in index form.

C:\\source\\myproject \[Feature/GuestId\]\> git stash apply 'stash@{2}'   
Removing Readme.txt   
On branch Feature/GuestId   
Changes not staged for commit:   
 (use "git add/rm <file>..." to update what will be committed)   
 (use "git checkout -- <file>..." to discard changes in working directory)   
   
 modified:   CONTRIBUTING.md   
 deleted:    Readme.txt   
   
no changes added to commit (use "git add" and/or "git commit -a")   
C:\\source\\myproject \[Feature/GuestId +0 ~1 -1 !\]\> 

### Applying the Last Stash

Another option is to just use `git stash apply` which will apply the most recent stash (stash@{0}).

### Applying the Last Stash Then Removing It

A third option is using `git stash pop` which will apply the most recent stash and then pop it off the stash list (drops / deletes stash). Technically the stash apply could fail (effectively a merge), so there could be conflicts, in which case the stash would not be dropped.

C:\\source\\myproject \[Feature/GuestId\]\> git stash list   
stash@{0}: On Feature/GuestId: Program.cs Trying Something Crazy   
stash@{1}: WIP on Feature/GuestId: aae9719 readme change   
stash@{2}: WIP on Feature/GuestId: aae9719 readme change   
stash@{3}: On Feature/LoadSession: LocationPageVMAsync   
C:\\source\\myproject \[Feature/GuestId\]\> git stash pop   
On branch Feature/GuestId   
Changes not staged for commit:   
 (use "git add <file>..." to update what will be committed)   
 (use "git checkout -- <file>..." to discard changes in working directory)   
   
 modified:   Company.MyApp.ServiceHost/Program.cs   
   
no changes added to commit (use "git add" and/or "git commit -a")   
Dropped refs/stash@{0} (bd56b2ed6f8506b6bd1623e0eb095373e8bd8c9a)   
C:\\source\\myproject \[Feature/GuestId +0 ~1 -0 !\]\> git stash list   
stash@{0}: WIP on Feature/GuestId: aae9719 readme change   
stash@{1}: WIP on Feature/GuestId: aae9719 readme change   
stash@{2}: On Feature/LoadSession: LocationPageVMAsync   
C:\\source\\myproject \[Feature/GuestId +0 ~1 -0 !\]\> 

Sometimes this is most handy if you want to pull the latest into your working tree but you have uncommitted changes. You can stash your changes away, pull the latest to your workspace, re-apply your stashed changes over your workspace, then drop the temporary stash.

`> git stash  
> git pull  
> git stash pop  
`

### Applying a Stash by Message

I found a Stack Overflow post about applying a stash by name / message but the answer didn't work for me and comments of others indicated the same. That led me to creating the following function which is in a custom Git utility module I created that I import via my PowerShell profile (more on that in the next post). The code could be condensed but I'm erroring on the side of better error messages, readability, and built-in help.

\[powershell\] <# .SYNOPSIS Restores (applies) a previously saved stash based on full or partial stash name.

.DESCRIPTION Restores (applies) a previously saved stash based on full or partial stash name and then optionally drops the stash. Can be used regardless of whether "git stash save" was done or just "git stash". If no stash matches a message is given. If multiple stashes match a message is given along with matching stash info.

.PARAMETER message A full or partial stash message name (see right side output of "git stash list"). Can also be "@stash{N}" where N is 0 based stash index.

.PARAMETER drop If -drop is specified, the matching stash is dropped after being applied.

.EXAMPLE Restore-Stash "Readme change" Apply-Stash MyStashName Apply-Stash MyStashName -drop Apply-Stash "stash@{0}" #> function Restore-Stash { \[CmdletBinding()\] \[Alias("Apply-Stash")\] PARAM ( \[Parameter(Mandatory=$true)\] $message, \[switch\]$drop )

$stashId = $null

if ($message -match "stash@{") { $stashId = $message }

if (!$stashId) { $matches = git stash list | Where-Object { $\_ -match $message } if (!$matches) { Write-Warning "No stashes found with message matching '$message' - check git stash list" return }

if ($matches.Count -gt 1) { Write-Warning "Found $($matches.Count) matches for '$message'. Refine message or pass 'stash{@N}' to this function or git stash apply" return $matches }

$parts = $matches -split ':' $stashId = $parts\[0\] }

git stash apply ''$stashId''

if ($drop) { git stash drop ''$stashId'' } } \[/powershell\]

The below console output shows a list of stashes and a few examples of applying a stash by name via this function.

C:\\source\\myproject \[develop ↓12 ↑1\]\> git stash list 
stash@{0}: On develop: Markdown changes again 
stash@{1}: On Feature/SomeCoolThang: Crazy Change 
stash@{2}: WIP on Feature/SomeCoolThang: b48736a temp 
stash@{3}: On Feature/SomeCoolThang: Markdown changes 
stash@{4}: On Feature/SomeCoolThang: Markdown changes 
C:\\source\\myproject \[develop ↓12 ↑1\]\> 
C:\\source\\myproject \[develop ↓12 ↑1\]\> apply-stash markdonw 
WARNING: No stashes found with message matching 'markdonw' - check git stash list 
C:\\source\\myproject \[develop ↓12 ↑1\]\> 
C:\\source\\myproject \[develop ↓12 ↑1\]\> apply-stash markdown 
WARNING: Found 3 matches for 'markdown'. Refine message or pass 'stash{@N}' to this function or git stash apply 
stash@{0}: On develop: Markdown changes again 
stash@{3}: On Feature/SomeCoolThang: Markdown changes 
stash@{4}: On Feature/SomeCoolThang: Markdown changes 
C:\\source\\myproject \[develop ↓12 ↑1\]\> 
C:\\source\\myproject \[develop ↓12 ↑1\]\> apply-stash "Markdown changes again" 
On branch develop 
Your branch and 'origin/develop' have diverged, 
and have 1 and 12 different commits each, respectively. 
 (use "git pull" to merge the remote branch into yours) 
Changes not staged for commit: 
 (use "git add <file>..." to update what will be committed) 
 (use "git checkout -- <file>..." to discard changes in working directory) 
 
 modified:   CONTRIBUTING.md 
 
no changes added to commit (use "git add" and/or "git commit -a") 
C:\\source\\myproject \[develop ↓12 ↑1 +0 ~1 -0 !\]\> 
C:\\source\\myproject \[develop ↓12 ↑1 +0 ~1 -0 !\]\> git checkout \-- CONTRIBUTING.md 
C:\\source\\myproject \[develop ↓12 ↑1\]\> 
C:\\source\\myproject \[develop ↓12 ↑1\]\> apply-stash crazy \-drop 
Auto-merging CONTRIBUTING.md 
CONFLICT (content): Merge conflict in CONTRIBUTING.md 
Dropped stash@{1} (6b1f69e39f5806d54fbe9fe574d64b45e7e4597f) 
C:\\source\\myproject \[develop ↓12 ↑1 +0 ~0 -0 !1 | +0 ~0 -0 !1 !\]\> 

Not shown above is invoking the function by stash id such as `Apply-Stash '@stash{_N_}'` which is the same as calling git stash apply but could be useful if the first call to the function returned multiple matches.

With the comments added for the function I can use `Get-Help Apply-Stash` or `Get-Help Restore-Stash` for additional reference.

C:\\source\\myproject \[develop ↓12 ↑1\]\> get-help apply-stash   
   
NAME   
 Restore-Stash   
   
SYNOPSIS   
 Restores (applies) a previously saved stash based on full or partial stash name.   
   
   
SYNTAX   
 Restore-Stash \[-message\] <Object> \[-drop\] \[<CommonParameters>\]   
   
   
DESCRIPTION   
 Restores (applies) a previously saved stash based on full or partial stash name and then optionally drops the   
 stash. Can be used regardless of whether "git stash save" was done or just "git stash". If no stash matches a   
 message is given. If multiple stashes match a message is given along with matching stash info.   
   
   
RELATED LINKS   
   
REMARKS   
 To see the examples, type: "get-help Restore-Stash -examples".   
 For more information, type: "get-help Restore-Stash -detailed".   
 For technical information, type: "get-help Restore-Stash -full". 

## Cleaning up Stashes

Often I create stashes just in case and they sit for a while and later I realize they are no longer relevant so I delete them using `git stash drop`.

C:\\source\\myproject \[Feature/GuestId\]\> git stash drop 'stash@{4}'   
Dropped stash@{4} (194ba42c308f0419ea257a5b4107ccd0a5cc19e4) 

## Undoing Commits with Reset

Using `git reset` has a few different modes but it can be used to undo a commit and make it look like it never happened from a history perspective. Generally I would only use this if I haven't pushed these commits for others to see and maybe I just made a hasty local commit.

### Undo Last N Commits, Keep Changes Staged

If I made the last commit prematurely, I can undo it with `git reset --soft HEAD~1` and leave the files from that commit staged. That way I can continue editing and correcting those files then make a cleaner commit.

C:\\source\\myproject \[Feature/GuestId\]\> "hello 1" >out.txt 
C:\\source\\myproject \[Feature/GuestId +1 ~0 -0 !\]\> git add out.txt 
C:\\source\\myproject \[Feature/GuestId +1 ~0 -0 ~\]\> git commit \-m 'commit 1' 
\[Feature/GuestId 74fb5bd\] commit 1 
 1 file changed, 0 insertions(+), 0 deletions(-) 
 create mode 100644 out.txt 
C:\\source\\myproject \[Feature/GuestId\]\> git reset --soft HEAD~1 
C:\\source\\myproject \[Feature/GuestId +1 ~0 -0 ~\]\> git status 
On branch Feature/GuestId 
Changes to be committed: 
 (use "git reset HEAD <file>..." to unstage) 
 
 new file:   out.txt 
 
C:\\source\\myproject \[Feature/GuestId +1 ~0 -0 ~\]\> type out.txt 
hello 1 
C:\\source\\myproject \[Feature/GuestId +1 ~0 -0 ~\]\> git history -2 
61576d3 N 41 minutes.. Geoff Hu.. SLN rename 
930737c N2 days ago Geoff Hu.. changing some tools and markdown 

Note that commit in history/log is gone and the file still exists in my workspace and in the index ready to commit. In a similar fashion I could do `git reset --soft HEAD~3` and undo the last 3 commits. Note `git history` is a custom alias from a prior post in this series.

### Undo Last N Commits, Unstage Changes

A mixed `git reset` type does the same as the above but the changed files just remain in my local workspace, they are not in the index to be committed. Maybe after rolling back the commit it's easier to have everything unstaged and review the changes to more selectively include what files should go in the next commit.

C:\\source\\myproject \[Feature/GuestId +0 ~1 -0 !\]\> git commit \-am 'Contributing markdown changes' 
\[Feature/GuestId 8ef67c4\] Contributing markdown changes 
 1 file changed, 3 insertions(+) 
C:\\source\\myproject \[Feature/GuestId\]\> git reset --mixed HEAD~1 
Unstaged changes after reset: 
M       CONTRIBUTING.md 
C:\\source\\myproject \[Feature/GuestId +0 ~1 -0 !\]\> git history -1 
61576d3 N 74 minutes.. Geoff Hu.. SLN rename 
    

Mixed mode is the default so I could've also just done `git reset HEAD~1` without specifying the reset type.

### Undo Last N Commits, Delete Changes

**Warning!** A hard git reset is destructive and should be used with caution.

C:\\source\\myproject \[Feature/GuestId +1 ~2 -1 ~\]\> git commit \-m 'changes' 
\[Feature/GuestId 77c7e9d\] changes 
 4 files changed, 1 insertion(+), 56 deletions(-) 
 rename Company.MyApp.sln => Company.MyApp.Temp.sln (100%) 
 delete mode 100644 Company.MyApp.sln.GhostDoc.xml 
 create mode 100644 README.md 
C:\\source\\myproject \[Feature/GuestId\]\> git reset --hard HEAD~1 
HEAD is now at a5447e0 Merged in fix/GuestIdMultiline (pull request #130) 
C:\\source\\myproject \[Feature/GuestId\]\> type readme.md 
type : Cannot find path 'C:\\source\\myproject\\readme.md' because it does not exist. 
At line:1 char:1 
\+ type readme.md 
\+ ~~~~~~~~~~~~~~ 
 + CategoryInfo          : ObjectNotFound: (C:\\source\\myproject\\readme.md:String) \[Get-Content\], ItemNotFoundExcept
 ion 
 + FullyQualifiedErrorId : PathNotFound,Microsoft.PowerShell.Commands.GetContentCommand 
 
C:\\source\\myproject \[Feature/GuestId\]\> git log --oneline -1 
a5447e0 Merged in fix/GuestIdMultiline (pull request #130) 

In this case the README.md file that was created and committed will be gone permanently - it won't show up in history and it won't be staged for commit or in my workspace anymore.

## Undoing Changes with Revert

Especially if I've shared my commits with others (e.g. push), `git revert` is better as it undoes changes without losing history. The specified commit is reverted and those undo changes are committed as a new commit.

C:\\source\\myproject \[Feature/GuestId\]\> "DELETE FROM ACCOUNTS WHERE STATUS = 4" >delete.sql 
C:\\source\\myproject \[Feature/GuestId +1 ~0 -0 !\]\> git add delete.sql 
C:\\source\\myproject \[Feature/GuestId +1 ~0 -0 ~\]\> git commit \-m 'adding delete sql' 
\[Feature/GuestId 6498acb\] adding delete sql 
 1 file changed, 0 insertions(+), 0 deletions(-) 
 create mode 100644 delete.sql 
C:\\source\\myproject \[Feature/GuestId\]\> "DELETE FROM ACCOUNTS" >>delete.sql 
C:\\source\\myproject \[Feature/GuestId +0 ~1 -0 !\]\> git commit \-am 'adding more deletes, in a rush' 
\[Feature/GuestId 26de3fd\] adding more deletes, in a rush 
 1 file changed, 0 insertions(+), 0 deletions(-) 
C:\\source\\myproject \[Feature/GuestId\]\> "<h1>Hello World</h1>" >index.html 
C:\\source\\myproject \[Feature/GuestId +0 ~1 -0 !\]\> git add . 
C:\\source\\myproject \[Feature/GuestId +0 ~1 -0 ~\]\> git commit \-m index 
\[Feature/GuestId c573215\] index 
 1 file changed, 0 insertions(+), 0 deletions(-) 
C:\\source\\myproject \[Feature/GuestId\]\> git history -4 
c573215 N 14 seconds.. Geoff Hu.. index 
26de3fd N 43 seconds.. Geoff Hu.. adding more deletes, in a rush 
6498acb N 78 seconds.. Geoff Hu.. adding delete sql 
00435bf N 2 minutes .. Geoff Hu.. readme changes 
C:\\source\\myproject \[Feature/GuestId\]\> git revert 26de3fd --no-edit 
\[Feature/GuestId 3bc3b28\] Revert "adding more deletes, in a rush" 
 1 file changed, 0 insertions(+), 0 deletions(-) 
C:\\source\\myproject \[Feature/GuestId\]\> git history -4 
3bc3b28 N 10 seconds.. Geoff Hu.. Revert "adding more deletes, in a rush" 
c573215 N 47 seconds.. Geoff Hu.. index 
26de3fd N 76 seconds.. Geoff Hu.. adding more deletes, in a rush 
6498acb N 2 minutes .. Geoff Hu.. adding delete sql 
C:\\source\\myproject \[Feature/GuestId\]\> type delete.sql 
DELETE FROM ACCOUNTS WHERE STATUS = 4 

Notes:  

- In the above example the 'rush commit' left off a WHERE clause in the SQL statement.
- That commit is undone with `git revert 26de3fd --no-edit`.
- Use of `--no-edit` tells Git to not start the commit message editor (vim by default) to enter a custom commit message. By default the message with be "Revert \[message of commit being reverted\]". Removing this would allow entering a custom message such as "Forgot WHERE clause - dropped all the accounts LULZ."
- The revert commit undid the "DELETE FROM ACCOUNTS" line add.
- Note that this reverted a commit "in the middle" of history, leaving surrounding commits as is.
- Generally there might be a push (not shown above) before doing the revert.
