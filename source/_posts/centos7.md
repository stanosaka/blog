---
title: centos7
tags: centos
date: 2016-08-29 16:18:03
---

##### 1. Stop and Start RHEL7 firewall
```
[root@rhel7 ~]# service firewalld stop
Redirecting to /bin/systemctl stop  firewalld.service

[root@rhel7 ~]# service firewalld start
Redirecting to /bin/systemctl start  firewalld.service
```


##### 2. Disable and Enable RHEL7 firewall
```
[root@rhel7 ~]# systemctl disable firewalld
rm '/etc/systemd/system/dbus-org.fedoraproject.FirewallD1.service'
rm '/etc/systemd/system/basic.target.wants/firewalld.service'


[root@rhel7 ~]# systemctl enable firewalld
ln -s '/usr/lib/systemd/system/firewalld.service' '/etc/systemd/system/dbus-org.fedoraproject.FirewallD1.service'
ln -s '/usr/lib/systemd/system/firewalld.service' '/etc/systemd/system/basic.target.wants/firewalld.service'
```

##### 3. Set hostname
```
[root@rhel7 ~]# hostnamectl

[root@rhel7 ~]# hostnamectl set-hostname nameyoulike
```
