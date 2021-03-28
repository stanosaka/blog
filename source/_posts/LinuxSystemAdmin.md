---
title: Linux System Admin
date: 2019-07-23 09:00:22
tags: 
  - Linux
  - devops
  - SRE
---
# TCP/IP networking

## Troubleshoot networking
**Tools for troubleshooting the network**
- *ping* - ICMP echo requests
![ping -I eth1 192.168.10.12](https://i.imgur.com/RcrCjea.jpg)
- *traceroute* and *tracepath* - Trace the path taken to a given host
- *netcat* - Arbitrary TCP and UDP network communication
```
stan@stan-virtual-machine:~$ ifconfig
eth0      Link encap:Ethernet  HWaddr 00:0c:29:a2:c2:5d  
          inet addr:192.168.199.107  Bcast:192.168.199.255  Mask:255.255.255.0
          inet6 addr: fe80::20c:29ff:fea2:c25d/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:109798 errors:0 dropped:0 overruns:0 frame:0
          TX packets:34545 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:53893408 (53.8 MB)  TX bytes:15357635 (15.3 MB)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:5917 errors:0 dropped:0 overruns:0 frame:0
          TX packets:5917 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:1699426 (1.6 MB)  TX bytes:1699426 (1.6 MB)

stan@stan-virtual-machine:~$ nc -l 8000
nc -v -z 192.168.199.107 8000
Connection to 192.168.199.107 8000 port [tcp/*] succeeded!
```
![nc](https://i.imgur.com/60o3bav.png)
- *tcpdump* and *wireshark* - Packet captures for network analysis
```
 nc -l 8000 &
[1] 20393
➜  ~ sudo tcpdump -i enp2s0 port 8000
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on enp2s0, link-type EN10MB (Ethernet), capture size 262144 bytes
Hi
[1]  + 20393 done       nc -l 8000
09:52:41.674092 IP stan-virtual-machine.lan.57089 > stan-OptiPlex-380.lan.8000: Flags [S], seq 3581133462, win 29200, options [mss 1460,sackOK,TS val 12349350 ecr 0,nop,wscale 7], length 0
09:52:41.674180 IP stan-OptiPlex-380.lan.8000 > stan-virtual-machine.lan.57089: Flags [S.], seq 2419171515, ack 3581133463, win 28960, options [mss 1460,sackOK,TS val 2113949133 ecr 12349350,nop,wscale 7], length 0
09:52:41.674412 IP stan-virtual-machine.lan.57089 > stan-OptiPlex-380.lan.8000: Flags [.], ack 1, win 229, options [nop,nop,TS val 12349351 ecr 2113949133], length 0
09:52:41.674518 IP stan-virtual-machine.lan.57089 > stan-OptiPlex-380.lan.8000: Flags [P.], seq 1:4, ack 1, win 229, options [nop,nop,TS val 12349351 ecr 2113949133], length 3
09:52:41.674541 IP stan-OptiPlex-380.lan.8000 > stan-virtual-machine.lan.57089: Flags [.], ack 4, win 227, options [nop,nop,TS val 2113949133 ecr 12349351], length 0
09:52:41.674582 IP stan-virtual-machine.lan.57089 > stan-OptiPlex-380.lan.8000: Flags [F.], seq 4, ack 1, win 229, options [nop,nop,TS val 12349351 ecr 2113949133], length 0
09:52:41.674718 IP stan-OptiPlex-380.lan.8000 > stan-virtual-machine.lan.57089: Flags [F.], seq 1, ack 5, win 227, options [nop,nop,TS val 2113949133 ecr 12349351], length 0
^C
7 packets captured
8 packets received by filter
1 packet dropped by kernel

# saving captures to a file
➜  ~ sudo tcpdump -i enp2s0 port 8000 -w webserver.pcap

# read a file in binary format
tcpdump -nn -r webserver.pcap

stan@stan-virtual-machine:~$ echo Hi| nc 192.168.199.178 8000
tcpdump -i eth1 port 8000 and not port 22 and not icmp
tcpdump -i eth1 not udp 53
tcpdump -nX -i eth1 port 8000
```
![tcpdump](https://i.imgur.com/D7Vewu7.jpg)
![tcp three-way handshake](https://i.imgur.com/4WIQHtZ.png)

## Backup and streaming
**What to expect from a backup tool?**
- Any backup solution should -roughly- provide the following:
- Full and incrementail backups
- File permissions and ownership preservation
- The ability to be automated

**Introducing rsync**
- Rsync is native Linux tool that can be deployed from the official repositories
- It supports incremental and full backups- It transfers files over SSH
- It can be automated via cron jobs to run in unattended mode.
```
type rsync
rsync -zvr simple-php-website/ ~/backup/
sudo rsync -azv simple-php-website/ ~/backup/ # backup with file created time stampe and ownership
```
**using rsync over the network**
```
rsync -azv simple-php-website/ pi@rpi-01:~/backup/
rsync -azv  pi@rpi-01:~/backup/ simple-php-website
```
**advanced ssh options with rsync**
``` 
ssh-keygen
ssh-copy-id pi@rpi-01 
rsync -avz -e "ssh -p 2222" simple-php-website/ pi@rpi-01:~/backup/ # specify the ssh port
rsync -azv --existing simple-php-website/ pi@rpi-01:~/backup/ ## only sync the file existing in destnation
rsync -avzi simple-php-website/ pi@rpi-01:~/backup/ # i show what has been changed in source and destnation
dd if=/dev/zero of=data.bin bs=102400 count=10240
rsync -azv --progress simple-php-website/ pi@rpi-01:~/backup/ #show the transfers info
rsync -azv --include '*.php' --exclude '*.jpg' simple-php-website/ pi@rpi-01:~/backup/
```

## Performance Analysis
**How to improve performance?**

- The following are general guidelines for achieving a higher performance level on a typical Linux box:
  - Make sure that you have enough memory to serve the running applications
  - Use softwre or hardware load balancing systems. They not only provide faster responses from network applications, but they also provide redundancy should one of the servers go down undexpectedly.
  - Review the application specific documentation and configuration files. Some settings may dramatically boost application performance like turning on caching in webservers or unning multiple instances of a network application.
  - Avoid storage I/O bottlnecks by installing faster disks like SSD's, which do not depend on mechanically moving parts to offer much higher read/write speed than their old counterparts.
  - Use technologies like RAID to distribute I/O evenly on disks (like striping). However, not all applications/databases benefit from striping and RAID and sometimes this my lead to negative results. Application and database vendor and/or documentation should be consulted before moving to RAID.
  - Keep an eye on the network bandwidth and errors to ensure that the bandwidth is not saturated and that the error rate is at the minimum
  
**Possible causes of bottlenecks**
- Hardware-wise, performance is affected mainly by one or more of the following system components: CPU, memory, and disk and network I/O.
- Processes running on the system must access the above components. They compete to have, for example a CPU cycle or an I/O from the disk to read to write data. If the component is busy, the process will have to wait for its trun to be served. This wait time means that the sytem will run slower and implies that are have a performance issue.

**Check your resources**

- Before addressing a performance degradation problem, you must first check your assets to have an estimate of the upper bound for system's general performance level

- The following files provide hardware information:

  - /proc/cpuinfo: take note of the vendor ID, cpu family, model and model name. Each processor core will have a stanza of its own. Useful information can be extracted from the CPU flags like ht which means that the CPU is using the hyper threading technology.
  - /proc/meminfo: details on total, used, and free memeory
  - /proc/diskstats: disk devices statistics
  
- Another useful command for this purpose if dmidecode. This will print a lot of hardware information about the machine like the mothermoard type, BIOS version, installed memory amont many other information.

**Using vmstat to measure CPU utilization**

- When meansuring CPU performance, you may want to determine the overall CPU utilization to know whether or not the overall clock speed is the problem, load averages may also aid you in this. In addition, you may want to check perprocess CPU consumption to know which process really hogging the CPU
- Running vmstat gives you the information you need. It takes the number of seconds and the number of reports as the first and second arguments to determine the number of seconds for which the tool will calculate the averages. The first line of output represents the averages since the systems boot time. The subsequent line present the average per n seconds.
- The right most column is for CPU readings. Us, sy,id, and wa represent the user, system, idle time, and wait time for CPU.
- A high us means that the system is busy doing computational tasks, while a high **sy** time means the system is making a lot of system calls and/or making a lot of I/O requests. A system-typically-should be using no more than 50% in user time, no more than 50% in system time, and have a non-zero idle time.
- The **cs** is short for context switches per interval. That is how many times the kernel switched the running process per interval. The **in** is short for interrupts, it shows the number of interrupts per interval. A high **cs** or **in** rate may be an indication to a malunctioning hardware device.

**CPU load average and per-process**

- Using the **uptime** command, it essentially provides the total time spent since the system was booted, but it also offers a CPU load average for the same period.
- The load average consists of three vlues that represent 5,10, and 15 minutes averages.
- A load average that stays the same on a "good performance" and on a "performance degraded" one is an indication that you have to look elsewhere,perhaps at the network bandwidth, disk I/O, or the intalled memroy.
- Other commands that offer real time view of the CPU per-process load is **ps -aux** and **top**. You may find a single process using more than 50% of the available CPU time. Using **nice** to decrease the execution prioroty of this process may help boost performance.

**Memeory management**

- When an application requests memeory to operate, the kernel offers this memeory in the form of "pages". In linux, a page size is 4KiB.
- The kernel serves those pages from physical storage hardware (either RAM or SWAP space on the disk).
- The kernel shuffles pages between the SWAP space together with RAM. Memroy that is not accessed for a specific period of time is moved into SWAP space (paged) to free more space for rather more frequently accessed memory.
- As more and more processes demand memroy, the kernel tries to fulfil the reqeusts by paging in and out memory pages from and to the SWAP space. And because the disk is the slowest coponent of the system, as the paging rate increates, performance is degraded as processes will have to wait longer before they can have their requested memory and things start to get slower.
- Fainlly, if the system runs out of both physical memory and SWAP space, the kernel resorts to more drastic measures: it kills the least important process with an out-of-memory killer function, a situation that should be avoided at all costs by anticipating the need to install more memroy early enough.

**Using vmstat to measure memory utilization**

- **vmstat** is used the same way it was used to measure CPU utilization.
- The swap in (si) and swap out (so) columns in the SWAP area of the output are of the most importance here. Pages that are read from disk into memory are "swapped in" while those which are ejected by the kernel into the disk are "swapped out". A high rate of si and so may be an indication that the system is using SWAP sapce extensively and that it might need more physical memeory to be installed.
- Such a decision should not be reached by the **si** and **so** rates alone as the system normally does page in and page out operations. Only if is accompanied by slow system response and user complaints.
```
iostat -dx 5 5
```
**A slow system quick diagnosis and remedy**
- If you find that the system is suddenly running slower than before and users start complaining, you can examine the resources discussed in this section for bottlenecks.
- For example, running **ps -auxww** will show you the CPU utilization per process. If you find that a single process is using more than 50% of the CPU ofr a long time, this might be an indication of fault in the process itself. Also check the load average with uptime to determine whether or not the CPU is contended.

- Check the paging activity with vmstat. If there are a lot of page-outs this means the physical memeory is overloaded. Additionalyy, if there is a lot of disk activity without paging this means the a process is extensively using the disk for read and write requests. If this is not the normal behavior (e.g. a database), the process activity should be further examined.
- It is difficult to know exactly which process is using the disk I/O the most, but using kill -STOP to temporarily suspend the susceptiable process can narrow down the possibilities.
- If a process is identified as resource intensive, a number of actions can be taken: if it is CPU intensive you can use the renice command to descrease its priority. You can also ask the user to run it later. I the process is hogging the disk and/or then network, renice will not solve the problem, but you can tune the process itself to optimize its behavior (for example web servers).


Using screen, you can start a session that’s tied to a single operation. Then you can connect or disconnect whenever you want, and come back to the session to check on its progress.
```
screen
screen -r
screen -ls
screen -r num
cat << EOF > foobar
abc foobar
a
b
c
EOF
```
### change default text editor
```
$ sudo update-alternatives --config editor
```

# Linux troubleshooting
## System access troubleshooting
**Server is not reachable**
- Ping the destination server name
  - if server name is not pingable
- Ping the destination server by IP
  - if IP is pingable = Name resolution issue
    - Check /etc/hosts file
    - Check /etc/resolv.conf
    - Check /etc/nsswitch.conf
  - If IP is NOT pingable
    - Ping another server by name and then by IP
    - Checking if your server has an IP address
    - Ping your gateway (netstat -rnv)/modem IP
    - Check physical cable connection
## Resolving configuration issues
- Start in runtime
  - ip addr show
  - ip route show
  - cat /etc/resolv.conf
  - Consider running dhclient
- Analyze persistent
  - /etc/sysconfig/network-scripts/ifcfg*
  - nmcli conn show for more detailed information
## Applying changes to config files
- If you directly modify the config files in network-scipts, use nmcli conn reload to make them effective
- If the network card is already active, use nmcli conn down th0; nmcli con up eth0 to make changes effective

## Testing Connectivity
- Ping a name to test DNS
- Ping the DNS server by IP address to test IP setup adn routing
- Ping the local router if you have routing issues
- Ping another node on the same network to test direct connectivity issues
ping -b 192.168.199.0

ping -f 192.168.199.188

## Sections in PAM
- Where to apply
  - account: user related settings
  - auth: authentication related settings
  - password: updating passwords
  - session: access to different items during a session
- What to do
  - required
  - requisite
  - sufficient
  - optional
  - include
  - substack
  - [vaule1=action value2=action]
```
ldd $(which login)
cd /etc/padm.d/
```

## PAM troubleshooting
- if no appropriate PAM module is found, the other module is used and blocks everything
- Watch /var/log/secure for additional info
- PAM is normally written by the authconfig utilities
- There mark all their changes in /etc/sysconfig/authconfig
- If you manually mess up PAM configuration, restore original configuration using authconfig --updateall, 
it will re-apply all settings in /etc/sysconfig/authconfig
**Cannot connect to a website or an application**
Troubleshoting Steps: ping server by hostname and IP
if NOT pinable= go back to **Server is not reachable**

if pingable = Connect to service 
``` 
telnet 192.168.1.5 80 (http)
``` 
Verify the installation:
``` 
rpm -q haproxy
```
Show the log file which is last written to:
``` 
 cd /var/log
 ls -lrt
``` 
# ssh tunnel

Stan@Company     root@toad.cwzhou.win ||  root@Kali

1. From Kalia connect to Toad
```
root@Kalia:~# ssh -R 2222:localhost:22 root@toad.cwzhou.win

root@Toad:~# while [1]; do date; sleep 300; done

```

2. Stan in Company machine:
```
root@stan:~# ssh root@toad.cwzhou.win
root@toad:~# ssh -p 2222 root@localhost
```
### mount a new disk
```
# mount /dev/xvdc1 /data
vim /etc/fstab
/dev/xvdc1	/data	ext4	defaults     0   0
```
### pass the output of one command as the command-line argument to another
```
find / -name pg_hba.conf|xargs cat
```
### Using journalctl on systemd systems

```
## man system.jounal-fields
## after err level messages
#journalctl -p err
# configured in /etc/systemd/journald.conf
$ logger -p cron.err "I'M ANOTHER PARADOXICAL LOG MESSAGE."

$ sudo journalctl -p err _TRANSPORT=syslog --since 18:00-- Logs begin at Tue 2018-10-09 18:43:07 UTC, end at Wed 2018-10-10 18:11:15 UTC. --Oct 10 18:08:18 centos1 vagrant[1736]: I'M ANOTHER PARADOXICAL LOG MESSAGE.

The options SystemMaxUse and RuntimeMaxUse govern the maximum disk space the journal can use

The SystemKeepFree and RuntimeKeepFree options govern how much disk space journald leaves free for other uses
# journalctl --vacuum-size=2G
```
## Exclude a specific directory in ncdu command
ncdu -x / --exclude /mnt

## repeat currently typed in parameter on bash console
cp /etc/systemd/{journald,journald-bk}.conf

## Lost password
Boot the Ubuntu Live CD.
Press Ctrl-Alt-F1

sudo mount /dev/sda1 /mnt

If you created a custom partition layout when installing Ubuntu you have to find your root partition using the fdisk utility. See the section Finding your root partition.

sudo chroot /mnt

## Print line only if the first field start with string, and copy it to clipboard
```
awk '$1~/Value/' sau2-db-06.txt |cut -d ":" -f 2|xsel -b
## Print the line start with string
awk '/^Variable_name/' eqx1-db-02.txt |cut -d ":" -f 2|xsel -b
```
# System access troubleshooting
## Server is not reachable
- Ping the destination server name
  - if server name is not pingable
- Ping the destination server by IP
  - if IP is pingable = Name resolution issue
     - Check /etc/hosts file
     - Check /etc/resolv.conf
     - Check /etc/nsswitch.conf
  - if IP is NOT pingable
    - Ping another server by name and then by IP
    - Checking if your server has an IP address
    - Ping your gateway/modem ip (check gateway by netstat -rnv)
    - Check physical cable connection
## Cannot connect to a Website or an application
service down or server down?
- Ping server by hostname and IP
  - If not pingable = go back to "server not reachable" lecture
  - If pingable = connect to service
    # tlenet 192.168.1.5 80 (http)
    - if connected = service is running
      Must be sth wrong with the client machine
    - if Not connected = service (http) not running
      Login to the server and start the service (http)
