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
- Systems thinking: Avoid focusing on only one piece
- Feedback: Keeping production healthy is everyone's job
- Learning: Continual experimentation and learing

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

