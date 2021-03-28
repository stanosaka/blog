---
title: Complete Git and Github Masterclass
tags: 
  - git
  - github
date: 2019-05-06 00:25:42
---
# Git material
## Pro Git
[Pro Git](https://git-scm.com/book/zh/v2) 

## LearnGitBranching
[Learn Git Branching](https://learngitbranching.js.org/)

## tryGit
[tryGit](https://try.github.io/)

# Rebase with a pull
`git pull --rebase origin master`

It essentially puts the new commits in the master of the remote in your hisotry, and then superimpose your commits on them.

# Squash commits together
`git rebase -i HEAD~2`

Squash the last two commits.
The HEAD~2 refers to the last two commits in the current branch, and the -i option stands for interactive.
[pick and squash](https://i.imgur.com/AaMM1KK.png)
Pick the old commit and squash the latest commit.

# Aborting a squash
`git rebase --abort`
get back to the pre-squash state

# git log
`git log --oneline --decorate --graph --all`
# Complete Git and Github Masterclass
## How Git works - 3 main states of artifact
Modified: Here, the artifact goes through chaneg made by the user 

Staged: Here, the user/developer adds the artifact to Git index or staging area

Committed: Here, the artifact gets safely stored in Gt database.

### git add: files moved from modified state to staged state
### git commit: files moved from staged state to committed state

natural state progression is from modified-> staged-> committed

express commit: git commit -am "commiting readme.md"


## 3 sections of a Git project
Working directory: this is the root directory of your Git project

Staging area: Also called index, this is where all related changes are build up

Commit area: this is where all artifacts are stacked safely in Git Database

## 3 ways of setting up Git repository
From scratch: we will create a repository from absolutely blank state

Existing project: here we will convert an existing unversioned project to a Git repository

By Copying: we will copy an existing Git repository from Github

## Git Help system
`git help -a` and `git help -g` list available subcommands and some concept guides

## What is Fork and how to do it
What it means: Creating a project from another existing project

Encouragement: encourages project advancement & collaboration

Contribution: encourages outside contribution

Updates: Forks can updated

Command `git status`: tells us the status of the working directory and staging area

Command `git log`: displays committed snapshots or commit history


![Real world branching scenario](https://i.imgur.com/NKpsqvI.png)
## Git branching
Show branch: `git branch`

Create a new branch: `git branch demobranch`

Switch to the new branch: `git checkout demobranch`

checkout git branches: `git checkout; git checkout -b`

list git branches: `git branch; git branch -a`

rename a git branch: `git branch -m`

delete a git branch: `git branch -d; git branch -D`


## Undoing changes in a Git repository
### Checking out commits in a Git repository
`vim robot.txt`

Express git add and commit command:

```sh 
git commit -am "1st commit - robot.txt"

git log --oneline

git check 265fbe5
```
Create a new branch:

`git checkout -b newbranch`

Switch branch:

```sh
git checkout master

git checkout newbranch
```
### Checking out files in a Git repository
```sh
git log --oneline

git checkout bb3d025 checkoutfile.txt

git checkout HEAD checkoutfile.txt

git status
```
### Reverting changes in a Git repository
```sh
git add . 

git commit -m "commit file to revert"

git revert HEAD
```
### Resetting Git repository
`git reset --hard`

> resets the staging area and working directory to match the most recent commit.

![git reset --hard](https://i.imgur.com/XxVvIOm.png)

`git reset --hard hash_vaule`

> moves the current branch id backward to commit id we are using and reset the both the staging area and work dir to match, this destroy not the current change and both the commit id as well.

![git reset --hard hash_vaule](https://i.imgur.com/hDkx3Z8.png)

### Cleaning Git repository
```sh
git clean -n

git clean -f

git clean -fd

vim .gitignore

git add .gitignore

git commit -m "committing .gitignore"

git clean -xf
```
```sh
#  ####  ######   ######  ##     ## ######## 
#   ##  ##    ## ##    ## ##     ## ##       
#   ##  ##       ##       ##     ## ##       
#   ##   ######   ######  ##     ## ######   
#   ##        ##       ## ##     ## ##       
#   ##  ##    ## ##    ## ##     ## ##       
#  ####  ######   ######   #######  ######## 

#  ██╗███████╗███████╗██╗   ██╗███████╗
#  ██║██╔════╝██╔════╝██║   ██║██╔════╝
#  ██║███████╗███████╗██║   ██║█████╗  
#  ██║╚════██║╚════██║██║   ██║██╔══╝  
#  ██║███████║███████║╚██████╔╝███████╗
#  ╚═╝╚══════╝╚══════╝ ╚═════╝ ╚══════╝
#                                      
```
If github ask for username and password:
> `git remote set-url origin git@github.com:szhouchoice/EffectiveDevOpsTerraform.git`

![git flow](https://i.imgur.com/iIbiZSY.png)

**Merge branch into master**
```
# ...develop some code...
$ git add –A
$ git commit –m "Some commit message"
$ git checkout master
Switched to branch 'master'
$ git merge new-branch
```

```
git add foobar/foobar.php
git status
git reset HEAD foobar/foobar.php ## unstaged changes after reset
git diff
git log

## branching and merging
git checkout -b newfunction
vim foobar
git add *
git diff
git commit -m "Adds newfunction"
git log
git checkout master
git merge newfunction
git branch -D newfunction
```
# Git Fundamentals
## Initalizing a Repo and Adding Files (commands)
```
git init
# for creating a repository in a directory with existing files
# creates repository skeletion in .git directory
git add
# Tells git to start tracking files
# Patterns: git add *.c or git add. (.= files and dirs recursively)
git commit
# Promotes files into the local repository
# Uses -m to supply a comment
# Commits everthing unless told otherwise
```
## Undoing/updating things
```
git commit --amend
# allows you to "fix" last commit
# updates last commit using staging area
# with nothing new in staging area, just updates comment(commit message)
# example
  >> git commit -m "my first update"
  >> git add newfile.txt (add command stages it into staging area)
  >> git commit --amend
  >> This updated commit takes the place of the first (updates instead of adding adding another in the chain)
```
## Getting help
```
git <command> -h
# brings up on screen list of options
git hlep <command>
# brings up html man page
```
## Git config file
```
Scope
  Local (repo)
    >>.git/config
  Global (user)
    >> ~/.gitconfig
  System (all users on a system)
    >> <usr|usr/local>/etc/gitconfig
Divied into sections
Can have user settings, aliases, etc.
Text file that can be edited, but safer to use git config
```
## Git Status
```
Command: git status
  Primary tool fo showing which files are in which state
  -s is short option - output<1 or more characters><filename>
    >> ?? = untracked
    >> M = modified
    >> A = added
    >> D = deleted
    >> R = renamed
    >> C = copied
    >> U = updated but unmerged
  -b option - lways show branch and tracking info
Common usage: git status -sb
```
## Git special references: Head and Index
```
 Head
   snapshot of your last commit
   next parent (in chain of commits)
   pointer to current branch reference (reference = SHA1)
   Think of HEAD as pointer to last commit on current branch
 Index (Staging Area)
   place where changes for the next commit get registered
   temporary staging area for what you're working on
   proposed next commit
 Cache - old name for index
 Think of cache, index, and staging area as all the same
```
## Showing differences
```
  command: git diff
  default is to show changes in the workfing directory that are not yet staged
  if sth is staged, shows diff between working directory and staging area
  option of  --cached or --staged shows difference between staging area and last commit (HEAD)
  git diff <reference> shows differences between working directory and what <reference> points to - example "git diff HEAD"
```
## History
``` 
  command: git log
  with no options, shows name, sha1, email, commit message in reverse chronological order (newest first)
  -p option shows patch - or differences in each commit 
     Shortcut: git show
  -# -shows last # commits
  --stat - shows statistics on number of changes
  --pretty=oneline|short|full|fuller
  --format - allows you to specify your own output format
  --oneline and --format can be very useful for machine parsing
  Time-limiting options --since|until|after|before (i.e. --since=2.weeks or --before=3.19.2011)
  gitk tool also has history visualizer
  git log --oneline <since>..<until>  e.g. foobar..barfoo
  git log --oneline <file-name>   e.g. teststatusfile
  git log --oneline -n <limit> e.g. 2
```


## git useful command
```
git clone git@bitbucket.org:foobar/pagerduty.git --config core.autocrlf=input
```
