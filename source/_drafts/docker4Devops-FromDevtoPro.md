---
title: docker for Devops From Development to Production
tags:
---
**THe art of debugging issues**
- Check what the error is
- Check your code aginst mine
- Open Google and search for the error
- Report it in the course's forums

- Debian is a rock solid Linux distribution

**Debian (codename: Jessie)**
good side
- Extremely stable
- Has support for systemd
- Great support on hosting providers
down side
- You have to install Docker
- You have to install and cinfgure server clustering tools

**CoreOS**
good:
- Docker comes pre-installed
- Clustering tools come pre-installed
- Has support for systemd
down side:
- Relatively new and fast changing
- Limited hosting options
- Difficult to lock in a specific version of Docker

**CentOS (7.0*)**
good:
- Extemely stable
- Has support for systemd
- Great support on hosting providers

down side:
- You have to install Docker
- You have to install and configure server clustering tools

**'Snappy" Ubuntun Core & Red Hat Atomic**
good:
- Designed to run containers
- Has support for systemd
- Transactional updates

down side:
- Very very limted hosting options
- Extremely new and unproven
- Very little documentation and overall community blog posts, etc.

**What is systemd?**
- Drop in replacement for SystemV (init.d)
- Multiple binaries to help control your system
  - Process management
  - Log management
  - Repace cron
  - much more...

![who uses](https://i.imgur.com/EGx3QeV.png)

**Why systemd?**
- Concise "unit files"
- Process dependencies
- Unified toolset

**What does nginx do?**
- Web server, focused on concurrency and performance
- Reverse proxy
- Serve static assets
- Terminate SSL
- URL rewriting
- Virtual hosts
- Load balancer


