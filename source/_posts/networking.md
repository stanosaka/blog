---
layout: blog
title: networking
date: 2019-09-12 18:37:21
tags: networking
---
# IP subnetting
**Subnetting part 1**
IP address
- Subnet address 
- 1st Host address 
- Last Host address 
- Broadcast address 

Two methods
- Method 1 - Binary Method
- Method 2 - Quick Method

![Typical example](https://i.imgur.com/GFUUIuH.png)
- What IP address would router1 be configured with if it is to use the first ip address in the same subnet as PC1?

- What broadcast address is use by PC1?

- What ip address would Router1 be configured with if it is to use the last IP address in the same Subnet as PC1?

- What subnet is PC1 part of?

**Binary Rules**

Network/subnet address 
  - Fill the host portion of an address with binary 0's 

Broadcast address 
  - Fill the host portion of an address with binary 1's 

First Host
  - Fill the host portion of an address with binary 0's except for the last bit which is set to binary 1

Last Host
  - Fill the host portion of an address with binary 1's except for the last bit which is set to binary 0

3rd example
172.16.129.1/17

172.16.1|000 0001.0000 0001
subnet = 172.16.1000 0000.0000 0000=172.16.128.0
Broadcast= 172.16.1111 1111.1111 1111=172.16.255.255
1st host=172.16.128.1
last host=172.16.255.254

**Shortcut table**
128|64|32|16|8|4|2|1
128|192|224|240|248|252|254|255

Tip: the first line shows the decimal values for the binary numbers of an octet
10000000=128   00010000=16

Second row, subtract the value in the frist row from 256
256-128=128    256-32=224   256-64=192  256-1=255

**Binary odometer:(0-255)**
10.1.1.254+1=10.1.1.255
10.1.1.255+1=10.1.2.0
10.1.2.0+1=10.1.2.1
or in reverse:
10.1.2.0-1=10.1.1.255

Broadcast address = Next Network -1
First host = subnet +1
last host = broadcast -1
What subnet is this host on?
What is a last host?


**Subnetting part 2**
creat multiple subnets

Subnet this network into at least 10 subnets
Subnet this network into subnets each having 10 host

# Azure VMs networking
Priority: the lower the number, the higher the priority.

# ipcalc in linux
ipcalc 13.75.0.0/16
Address:   13.75.0.0            00001101.01001011. 00000000.00000000
Netmask:   255.255.0.0 = 16     11111111.11111111. 00000000.00000000
Wildcard:  0.0.255.255          00000000.00000000. 11111111.11111111
=>
Network:   13.75.0.0/16         00001101.01001011. 00000000.00000000
HostMin:   13.75.0.1            00001101.01001011. 00000000.00000001
HostMax:   13.75.255.254        00001101.01001011. 11111111.11111110
Broadcast: 13.75.255.255        00001101.01001011. 11111111.11111111
Hosts/Net: 65534                 Class A
