---
title: amazon linux 2
tags: 
  - aws 
  - Linux
date: 2019-05-05 20:46:16
---

### Amazon Linux 2
Change hostname use init-cloud:
```sh
vim /etc/cloud/cloud.cfg.d/11_changehostname.cfg
#cloud-config
preserve_hostname: false
hostname: myhostname
fqdn: myhostname.example.com
manage_etc_hosts: true
```
By Command:
```sh
hostnamectl set-hostname foobar.localdomain
sed -i '1s/localhost/foobar/1' /etc/hosts|sed -i '1s/localhost/foobar/2' /etc/hosts
```

Extra Library:
```sh
[ec2-user ~]$ amazon-linux-extras list
[ec2-user ~]$ sudo amazon-linux-extras install topic=version topic=version
```
