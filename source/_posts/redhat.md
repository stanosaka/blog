---
title: redhat
tags: redhat
date: 2017-09-06 17:49:54
---

## Issue 1. ##
``` sh
[root@localhost stan]# yum install -y ntp
Loaded plugins: langpacks, product-id, search-disabled-repos, subscription-manager
This system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.
There are no enabled repos.
 Run "yum repolist all" to see the repos you have.
 You can enable repos with yum-config-manager --enable <repo>
```
## Solve: ##
``` sh
[root@localhost yum.repos.d]# mkdir /cdrom
[root@localhost yum.repos.d]# mount /dev/cdrom /cdrom
[root@localhost yum.repos.d]# pwd
/etc/yum.repos.d
[root@localhost yum.repos.d]# vim dvd.repo
[dvd-source]
name=RHEL 7.2 dvd repo
baseurl=file:///cdrom
enabled=1
gpgcheck=0
[root@localhost yum.repos.d]# yum repolist
Loaded plugins: langpacks, product-id, search-disabled-repos, subscription-manager
This system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.
dvd-source                                                                                     | 4.1 kB  00:00:00
(1/2): dvd-source/group_gz                                                                     | 136 kB  00:00:00
(2/2): dvd-source/primary_db                                                                   | 3.6 MB  00:00:00
repo id                                             repo name                                                   status
dvd-source                                          RHEL 7.2 dvd repo                                           4,620
repolist: 4,620
[root@localhost yum.repos.d]# yum install rpm-build
Loaded plugins: langpacks, product-id, search-disabled-repos, subscription-manager
This system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.
Resolving Dependencies
--> Running transaction check
---> Package rpm-build.x86_64 0:4.11.3-17.el7 will be installed
--> Processing Dependency: perl(Thread::Queue) for package: rpm-build-4.11.3-17.el7.x86_64
--> Processing Dependency: system-rpm-config for package: rpm-build-4.11.3-17.el7.x86_64
--> Running transaction check
---> Package perl-Thread-Queue.noarch 0:3.02-2.el7 will be installed
---> Package redhat-rpm-config.noarch 0:9.1.0-68.el7 will be installed
--> Processing Dependency: dwz >= 0.4 for package: redhat-rpm-config-9.1.0-68.el7.noarch
--> Processing Dependency: perl-srpm-macros for package: redhat-rpm-config-9.1.0-68.el7.noarch
--> Running transaction check
---> Package dwz.x86_64 0:0.11-3.el7 will be installed
---> Package perl-srpm-macros.noarch 0:1-8.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

======================================================================================================================
 Package                          Arch                  Version                       Repository                 Size
======================================================================================================================
Installing:
 rpm-build                        x86_64                4.11.3-17.el7                 dvd-source                144 k
Installing for dependencies:
 dwz                              x86_64                0.11-3.el7                    dvd-source                 99 k
 perl-Thread-Queue                noarch                3.02-2.el7                    dvd-source                 17 k
 perl-srpm-macros                 noarch                1-8.el7                       dvd-source                4.7 k
 redhat-rpm-config                noarch                9.1.0-68.el7                  dvd-source                 77 k

Transaction Summary
======================================================================================================================
Install  1 Package (+4 Dependent packages)

Total download size: 341 k
Installed size: 734 k
Is this ok [y/d/N]: y
Downloading packages:
----------------------------------------------------------------------------------------------------------------------
Total                                                                                 1.9 MB/s | 341 kB  00:00:00
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : perl-Thread-Queue-3.02-2.el7.noarch                                                                1/5
  Installing : dwz-0.11-3.el7.x86_64                                                                              2/5
  Installing : perl-srpm-macros-1-8.el7.noarch                                                                    3/5
  Installing : redhat-rpm-config-9.1.0-68.el7.noarch                                                              4/5
  Installing : rpm-build-4.11.3-17.el7.x86_64                                                                     5/5
dvd-source/productid                                                                           | 1.6 kB  00:00:00
  Verifying  : redhat-rpm-config-9.1.0-68.el7.noarch                                                              1/5
  Verifying  : rpm-build-4.11.3-17.el7.x86_64                                                                     2/5
  Verifying  : perl-srpm-macros-1-8.el7.noarch                                                                    3/5
  Verifying  : dwz-0.11-3.el7.x86_64                                                                              4/5
  Verifying  : perl-Thread-Queue-3.02-2.el7.noarch                                                                5/5

Installed:
  rpm-build.x86_64 0:4.11.3-17.el7

Dependency Installed:
  dwz.x86_64 0:0.11-3.el7                  perl-Thread-Queue.noarch 0:3.02-2.el7  perl-srpm-macros.noarch 0:1-8.el7
  redhat-rpm-config.noarch 0:9.1.0-68.el7

Complete!
```
