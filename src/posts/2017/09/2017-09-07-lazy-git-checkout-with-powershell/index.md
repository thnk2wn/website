---
title: "Lazy Git Checkout with PowerShell"
date: "2017-09-07"
---

Some of the feature branch names of my current Git based project can get long at times. I was surprised at first that it didn't seem I could use a wildcard for a branch name when switching branches with `git checkout`. That led me to creating the below PowerShell function that was a pretty quick and effective way of doing a checkout by wildcard.

\[powershell\] function Set-Branch { \[CmdletBinding()\] \[Alias("Git-Checkout")\] PARAM ( \[Parameter(Mandatory=$true, Position=0)\] \[string\] $pattern )

$branches = @(git branch \` | Select-Object @{Name = "Name"; Expression = {$\_.Replace("\*", "").Trim()}} \` | Where-Object {$\_ -match $pattern})

if ($branches.Length -ne 1) { Write-Warning "Expected 1 branch match but found $($branches.Length). Refine pattern." $branches return }

$branch = $branches\[0\].Name git checkout $branch } \[/powershell\]

Sample usage follows.

C:\\source\\myproject \[Feature/ResourceLoading ≡\]\> git branch 
 Feature/AddChecksPage 
 Feature/ConsoleTesterPort1-3 
 Feature/CreateAdjustmentPage 
\* Feature/ResourceLoading 
 Feature/ServingOptionsWC789 
 Improvement/AllowUserToLockUnlockLocation 
 develop 
 master 
C:\\source\\myproject \[Feature/ResourceLoading ≡\]\> set-branch allow 
Switched to branch 'Improvement/AllowUserToLockUnlockLocation' 
Your branch is behind 'origin/Improvement/AllowUserToLockUnlockLocation' by 34 commits, and can be fast-forwarded. 
 (use "git pull" to update your local branch) 
C:\\source\\myproject \[Improvement/AllowUserToLockUnlockLocation ↓34\]\> 

I added this into my [My-GitUtilities.psm1](https://gist.github.com/thnk2wn/a9cddfa866272dd8aa9b6b4285a0f237) module that I import when loading my PowerShell profile script.
