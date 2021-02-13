---
title: system admin tools
tags:
  - system admin
  - centos
date: 2016-10-13 16:23:04
---

## system admin tool ##
[iostat介绍](http://blog.chinaunix.net/uid-10915175-id-3246219.html%22iostat%E4%BB%8B%E7%BB%8D%22)
[全能系统监控工具dstat ](http://blog.chinaunix.net/uid-10915175-id-4543091.html)
most use: 
```sh
[root@toad pxe]# dstat -cmsdnl -D sda3 -N lo,eth0 100 5
```
网络监控工具
本地服务器的网卡吞吐量：
```sh
iptraf -d eth0
```

----------
```sh
# both
wget ftp://ftp.netperf.org/netperf/netperf-2.7.0.tar.gz
tar xf netperf-2.7.0.tar.gz
./configure
make
make install

#服务端：
[root@toad netperf-2.7.0]# netserver
Starting netserver with host 'IN(6)ADDR_ANY' port '12865' and family AF_UNSPEC
[root@toad netperf-2.7.0]# lsof -i :12865
COMMAND     PID USER   FD   TYPE  DEVICE SIZE/OFF NODE NAME
netserver 18230 root    3u  IPv6 2815424      0t0  TCP *:12865 (LISTEN)
[root@toad netperf-2.7.0]# ethtool eth0 |grep -i speed
        Speed: 1000Mb/s

#客户端
root@kali:~/Downloads/netperf-2.7.0#  netperf -H 192.168.199.231 -l 60
MIGRATED TCP STREAM TEST from 0.0.0.0 () port 0 AF_INET to 192.168.199.231 () port 0 AF_INET
Recv   Send    Send
Socket Socket  Message  Elapsed
Size   Size    Size     Time     Throughput
bytes  bytes   bytes    secs.    10^6bits/sec

87380  16384  16384    60.02      94.13
root@kali:~/Downloads/netperf-2.7.0# ethtool eth0 |grep -i speed
        Speed: 100Mb/s
```
Netperf缺省情况下进行TCP批量传输，即-t TCP_STREAM。测试过程中，netperf向netserver发送批量的TCP数据分组，以确定数据传输过程中的吞吐量：
 从netperf的结果输出中，我们可以知道以下的一些信息：
1）	远端系统（即server）使用大小为87380字节的socket接收缓冲
2）	本地系统（即client）使用大小为16384字节的socket发送缓冲
3）	向远端系统发送的测试分组大小为16384字节
4）	测试经历的时间为60秒
5）	吞吐量的测试结果为94.13Mbits/秒
> Reference [netperf 与网络性能测量](https://www.ibm.com/developerworks/cn/linux/l-netperf/)


# useful commands
# Create a configuration file and add your license key \

echo "license_key: a61ac49b8930c041d9d940b8227f5d716819NRAL" | sudo tee -a /etc/newrelic-infra.yml && \

\

# Create the agent’s yum repository \

sudo curl -o /etc/yum.repos.d/newrelic-infra.repo https://download.newrelic.com/infrastructure_agent/linux/yum/el/7/x86_64/newrelic-infra.repo && \

\

# Update your yum cache \

sudo yum -q makecache -y --disablerepo='*' --enablerepo='newrelic-infra' && \

\

# Run the installation script \

sudo yum install newrelic-infra -y
> Written with [StackEdit](https://stackedit.io/).
