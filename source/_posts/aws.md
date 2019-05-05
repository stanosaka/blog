---
title: aws init setup
tags: 'aws,ec2'
date: 2016-08-17 14:07:13
---

#### ASW init setup

Managing User Accounts on Your Linux Instance
```
[ec2-user@ip-172-31-17-193 ~]$ sudo -i
[root@ip-172-31-17-193 ~]# adduser stan
[root@koopa ~]# passwd stan
[root@ip-172-31-17-193 ~]# su - stan
[stan@ip-172-31-17-193 ~]$ mkdir .ssh
[stan@ip-172-31-17-193 ~]$ chmod 700 .ssh
[stan@ip-172-31-17-193 ~]$ vim .ssh/authorized_keys
[stan@ip-172-31-17-193 ~]$ chmod 600 .ssh/authorized_keys
[root@ip-172-31-17-193 ~]# groupadd auegg
[root@ip-172-31-17-193 ~]# usermod -a -G auegg stan
[root@ip-172-31-17-193 ~]# visudo
#add the following line

%auegg ALL=(ALL) ALL
[root@ip-172-31-17-193 ~]# service sshd restart

```

#### ssh to aws without key pairs
```
vim /etc/ssh/sshd_config
PasswordAuthentication yes
Reload ssh daemon
service sshd reload
```

#### set root login
```
vi /root/.ssh/authorized_keys
Delete the lines at the begining of the file until you get to the words ssh-rsa
vim /etc/ssh/sshd_config
PermitRootLogin yes
```
