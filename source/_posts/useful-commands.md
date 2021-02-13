---
title: useful commands
date: 2019-06-19 17:11:17
tags: [linux, commands]
---
```
cp /etc/vsftpd/vsftpd.conf{,.bak}

[root@agent1 ~]# >/etc/motd #删除文件内容


[root@puppetmaster ~]# mv /etc/puppet/autosign.conf{,.bak}   #删除自动注册ACL列表

[root@linux-node1 /]# grep '^[a-z]' /etc/elasticsearch/elasticsearch.yml

sudo tar xfz pycharm-*.tar.gz -C /opt/

nohup & screen


# send a command to be executed on the remote machine and send back the output:
ssh -t user1@server1.packt.co.uk cat /etc/hosts

#use SSH to send files between two machines to or from a remote machine, using the scp command:
scp user1@server1.packt.co.uk:/home/user1/Desktop/file1.txt  ./Desktop/

ssh key-based authentication
ssh-keygen -t rsa -b 2048 -v
ssh-copy-id user1@server1.packt.co.uk

[root@ip-172-31-5-191 ~]# cat /tmp/passwd-truncated
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
[root@ip-172-31-5-191 ~]# cut -d: -f2,3 /tmp/passwd-truncated
x:0
x:1
x:2
x:3
x:4
[root@ip-172-31-5-191 ~]# cut -d: -f2,3 --output-delimiter " " /tmp/passwd-truncated
x 0
x 1
x 2
x 3
x 4
[root@ip-172-31-5-191 ~]# egrep -v '#|^$' /etc/services |head -5 > /tmp/services-truncated
[root@ip-172-31-5-191 ~]# cat /tmp/services-truncated
echo            7/tcp
echo            7/udp
discard         9/tcp           sink null
discard         9/udp           sink null
systat          11/tcp          users
[root@ip-172-31-5-191 ~]# sed  's/ /*/g' /tmp/services-truncated
echo************7/tcp
echo************7/udp
discard*********9/tcp***********sink*null
discard*********9/udp***********sink*null
systat**********11/tcp**********users
[root@ip-172-31-5-191 ~]# awk -F' ' '{print $2" "$3}' /tmp/services-truncated
7/tcp
7/udp
9/tcp sink
9/udp sink
11/tcp users
[root@ip-172-31-5-191 ~]# # tr [CHARACTER_FROM] [CHARACTER_TO]
[root@ip-172-31-5-191 ~]# cat /tmp/services-truncated |tr 'a' 'X'
echo            7/tcp
echo            7/udp
discXrd         9/tcp           sink null
discXrd         9/udp           sink null
systXt          11/tcp          users
[root@ip-172-31-5-191 ~]# cat /tmp/services-truncated |tr '[a-z]' '[A-Z]'
ECHO            7/TCP
ECHO            7/UDP
DISCARD         9/TCP           SINK NULL
DISCARD         9/UDP           SINK NULL
SYSTAT          11/TCP          USERS
[root@ip-172-31-5-191 ~]# awk -F ' ' '{print $1}' /etc/services |egrep -v '^#|^$'| sort| uniq -c| sort -n -k1,1 -r|head
      4 exp2
      4 exp1
      4 discard
      3 v5ua
      3 syslog-tls
      3 sua
      3 ssh
      3 nfsrdma
      3 nfs
      3 megaco-h248

[root@ip-172-31-5-191 ~]# cp /etc/services /tmp/services
[root@ip-172-31-5-191 ~]# gzip /tmp/services
[root@ip-172-31-5-191 ~]# ls -lh /etc/services /tmp/services.gz
-rw-r--r--. 1 root root 655K Jun  7  2013 /etc/services
-rw-r--r--. 1 root root 133K Jun 19 06:01 /tmp/services.gz
[root@ip-172-31-5-191 ~]# gunzip /tmp/services.gz

[root@ip-172-31-5-191 ~]# tar zcf /tmp/archive.tar.gz /home/centos
tar: Removing leading `/' from member names
[root@ip-172-31-5-191 ~]# mkdir /tmp/extracted
[root@ip-172-31-5-191 ~]# tar xvf /tmp/archive.tar.gz -C /tmp/extracted
home/centos/
home/centos/.bash_logout
home/centos/.bash_profile
home/centos/.bashrc
home/centos/.ssh/
home/centos/.ssh/authorized_keys
home/centos/.bash_history
[root@ip-172-31-5-191 ~]# cat /proc/meminfo
MemTotal:        3878872 kB
MemFree:         2586524 kB

[root@ip-172-31-5-191 ~]# free -mh
              total        used        free      shared  buff/cache   available
Mem:           3.7G        716M        2.5G         16M        546M        2.7G
Swap:            0B          0B          0B
[root@ip-172-31-5-191 ~]# df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/xvda1       60G  2.2G   58G   4% /
devtmpfs        1.9G     0  1.9G   0% /dev
tmpfs           1.9G     0  1.9G   0% /dev/shm
tmpfs           1.9G   17M  1.9G   1% /run
tmpfs           1.9G     0  1.9G   0% /sys/fs/cgroup
tmpfs           379M     0  379M   0% /run/user/1000
[root@ip-172-31-5-191 ~]# du -h
4.0K	./.ssh
0	./.pki/nssdb
0	./.pki
52K	.
[root@ip-172-31-5-191 ~]# du -h / --max-depth=1
0	/dev
du: cannot access ‘/proc/14331/task/14331/fd/3’: No such file or directory
du: cannot access ‘/proc/14331/task/14331/fdinfo/3’: No such file or directory
du: cannot access ‘/proc/14331/fd/4’: No such file or directory
du: cannot access ‘/proc/14331/fdinfo/4’: No such file or directory
0	/proc
17M	/run
0	/sys
35M	/etc
52K	/root
641M	/var
4.5M	/tmp
1.3G	/usr
183M	/boot
76K	/home
0	/media
0	/mnt
0	/opt
0	/srv
2.2G	/
root@ip-172-31-5-191 ~]# dd if=/dev/zero of=/tmp/lgig_file.empty bs=1M count=1024
1024+0 records in
1024+0 records out
1073741824 bytes (1.1 GB) copied, 3.61028 s, 297 MB/s
$ su -c 'dd if=/dev/sda1 of=/tmp/img.image'
[root@ip-172-31-5-191 ~]# dd if=/home/centos/.bashrc of=/tmp/.bashrc-copy
0+1 records in
0+1 records out
231 bytes (231 B) copied, 0.000164855 s, 1.4 MB/s
[root@ip-172-31-5-191 ~]# rsync -rav /home/centos/ /tmp/new-centos-home
sending incremental file list
created directory /tmp/new-centos-home
./
.bash_history
.bash_logout
.bash_profile
.bashrc
.ssh/
.ssh/authorized_keys

sent 1,280 bytes  received 169 bytes  2,898.00 bytes/sec
total size is 843  speedup is 0.58

# rsync -rav /home/centos/ stan@192.168.178.300:/tmp
# rsync -rav stan@192.168.178.300:/tmp/stan /tmp/another-copy

[root@ip-172-31-5-191 ~]# telnet google.com 80
Trying 216.58.200.110...
Connected to google.com.
Escape character is '^]'.
^]
HTTP/1.0 400 Bad Request
Content-Type: text/html; charset=UTF-8
Referrer-Policy: no-referrer
Content-Length: 1555
Date: Wed, 19 Jun 2019 06:30:24 GMT

<!DOCTYPE html>
<html lang=en>
  <meta charset=utf-8>
  <meta name=viewport content="initial-scale=1, minimum-scale=1, width=device-width">
  <title>Error 400 (Bad Request)!!1</title>
  <style>
    *{margin:0;padding:0}html,code{font:15px/22px arial,sans-serif}html{background:#fff;color:#222;padding:15px}body{margin:7% auto 0;max-width:390px;min-height:180px;padding:30px 0 15px}* > body{background:url(//www.google.com/images/errors/robot.png) 100% 5px no-repeat;padding-right:205px}p{margin:11px 0 22px;overflow:hidden}ins{color:#777;text-decoration:none}a img{border:0}@media screen and (max-width:772px){body{background:none;margin-top:0;max-width:none;padding-right:0}}#logo{background:url(//www.google.com/images/branding/googlelogo/1x/googlelogo_color_150x54dp.png) no-repeat;margin-left:-5px}@media only screen and (min-resolution:192dpi){#logo{background:url(//www.google.com/images/branding/googlelogo/2x/googlelogo_color_150x54dp.png) no-repeat 0% 0%/100% 100%;-moz-border-image:url(//www.google.com/images/branding/googlelogo/2x/googlelogo_color_150x54dp.png) 0}}@media only screen and (-webkit-min-device-pixel-ratio:2){#logo{background:url(//www.google.com/images/branding/googlelogo/2x/googlelogo_color_150x54dp.png) no-repeat;-webkit-background-size:100% 100%}}#logo{display:inline-block;height:54px;width:150px}
  </style>
  <a href=//www.google.com/><span id=logo aria-label=Google></span></a>
  <p><b>400.</b> <ins>That’s an error.</ins>
  <p>Your client has issued a malformed or illegal request.  <ins>That’s all we know.</ins>
Connection closed by foreign host.

[root@ip-172-31-5-191 ~]# wget -O /tmp/output.txt http://whatthecommit.com/index.txt
--2019-06-19 06:32:13--  http://whatthecommit.com/index.txt
Resolving whatthecommit.com (whatthecommit.com)... 52.86.186.182, 52.200.233.201, 52.202.60.111, ...
Connecting to whatthecommit.com (whatthecommit.com)|52.86.186.182|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 14 [text/plain]
Saving to: ‘/tmp/output.txt’

100%[=============================================================>] 14          --.-K/s   in 0s

2019-06-19 06:32:13 (1.88 MB/s) - ‘/tmp/output.txt’ saved [14/14]

[root@ip-172-31-5-191 ~]# cat /tmp/output.txt
Popping stash
[root@ip-172-31-5-191 ~]# wget -qO- http://whatthecommit.com/index.txt
clarify further the brokenness of C++. why the fuck are we using C++?

root@stan-OptiPlex-380:~|⇒  nc -l -p 9999 < /etc/lsb-release
stan-OptiPlex-380% nc 192.168.199.178 9999 > /tmp/redhat-release
^C
stan-OptiPlex-380% cat /tmp/redhat-release
DISTRIB_ID=LinuxMint
DISTRIB_RELEASE=19
DISTRIB_CODENAME=tara
DISTRIB_DESCRIPTION="Linux Mint 19 Tara"


# links www.duckduckgo.com

[root@ip-172-31-5-191 ~]# echo '(8-(3+1))/4' |bc
1

[root@ip-172-31-5-191 ~]# screen

Ctrl + A + D

[detached from 17091.pts-0.ip-172-31-5-191]
[root@ip-172-31-5-191 ~]# exit
logout
[centos@ip-172-31-5-191 ~]$ exit
logout
Connection to 54.66.232.147 closed.

ssh cicd
Last login: Wed Jun 19 05:39:48 2019 from 220.240.212.9
[centos@ip-172-31-5-191 ~]$ sudo -i
[root@ip-172-31-5-191 ~]# screen -list
There is a screen on:
	17091.pts-0.ip-172-31-5-191	(Detached)
1 Socket in /var/run/screen/S-root.

[root@ip-172-31-5-191 ~]# screen -r 17091.pts-0.ip-172-31-5-191

Type exit for quit
[screen is terminating]
```
---
## various top-like programs
```
iotop ## Get a live view on the input and output, or short I/O, bandwidth usage of your system

iftop ## which gets a live view on network traffic and network bandwidth usage and monitor

htop ## improved version of the normal top program
lsof | grep lib64 ## To print out a list of all open files, which means programs accessing files at the moment
```
---
## Largest files and directories report

```
FS='./';resize;clear;echo "== Server Time: ==";date;echo -e "\n== Filesystem Information: ==";df -PTh ${FS} | column -t;echo -e "\n== Inode Information: ==";df -PTi ${FS} | column -t;echo -e "\n== Largest Directories: ==";du -hcx --max-depth=2 ${FS} 2>/dev/null | grep -P '^([0-9]\.*)*G(?!.*(\btotal\b|\./$))' | sort -rnk1,1 | head -10 | column -t;echo -e "\n== Largest Files: ==";find ${FS} -mount -ignore_readdir_race -type f -exec du {} + 2>&1 | sort -rnk1,1 | head -20 | awk 'BEGIN{ CONVFMT="%.2f";}{ $1=( $1 / 1024 )"M"; print;}' | column -t;echo -e "\n== Largest Files Older Than 30 Days: ==";find ${FS} -mount -ignore_readdir_race -type f -mtime +30 -exec du {} + 2>&1 | sort -rnk1,1 | head -20 | awk 'BEGIN{ CONVFMT="%.2f";}{ $1=( $1 / 1024 )"M"; print; }' | column -t;

== Server Time: ==
Thu Jun 20 04:57:53 UTC 2019

== Filesystem Information: ==
Filesystem  Type  Size  Used  Avail  Use%  Mounted  on
/dev/xvda1  xfs   60G   3.9G  57G    7%    /

== Inode Information: ==
Filesystem  Type  Inodes    IUsed  IFree     IUse%  Mounted  on
/dev/xvda1  xfs   31456704  62835  31393869  1%     /

== Largest Directories: ==

== Largest Files: ==
0.00M  ./.ssh/authorized_keys
0.00M  ./.lesshst
0.00M  ./.bashrc
0.00M  ./.bash_profile
0.00M  ./.bash_logout
0.00M  ./.bash_history
0M     ./stuff/5.txt
0M     ./stuff/4.txt
0M     ./stuff/3.txt
0M     ./stuff/2.txt
0M     ./stuff/1.txt

== Largest Files Older Than 30 Days: ==
0.00M  ./.bashrc
0.00M  ./.bash_profile
0.00M  ./.bash_logout
```
---
## check the port is open or not
```
nmap -vv -n -sS -sU -p443 52.62.248.48/32 | grep "Discovered open port" |awk {'print $6'} | awk -F/ {'print$1'
```
---
## AWS cli commands
List all the ec2 instances details:
```
aws ec2 describe-instances --output text --query 'Reservations[*].Instances[*].[InstanceId, InstanceType, ImageId, State.Name, LaunchTime, Placement.AvailabilityZone, Placement.Tenancy, PrivateIpAddress, PrivateDnsName, PublicDnsName, [Tags[?Key==`Name`].Value] [0][0], [Tags[?Key==`purpose`].Value] [0][0], [Tags[?Key==`environment`].Value] [0][0], [Tags[?Key==`team`].Value] [0][0] ]'}`
```
---
**List server files information**
`FS='./';resize;clear;echo "== Server Time: ==";date;echo -e "\n== Filesystem Information: ==";df -PTh ${FS} | column -t;echo -e "\n== Inode Information: ==";df -PTi ${FS} | column -t;echo -e "\n== Largest Directories: ==";du -hcx --max-depth=2 ${FS} 2>/dev/null | grep -P '^([0-9]\.*)*G(?!.*(\btotal\b|\./$))' | sort -rnk1,1 | head -10 | column -t;echo -e "\n== Largest Files: ==";find ${FS} -mount -ignore_readdir_race -type f -exec du {} + 2>&1 | sort -rnk1,1 | head -20 | awk 'BEGIN{ CONVFMT="%.2f";}{ $1=( $1 / 1024 )"M"; print;}' | column -t;echo -e "\n== Largest Files Older Than 30 Days: ==";find ${FS} -mount -ignore_readdir_race -type f -mtime +30 -exec du {} + 2>&1 | sort -rnk1,1 | head -20 | awk 'BEGIN{ CONVFMT="%.2f";}{ $1=( $1 / 1024 )"M"; print; }' | column -t;`

---
# about file 
**Change the extension of multiple files**
## add an extension for the source files
`for f in *; do mv -- "$f" "${f%.\*}.pdf";done`
## find the file size
```bash
root@stan-OptiPlex-380:~|⇒  du -sh /*
17M     /bin
77M     /boot
4.0K    /cdrom
0       /dev
17M     /etc
1.6G    /home
0       /initrd.img
0       /initrd.img.old
582M    /lib
4.0K    /lib64
16K     /lost+found
4.0K    /media
```

**about node process**
```
cat startApp.sh
#!/bin/sh
export NODE_ENV=production
export DB_PRD_HOST=stantest-postgresql.c3mzoji03zxf.ap-southeast-2.rds.amazonaws.com
export DB_PRD_USER=stantest
export NODE_HOST=localhost
export NODE_PORT=8080
node /myapp/index.js&
exit 0
cat stopApp.sh
#!/bin/sh
kill `ps -axf |grep node |grep -v grep|awk '{print $1}'` | exit 0

```

pgrep: pgrep, pkill - look up or signal processes based on name and other attributes

---
# python

create a random password:
```bash
stan@dockerfordevops:~/Projects/MobyDock$ python
Python 2.7.12 (default, Nov 12 2018, 14:36:49) 
[GCC 5.4.0 20160609] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> import os
>>> import binascii
>>> binascii.b2a_hex(os.urandom(31))
```

# postgres
```
kubernetes-release-1.5 psql -h database -U postgres
psql (11.4 (Ubuntu 11.4-1.pgdg18.04+1), server 9.4.23)
Type "help" for help.

postgres-# \dd
         Object descriptions
 Schema | Name | Object | Description
--------+------+--------+-------------
(0 rows)
```
$ mkdir vagrant_ubuntu_xenial_1 && cd $_

wget https://raw.githubusercontent.com/yogeshraheja/Effective-DevOps-with-AWS/master/Chapter02/helloworld.conf -O scripts/helloworld.conf
