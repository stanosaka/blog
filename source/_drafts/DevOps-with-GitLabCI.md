---
title: DevOps with GitLab CI
tags:
- DevOps
- Gitlab
- GitOps
---
# Automated Devops Pipelines

## Automated DevOps with GitLab CI

What Is DevOps?
**Dev**elopment + **Op**erations**s**
involving three main principles:
- Systems thinking: Avoid focusing on only one piece
By systems we mean not just the software what we create, not just the computers what we run it on, but we think about the entire
process that's involved in human beings interacting with the system.
Everything that needs to happen in order for it to be useful and what we say is we want to avoid any kind of focus on one piece of that whole system that is not good for the entire system. So, we might make a change that is easier for the programmers but, if it makes the system harder to use for the users, that might not be a good change. We might make a change that makes the user interface easier or simpler but, if it means that there are lots of calls back to the database and performance goes down, that's not a good change. That's not systems thinking.

- Feedback: Keeping production healthy is everyone's job
- Learning: Continual experimentation and learing
we can continuously make updates to the system, make continuous improvements, and see the result of that in our feedback.

What is GitLab CI?

a Continuous Integration server buildt right into GitLab
- Build, test, and deploy code automatically on every change
- Keep build scripting together with code
- Works on gitlab.com or your own GitLab server

---
GitLab CI Demo
- Using a sample JavaScript/Node application:
   1. Create a repository on GitLab.com
   2. Build a Docker image
   3. Run integration and fuctional tests
   4. Deploy to Kubernetes
## integrating Kubernetes and GtLab CI

What is Docker?
Containerized software
- Isolated file system, network, process space
- Simplified application packaging and deployment

What is Kubernetes?
Container Orchestration
- Run, manage, upgrade Docker containers
- Automatic load balancing and failover
- Expose services to the outside world

In GitLab go to Operations click Kubernetes 
## DevOps with zero configuration

