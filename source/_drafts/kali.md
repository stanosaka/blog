---
title: kali
tags:
---
```
root@kali:~# netstat -ant
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State      
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN     
tcp        0      0 10.0.2.15:50596         104.18.102.100:80       TIME_WAIT  
tcp        0      0 10.0.2.15:42524         192.99.200.113:80       TIME_WAIT  
tcp        0      0 10.0.2.15:59636         104.18.103.100:80       TIME_WAIT  
tcp        0      0 10.0.2.15:42528         192.99.200.113:80       TIME_WAIT  
tcp6       0      0 :::80                   :::*                    LISTEN     
tcp6       0      0 :::22                   :::*                    LISTEN     
root@kali:~# iptables -n -L INPUT
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         
root@kali:~# iptables -P INPUT DROP
root@kali:~# iptables -A INPUT -i lo -j ACCEPT
root@kali:~# iptables -n -L INPUT
Chain INPUT (policy DROP)
target     prot opt source               destination         
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0           
root@kali:~# iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
root@kali:~# iptables -A INPUT -m state --state NEW -p tcp --dport 22 -j ACCEPT
root@kali:~# iptables -A INPUT -m state --state NEW -p tcp --dport 80 -j ACCEPT
root@kali:~# iptables -A INPUT -m state --state NEW -p tcp --dport 443 -j ACCEPT
root@kali:~# iptables -n -L INPUT
Chain INPUT (policy DROP)
target     prot opt source               destination         
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0           
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0            state RELATED,ESTABLISHED
ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0            state NEW tcp dpt:22
ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0            state NEW tcp dpt:80
ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0            state NEW tcp dpt:443
```
