---
title: web security
tags: [web security,linux]
date: 2016-10-10 11:03:50
---
web安全
===================

### Table of contents

You can insert a table of contents using the marker `[TOC]`:

[TOC]


使用iptables防护网站安全
-------------

> **只允许特定流量通过，禁用其他流量**

> - permit ssh
```sh
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
```
> - permit DNS
```sh
iptables -l INPUT 1 -p tcp --sport 53 -j ACCEPT
iptables -l INPUT 1 -p udp --sport 53 -j ACCEPT
```
> - permit service
```sh
iptables -l INPUT 1 -p tcp --dport 80 -j ACCEPT
```
> - forbid all others
```sh
iptables -A INPUT -j DROP
```

```sh
[root@toad ~]# netstat -tn
Active Internet connections (w/o servers)
Proto Recv-Q Send-Q Local Address               Foreign Address             State
tcp        0      0 192.168.199.231:61094       101.200.31.147:443          TIME_WAIT
tcp        0     64 192.168.199.231:22          60.186.174.98:61241         ESTABLISHED
tcp        0      0 192.168.199.231:41795       192.168.199.1:445           ESTABLISHED
tcp        0      0 192.168.199.231:22          60.186.174.98:50722         ESTABLISHED
tcp        0      0 ::1:19187                   ::1:10010                   TIME_WAIT

[root@toad ~]# netstat -tn|grep 192.168.199.231|awk '{print $5}'|awk -F ":" '{print $1}'|sort|uniq -c|sort -r -n
      2 60.186.174.98
      2 101.200.31.147
      1 192.168.199.1
[root@toad ~]# iptables -l INPUT 1 -s 60.186.174.98 -j DROP
```
----------
AIDE入侵检测系统
-------------
```sh
#install aide
yum install -y aide

#config aide
vim /etc/aide.conf

#init aide database
aide --init
mv /var/lib/aide/aide.db.new.gz /var/lib/aide/aide.db.gz

#check monitor file
aide --check

#update aide database
aide --update

#setup cron job
crontab -e
* 3 * * * /usr/sbin/aide --check|mail -s "AIDE report " sizemore@gmail.com
```


