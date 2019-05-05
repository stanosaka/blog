---
title: saltstack minion
date: 2016-09-18 17:58:08
tags: saltstack, minion
---
### 1. Install minion:
```sh
[root@chatswood ~]# rpm -ivh http://mirrors.aliyun.com/epel/epel-release-latest-6.noarch.rpm
Retrieving http://mirrors.aliyun.com/epel/epel-release-latest-6.noarch.rpm
warning: /var/tmp/rpm-tmp.nKEP1w: Header V3 RSA/SHA256 Signature, key ID 0608b895: NOKEY
Preparing...                ########################################### [100%]
   1:epel-release           ########################################### [100%]
   
[root@chatswood ~]# yum install -y salt-minion
[root@chatswood ~]# chkconfig salt-minion on
```
### 2. Configuration:
```sh
[root@chatswood ~]# sed -n 16p /etc/salt/minion
#master: salt
[root@chatswood ~]# sed -i '16s#\#master: salt#master: 192.168.0.100#' /etc/salt/minion
[root@chatswood ~]# sed -n 16p /etc/salt/minion
master: 192.168.0.100
[root@chatswood ~]# cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.0.100   toad
192.168.0.168   salt-minion01
192.168.0.80    macadamina
192.168.0.49    chatswood
[root@chatswood ~]# /etc/init.d/salt-minion start
```
On salt master side:
```sh
[root@toad tomcat-formula]# cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.0.100   toad
192.168.0.168   salt-minion01
192.168.0.80    macadamina
192.168.0.49    chatswood
[root@toad tomcat-formula]# service salt-master restart
Stopping salt-master daemon:                               [  OK  ]
Starting salt-master daemon:                               [  OK  ]
[root@toad tomcat-formula]# salt-key
Accepted Keys:
macadamina
salt-minion01
Denied Keys:
Unaccepted Keys:
chatswood
Rejected Keys:
[root@toad tomcat-formula]# salt-key -A
The following keys are going to be accepted:
Unaccepted Keys:
chatswood
Proceed? [n/Y] Y
Key for minion chatswood accepted.
[root@toad tomcat-formula]# salt-key
Accepted Keys:
chatswood
macadamina
salt-minion01
Denied Keys:
Unaccepted Keys:
Rejected Keys:
```
