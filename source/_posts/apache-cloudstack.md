---
title: apache cloudstack
date: 2019-09-19 22:56:11
tags: cloudstack
---
# What is Apache CloudStack
- Top level Apache Software Foundation project for cloud computing
- Cloud computing framework to deploy IaaS cloud- private, public or hybrid
- Supports all popular hypervisors for virtualization
- Written in Java. Provides APIs and web GUI for management and administration

## Key features
- Rich management interface
- Powerful API
- Dynamic workload management
- Secure, configurable and extensible architecture
![architecuture](https://i.imgur.com/J7Jm1XC.png)

## Key components/terminology
- Region
 - Largest Organizational Unit with a CloudStack Deployment
 - Consists of one or more zones
- Zone
 - Consists of one or more pods
 - Contains one or more primary storage servers
 - Consists of a secondary storage
- Pod
 - Contains one or more clusters
 - Contains primary storage servers
- Cluster
 - Consists of a group of identical hosts running a common hypervisor
 - Contins one or more primary storage servers
- Host
 - Smallest orgnizational unit within a Cloustack deployment
 - Has hypervisor to manage the guest VMs
 - Provides the computing resources that run guest VMs.
- Primary storage
iSCSI; NFS; Ceph; Gluster;  Local FileSystem
- Secondary storage
Templates;  ISO images;   Disk volume; snapshots

---
Recap
- Cloud computing is more efficient than traditional IT infrastructure deployment and management
- Cloud can be deployed in public, private and hybrid configurations
- Cloud services can be provided as IaaS, PaaS or SaaS.
- Apache CloudStack is an open source IaaS cloud platform
- Apache CloudStack si very robust, extensible and open platform
