---
title: Complete Git and Github Masterclass
tags: 'git,github'
date: 2019-05-06 00:25:42
---

# Complete Git and Github Masterclass
## How Git works - 3 main states of artifact
Modified: Here, the artifact goes through chaneg made by the user

Staged: Here, the user/developer adds the artifact to Git index or staging area

Committed: Here, the artifact gets safely stored in Gt database.

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

#  ██╗███████╗███████╗██╗   ██╗███████╗
#  ██║██╔════╝██╔════╝██║   ██║██╔════╝
#  ██║███████╗███████╗██║   ██║█████╗  
#  ██║╚════██║╚════██║██║   ██║██╔══╝  
#  ██║███████║███████║╚██████╔╝███████╗
#  ╚═╝╚══════╝╚══════╝ ╚═════╝ ╚══════╝
#                                      
If github ask for username and password:
> `git remote set-url origin git@github.com:szhouchoice/EffectiveDevOpsTerraform.git`

