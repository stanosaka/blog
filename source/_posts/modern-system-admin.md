---
title: modern system administration
date: 2021-03-20 20:39:38
tags: 
  - modern system administration
  - dveops
  - sre
---
# A role by any other name
To keep current with the tides of change within the industry, oranizations have taken to retitling their system administration postings to
devops engineer or siste reliability engineer (SRE).

May people think about devops as specific tools like Docker or Kubernetes, or practices ike continuous deployment and continuous integration.
What makes tools and practices "devops" is how they are used, not the tools or practices directly.


Site Reliability Engineering is an engineering discipline that helps an organization achieve the appropriate levels of reliability in their
systems, services, and products.

One measure of reliability is often used in exclustion to any other, and that is availability.

SLIs
SLOs

## How do devops and SRE differ?
While devops and SRE arose around the same time, devops is more focused on culture change (hat happens to impact technology and tools) while
SRE is very focused on changing the mode of Operations in general.

With SRE, there is often an expectation that engineers are also software engineers with operability skills. With DevOps Engineers, there is often an assumption that engineers are strong in at least one modern language as well as have expertise in continuous integration and deployment.

A sysadmin is someone who is responsible for building, configuring, and maintaining reliable systems where systems can be specific tools,
applications, or services.

What technical stacks are you familiar with?

# Chatper 1. Version Control
## Resolving Conflicts
- Cherry-picking
- If there are any uncommitted changes, stash them with git stash
- Start from the default branch so that it is the parent for the new feature branch to complete the emergency work. It's important that 
the emergency work isn't blocked by any work in progress on a different branch.
- Pull changes from the remote shared repository to the local default branch so that it's up-to-date to minimize any potential confilicts.
- Create a hotfix branch.
- Make changes, test,stage, commit, push, and PR with the hotfix branch following the same process as regular changes.
- Bring back any stashed work with git stash list, and git stash apply.

## Fixing your local repository
```
$ git reset FILE
```
This replaces FILE in the current working directory with the last committed version.
## Advancing Collaboration with VC
- Credit all collaborators

```
Commit Message describing what the set of changes do.


Co-authored-by: Name <email@example.com>

```
- **Have at least one reviewer** other than yourself before merging code

- **Write quality commit messages** explaining the context for the changes within the commit.

## Chapter 2. Local Development Environments
### Choosing an editor
The work that a sysadmin needs to do with an editor includes developing scripts, codifying infrastructure, writing documentation, composing tests,
and reading code.

For example, Visual Studio Code(VS Code)
### Minimizing required mouse usage
- Splitting the screen up vertically and horizontally
- integrated terminal with VS Code
### Integrated Static Code Analysis
- shellcheck 
### Easing editing through auto completion
- Docker extension
### Indenting code to match team conventions
In VS Code, this is visible in the lower panel along with other conventions about the file type. From the Command Palette, you can change the spacing from tabs to spaces.
### Collaborating while editing
- Live Share extension
### Integrating workflow with git
M icon: modified files
U: updated and unmerged
### Extending the development environment
- Remote Extension: allows individuals to use a remote system for "local" development with familiar local features

more aewsome extension[checkout](https://github.com/viatsko/awesome-vscode)

## Selecting languages to install
While you might not spend time developing applications, honing development skills in shell code and at least one language is essential.

Python, Ruby, and Go become much more useful to write utilities.

## Installing the Configuring Applications
- The Silver Searcher
- bash-completion
- cURL
verify whether you can connect to a URL, which is one of the first validations when checking a web service.
use it to send or retrieve data from a URL
- Docker
- hub
- jq
- Postman
- shellcheck
- tree

## Chapter 3: What is Security?
Security is the practice of protecting hardware, software, networks, and data from harm, theft, or unauthorized access.

## Chapter 4: Server virtualization and containers

