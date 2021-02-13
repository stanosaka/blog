---
title: troubleshooting
tags:
---
**What is troubleshooting?"
- Troubleshooting is a bout encounter ing a problem, gathering information about it, analyzing it, and finally fixing it
- Generically, there are two approaches
  - Apply easy-to-implement quick fixes
  - Use a consistent methodology
     - define the issue
     - gather information:look at the relevant log files
      > On current REL-7, systemd is integrated with systemd-journald and you can often find some excellent information
     - form a hypothesis: a best guess 
     - test the hypothesis: test, validate?
     - fix the problem


**Gathering information**
- Use **journalctl** to consult the system journal
  - In-memory only, will clear after reboot
- Use **systemctl status** for an overview of relevant service-related messagages
- Use **rsyslog** for messages written to configuration files
  - Persistent by default

**command journalctl**
```
journalctl  (Shift+G go to the end of file)
journalctl -ef (stands for end an follow)
journalctl _SYSTEMD_UNIT=sshd.service
journalctl -u sshd.service

#use with priorities
journalctl -p emerg..err #p for priority, which will give you message that have a priority between emergency and error
journalctl -xb # is information from the last boot, this is boot information,everything happening from last boot
journalctl --since "2019-09-02 05:02:00" --until "2019-09-02 05:10:00"
journalctl -o verbose
```
**make journalctl persistent**
```
vim /etc/systemd/journal.conf
# look for the default parameter Storage is auto
chown root:systemd-journal /var/log/journal
chmod 2755 /var/log/journal
killall -USR1 systemd-journald
reboot
journalctl
```
So, it is not persistent. So, how are we going to apply the methodology in here? Well, first we need to define the issue. The issue is the journal is not persistent. That's easy enough. Next we need to gather information. Well, if you want to gather information, you need to know a little bit about your stuff. So, what is this about? Well, we go into /etc/systemd and we open journald.conf. And we look for the default parameter Storage is auto. Storage is auto is set, so that's okay, that's not the problem. Next we need the second check, because there's only two things that are required to make the journal persistent. So, /var/log, let's see if we have a journal directory. And, oh no, we don't have a journal directory. So, we can form the hypothesis. The hypothesis is that the journal is not persistent because we don't have the journal directory. So, mkdir journal should probably do it. Oh, and that already brings us to test the hypothesis and fix the problem. On this journal directory of course we need to set appropriate ownership, so chown root :systemd-journal. And we need to set the appropriate permissions. And now we can apply the changes to the journal. Which would start the persistency right now. And after a reboot, we can verify if the solution was correct. So again, we have applied the methodology and we have been quite fast because after the definition of the hypothesis, we haven't really verified it because this was so obvious that instead of verifying the hypothesis by trying different things, we immediately jumped to the solution. Now the only thing that still remains to be done is to verify the solution. So, back in a root shell, journalctl. And we can see that the start time is the same as the start time that we just had. So, verified, solution applied, problem fixed.
## Using Red Hat Resources
- Customer portal is provided at access.redhat.com
- Requires an active Red Hat subscription to anything
  - Tip! Register for a Red Hat developer subscription, it's free
- Knowledgebase contains support articles about bugs, etc

**Using red hat support tools**
- **sosreport** gathers log files, configuration files, and more to send them to Red Hat support
   - Start in interactive mode, by just running sosreport
   - Different options are available, use --help to list
- redhat-support-toll allows subscribers to get information from the Red Hat customer portal from the command line
  - Use the search function to search for information
  - Use **kb 123456** to display information from a specific knowledgebase document
```
## install sosreport
yum install sos
sosreport -l
sosreport -e selinux.list
```
# Using monitoring systems
- Several open source monitoring systems are available and help administrators get an overview of critical problems
  - Nagios
  - Zabbix
  - Performance Co Pilot
  - Cockpit (Red Hat)
  - Landscape (Ubuntu) 
**deploy cockpit**
```
yum install cockpit cockpit-system cockpit-storaged cockpit-pcp
systemctl enable --now cockpit.socket
firewall-cmd --permanent --add-service cockpit
firewall-cmd --reload
```
access by https://ipaddress:9090
**Using performace co pilot**

