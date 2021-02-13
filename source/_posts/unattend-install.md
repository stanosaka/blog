---
title: PXE + kickstart unattend install centos 6
tags: [pxe,kickstart]
date: 2016-08-29 16:25:00
---

### PXE + kickstart unattend install centos 6

##### Step 1: Install and configure DNSMASQ Server

###### 1. INSTALL DNSMASQ DEAMON
```
#yum install dnsmasq
```

###### 2. Edit dnsmasq conf file
```
# mv /etc/dnsmasq.conf  /etc/dnsmasq.conf.backup
# vim /etc/dnsmasq.conf



interface=eth0,lo
#bind-interfaces
domain=centos6.lan
# DHCP range-leases
dhcp-range= eth0,192.168.0.3,192.168.0.253,255.255.255.0,1h
# PXE
dhcp-boot=pxelinux.0,pxeserver,192.168.0.132
# Gateway
dhcp-option=3,192.168.0.1
# DNS
dhcp-option=6,192.168.0.1, 8.8.8.8
server=8.8.4.4
# Broadcast Address
dhcp-option=28,192.168.0.255
# NTP Server
dhcp-option=42,0.0.0.0
pxe-prompt="Press F8 for menu.", 60
pxe-service=x86PC, "Install CentOS 6 from network server 192.168.0.132", pxelinux
enable-tftp
tftp-root=/var/lib/tftpboot
```

interface – Interfaces that the server should listen and provide services.

bind-interfaces – Uncomment to bind only on this interface.

domain – Replace it with your domain name.

dhcp-range – Replace it with IP range defined by your network mask on this segment.

dhcp-boot – Replace the IP statement with your interface IP Address.

dhcp-option=3,192.168.0.1 – Replace the IP Address with your network segment Gateway.

dhcp-option=6,92.168.0.1 – Replace the IP Address with your DNS Server IP – several DNS IPs can be defined.

server=8.8.4.4 – Put your DNS forwarders IPs Addresses.

dhcp-option=28,192.168.0.255 – Replace the IP Address with network broadcast address –optionally.

dhcp-option=42,0.0.0.0 – Put your network time servers – optionally (0.0.0.0 Address is for self-reference).

pxe-prompt – Leave it as default – means to hit F8 key for entering menu 60 with seconds wait time..

pxe=service – Use x86PC for 32-bit/64-bit architectures and enter a menu description prompt under string quotes. Other values types can be: PC98, IA64EFI, Alpha, Arcx86, IntelLeanClient, IA32EFI, BCEFI, XscaleEFI and X86-64EFI.

enable-tftp – Enables the build-in TFTP server.

tftp-root – Use /var/lib/tftpboot – the location for all netbooting files.

read more dnsmasq manual

##### STEP 2: INSTALL SYSLINUX BOOTLOADERS
```
# yum install syslinux
# ls /usr/share/syslinux
```

##### STEP 3: INSTALL TFTP-SERVER AND POPULATE IT WITH SYSLINUX BOOTLOADERS
```
# yum install tftp-server
# cp -r /usr/share/syslinux/* /var/lib/tftpboot
```

##### STEP 4: SETUP PXE SERVER CONFIGURATION FILE

Typically the PXE Server reads its configuration from a group of specific files (GUID files – first, MAC files – next, Default file – last) hosted in a folder called pxelinux.cfg, which must be located in the directory specified in tftp-root statement from DNSMASQ main configuration file.

```
# mkdir /var/lib/tftpboot/pxelinux.cfg
# vim /var/lib/tftpboot/pxelinux.cfg/default
default menu.c32
prompt 0
timeout 300# 30 seconds
ONTIMEOUT local
menu title ########## PXE Boot Menu ##########
label 1
menu label ^1) Install or upgrade CentOS 6 x64 
kernel centos6/vmlinuz
append initrd=centos6/initrd.img ks=ftp://192.168.0.132/ks.cfg
label 2
menu label ^2) Install CentOS 6 x64 with http://mirror.centos.org Repo
kernel centos6/vmlinuz
append initrd=centos6/initrd.img method=http://mirror.centos.org/centos/6/os/x86_64/ devfs=nomount ip=dhcp
label 3
menu label ^3) Install CentOS 6 x64 with Local Repo using VNC
kernel centos6/vmlinuz
append  initrd=centos6/initrd.img method=ftp://192.168.0.132/pub devfs=nomount inst.vnc inst.vncpassword=password
label 4
menu label ^4) Boot from local drive
```

##### STEP 5: ADD CENTOS 7 BOOT IMAGES TO PXE SERVER
```
# mount -o loop /root/CentOS-6.8-x86-bin-DVD1.iso  /mnt
# ls /mnt
# mkdir /var/lib/tftpboot/centos6
# cp /mnt/images/pxeboot/vmlinuz  /var/lib/tftpboot/centos6
# cp /mnt/images/pxeboot/initrd.img  /var/lib/tftpboot/centos6
```

##### STEP 6: CREATE CENTOS 7 LOCAL MIRROR INSTALLATION SOURCE
```
# yum install vsftpd
# cp -r /mnt/*  /var/ftp/pub/
# chmod -R 755 /var/ftp/pub

```
Prepare local repo file
```
# wget -O /var/ftp/pub/Centos-6.repo http://mirrors.aliyun.com/repo/Centos-6.repo
```

##### STEP 7: CREATE KS.CFG FILE
```
# vim /var/ftp/ks.cfg
#platform=x86, AMD64, or Intel EM64T
#version=DEVEL
# Firewall configuration
firewall --disabled
# Install OS instead of upgrade
install
# Use network installation
url --url="ftp://192.168.0.132/pub"
# Root password
rootpw --iscrypted $1$MzJZISZ7$wX0pW3sFy/5y80l2BAQD81
# System authorization information
auth  --useshadow  --passalgo=sha512
# Use graphical install
#graphical
text

%include /tmp/network.ks

firstboot --disable
# System keyboard
keyboard us
# System language
lang en_US
# SELinux configuration
selinux --disabled
# Installation logging level
logging --level=info
# Reboot after installation
#reboot
# System timezone
timezone  Australia/Sydney
# Network information
network  --bootproto=dhcp --device=eth0 --onboot=on
# System bootloader configuration
bootloader --location=mbr
# Clear the Master Boot Record
zerombr
# Partition clearing information
clearpart --all --initlabel 
# Disk partitioning information
part /boot --asprimary --fstype="ext4" --size=250
part swap --asprimary --fstype="swap" --size=1024
part / --asprimary --fstype="ext4" --grow --size=1

%packages
@base
@chinese-support
@compat-libraries
@debugging
@development


%pre
#!/bin/sh
exec < /dev/tty3 > /dev/tty3 2>&1
chvt 3
hn=""

while [ "$hn" == "" ]; do
 clear
 echo " *** Please enter the following details: *** "
 echo
 read -p "Hostname: " hn
done
clear
chvt 1
echo "network --device eth0 --bootproto dhcp --hostname ${hn}" > /tmp/network.ks

%post
sed -i 's#ONBOOT=no#ONEBOOT=yes#g' /etc/sysconfig/network-scripts/ifcfg-eth0
cp /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.old
wget -O /etc/yum.repos.d/CentOS-Base.repo ftp://192.168.0.132/pub/Centos-6.repo
rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY*
yum update -y
sleep 30
runlevel
chkconfig --list|grep 3:on |grep -vE "crond|sshd|network|rsyslog|systat" |awk '{print "chkconfig " $1 " off"}'|bash
/etc/init.d/iptables stop
service ntpd stop
/usr/sbin/ntpdate au.pool.ntp.org
date
echo '#time sync by stan zhou' >> /var/spool/cron/root
echo '*/5 * * * * /usr/sbin/ntpdate au.pool.ntp.org > /dev/null 2>&1' >> /var/spool/cron/root
crontab -l
echo "UseDNS no" >> /etc/ssh/sshd_config
echo '*        -    nofile        65535' >> /etc/security/limits.conf
echo "net.ipv4.tcp_fin_timeout = 2
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_tw_recycle = 1
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_keepalive_time = 600
net.ipv4.ip_local_port_range = 4000 65000
net.ipv4.tcp_max_syn_backlog = 16384
net.ipv4.tcp_max_tw_buckets = 36000
net.ipv4.route.gc_timeout = 100
net.ipv4.tcp_syn_retries = 1
net.ipv4.tcp_synack_retries = 1
net.core.somaxconn = 16384
net.core.netdev_max_backlog = 16384
net.ipv4.tcp_max_orphans = 16384" >> /etc/sysctl.conf
sysctl -p
wget http://download.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
rpm -ivh epel-release-6-8.noarch.rpm
%end
```

Read more about kickstart CentOS6

##### STEP 7: START AND ENABLE DAEMONS SYSTEM-WIDE
```
#service dnsmasq start
#service vsftpd start
#chkconfig dnsmasq on
#chkconfig vsftpd on
```

##### STEP 8: MYSQL INSTALL SCRIPT
```
#vim /var/ftp/pub/install_mysql
groupadd mysql
useradd -s /sbin/nologin -g mysql -M mysql
tail -1 /etc/passwd
id mysql
# setup the tar file download add
wget ftp://192.168.0.134/pub/mysql-5.5.50-linux2.6-x86_64.tar.gz
tar xf mysql-5.5.50-linux2.6-x86_64.tar.gz
mkdir -p /application/
mv mysql-5.5.50-linux2.6-x86_64 /application/mysql-5.5.50
ln -s /application/mysql-5.5.50/ /application/mysql
ls -l /application/
cd /application/mysql/
ls -l support-files/*.cnf
/bin/cp support-files/my-medium.cnf /etc/my.cnf
mkdir -p /application/mysql/data
chown -R mysql.mysql /application/mysql/
/application/mysql/scripts/mysql_install_db --basedir=/application/mysql --datadir=/application/mysql/data/ --user=mysql
/bin/cp support-files/mysql.server /etc/init.d/mysqld
chmod +x /etc/init.d/mysqld
sed -i 's#/usr/local/mysql#/application/mysql#g' /application/mysql/bin/mysqld_safe /etc/init.d/mysqld

/etc/init.d/mysqld start
netstat -lntup|grep mysql

chkconfig --add mysqld
chkconfig mysqld on
chkconfig --list mysqld

echo 'export PATH=/application/mysql/bin:$PATH' >> /etc/profile
tail -1 /etc/profile
source /etc/profile
echo $PATH
# after install run the following command
#/application/mysql/bin/mysql_secure_installation

```

