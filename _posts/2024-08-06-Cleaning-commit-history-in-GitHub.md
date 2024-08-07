---
title: Git cleaning commit history in GitHub
date: 2024-08-06 17:00:00 -0700
categories: [GIT]
tags: [git]     # TAG names should always be lowercase
---

## Pre-requisites

Clone the repo from GitHub to any local computer or server

## 1. Backup the hidden .git folder

From the local copy of the repo, **create a backup** of the hidden `.git` file in the top folder and move it to any different folder outisde the local repo. I suggest to move it to a different folder hierarchy, otherwise it may still be detected. At least, in my case [VSCode](https://code.visualstudio.com/) was detecting it.

## 2. Delete the .git folder

After having backed up the `.git` folder, delete the one in the local repo. We should see no repositoriy initialized once the `.git` folder is removed

```
PS C:\Users\gusta\Documents\workspace_blog\gustavos86.github.io> git status
fatal: not a git repository (or any of the parent directories): .git
PS C:\Users\gusta\Documents\workspace_blog\gustavos86.github.io>
```

## 3. Initialize a new repository in the cloned/local repo

```
git init -b main
```

The `-b` flag is to name the initial branch, otherwise by default it is named `master`.

Note that the output of the command `git branch` will not show the branch name until the first commit is done later on

```
PS C:\Users\gusta\Documents\workspace_blog\gustavos86.github.io> git init -b main
Initialized empty Git repository in C:/Users/gusta/Documents/workspace_blog/gustavos86.github.io/.git/
PS C:\Users\gusta\Documents\workspace_blog\gustavos86.github.io> git branch
PS C:\Users\gusta\Documents\workspace_blog\gustavos86.github.io> git status
On branch main

No commits yet

Untracked files:
...
```

## 4. Stage all the files and create a new initial commit

```
git add *
git commit -am 'initial commit'
```

example

```
PS C:\Users\gusta\Documents\workspace_blog\gustavos86.github.io> git add *
PS C:\Users\gusta\Documents\workspace_blog\gustavos86.github.io> git status
On branch main

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
        new file:   .editorconfig
        new file:   .gitattributes
        ...


PS C:\Users\gusta\Documents\workspace_blog\gustavos86.github.io> git commit -am "clean commit"
[main (root-commit) de99e95] clean commit
 240 files changed, 25157 insertions(+)
 create mode 100644 .editorconfig
 ...


PS C:\Users\gusta\Documents\workspace_blog\gustavos86.github.io> git branch
* main
PS C:\Users\gusta\Documents\workspace_blog\gustavos86.github.io>  
```

## 5. Add the GitHub URL as Origin

```
git remote add origin <GITHUB_REPO_URL>
git remote -v
```

example

```
PS C:\Users\gusta\Documents\workspace_blog\gustavos86.github.io> git remote add origin https://github.com/gustavos86/gustavos86.github.io.git
PS C:\Users\gusta\Documents\workspace_blog\gustavos86.github.io> 
PS C:\Users\gusta\Documents\workspace_blog\gustavos86.github.io> git remote -v
origin  https://github.com/gustavos86/gustavos86.github.io.git (fetch)
origin  https://github.com/gustavos86/gustavos86.github.io.git (push)
PS C:\Users\gusta\Documents\workspace_blog\gustavos86.github.io>
```

## 6. Force pushing the update to GitHub

```
git push -f origin main
```

```
PS C:\Users\gusta\Documents\workspace_blog\gustavos86.github.io> git push -f origin main
Enumerating objects: 277, done.
Counting objects: 100% (277/277), done.
Delta compression using up to 8 threads
PS C:\Users\gusta\Documents\workspace_blog\gustavos86.github.io>
```

If not forced, the output of the command will tell you the remote repo has work not contained locally.

<details markdown=1>
<summary markdown="span">output</summary>

```
PS C:\Users\gusta\Documents\workspace_blog\gustavos86.github.io> git push
fatal: The current branch main has no upstream branch.
To push the current branch and set the remote as upstream, use

    git push --set-upstream origin main

To have this happen automatically for branches without a tracking
To push the current branch and set the remote as upstream, use

    git push --set-upstream origin main

To have this happen automatically for branches without a tracking

    git push --set-upstream origin main

To have this happen automatically for branches without a tracking

To have this happen automatically for branches without a tracking
upstream, see 'push.autoSetupRemote' in 'git help config'.

PS C:\Users\gusta\Documents\workspace_blog\gustavos86.github.io>
```
</details><br />

## 7. See the results in GitHub

In the GitHub repo we should nosw see a clean commit history

![]({{ site.baseurl }}/images/2024/08-06-Cleaning-commit-history-in-GitHub/01-Clean-commit-in-github.png)

## NOTE

Use this command to set the remote brach as the default upstream

```
git push --set-upstream origin main
```

example

```
PS C:\Users\gusta\Documents\workspace_blog\gustavos86.github.io> git push
fatal: The current branch main has no upstream branch.
To push the current branch and set the remote as upstream, use

    git push --set-upstream origin main

To have this happen automatically for branches without a tracking
upstream, see 'push.autoSetupRemote' in 'git help config'.

PS C:\Users\gusta\Documents\workspace_blog\gustavos86.github.io> git push --set-upstream origin main
Enumerating objects: 12, done.
Counting objects: 100% (12/12), done.
Delta compression using up to 8 threads
Compressing objects: 100% (8/8), done.
Writing objects: 100% (8/8), 15.58 KiB | 15.58 MiB/s, done.
Total 8 (delta 3), reused 0 (delta 0), pack-reused 0
remote: Resolving deltas: 100% (3/3), completed with 3 local objects.
To https://github.com/gustavos86/gustavos86.github.io.git
   de99e95..1cc5f3f  main -> main
branch 'main' set up to track 'origin/main'.
PS C:\Users\gusta\Documents\workspace_blog\gustavos86.github.io>
```

## References

- [how to delete all commit history in github?](https://stackoverflow.com/questions/13716658/how-to-delete-all-commit-history-in-github)
- [How can I create a Git repository with the default branch name other than "master"?](https://stackoverflow.com/questions/42871542/how-can-i-create-a-git-repository-with-the-default-branch-name-other-than-maste)