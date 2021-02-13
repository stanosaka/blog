---
title: Install Zabbix Agent
tags: zabbix
date: 2016-08-19 15:41:50
---

is required to install on all remote systems needs to be monitor through Zabbix server. The Zabbix Agent collects resource utilization data and applications data on client system and provide such information to Zabbix server on their requests.

There are two types of checks between Zabbix Server and Client.

- Passive Check : Zabbix Agent sent data to server on their request.
- Active Check : Zabbix Agent sends data periodically to Server.

After installing zabbix server on your server, Now we are moving to install agent on remote system’s. This article will help you to install zabbix agent on CentOS/RHEL 7/6/5 systems. After completing this step go to next article add Host in Zabbix Server.

## Installing Zabbix Agent ##

Follow the below instructions to install Zabbix Agent on CentOS, RHEL 7/6/5 systems.

### Step 1 – Add Required Repository ###
Before installing Zabbix Agent first configure zabbix yum repository using following commands as per your required version and operating system.

    # rpm -ivh http://repo.zabbix.com/zabbix/2.4/rhel/6/x86_64/zabbix-release-2.4-1.el6.noarch.rpm

 

### Step 2 – Install Zabbix Agent ###
After installing yum repository packages in our system. Now use following command to install Zabbix agent on your Linux system.

    # yum install zabbix zabbix-agent


### Step 3 – Edit Zabbix Agent Configuration ###
As zabbix agent has been successfully installed on our remote system. Now we just need to configure zabbix agent by adding zabbix server ip in its configuration file

    vim /etc/zabbix/zabbix_agentd.conf

    Server=[IP of Zabbix Server]
    ServerActive=[IP of Zabbix Server]
    Hostname=[use the FQDN of the node where the agent runs]

### Step 4 – Restarting Zabbix Agent ###
After adding zabbix server ip in configuration file, now restart agent service using below command.

    # /etc/init.d/zabbix-agent start
    # netstat -tulpn|grep zabbix
    
### Step 5 – Test ###
Finally, in order to test if you can reach Zabbix Agent from Zabbix Server, use Telnet command from Zabbix server machine to the IP addresses of the machines that run the agents:

    # telnet zabbix_agent_IP 10050
    
    
### Step 6 – Add Zabbix Agent Monitored Host to Zabbix Server ###

 [read](http://www.tecmint.com/install-and-configure-zabbix-agents-on-centos-redhat-and-debian/ "Step 3: Add Zabbix Agent Monitored Host to Zabbix Server") 
