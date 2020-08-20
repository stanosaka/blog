---
title: ubuntu
tags: [ubuntu, java]
date: 2016-09-21 23:19:19
---

Setup Java Development Environment in Ubuntu
===================


Ubuntu: ubuntu-16.04-desktop-amd64



Install Oracle Java on Ubuntu Linux
-------------

```sh
# apt-get upgrade -y
# apt-get install ssh -y
```
## 1. Check to see if your Ubuntu Linux operating system architecture is 32-bit or 64-bit ##
```sh
root@stan-virtual-machine:~# file /sbin/init
/sbin/init: symbolic link to /lib/systemd/systemd
root@stan-virtual-machine:~# file /lib/systemd/systemd
/lib/systemd/systemd: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 2.6.32, BuildID[sha1]=03b9d43299696aa6b67b92f1225fa6045b978cb2, stripped
```

## 2. Check if you have Java installed on your system ##
```sh
root@stan-virtual-machine:~# java -version
The program 'java' can be found in the following packages:
 * default-jre
 * gcj-5-jre-headless
 * openjdk-8-jre-headless
 * gcj-4.8-jre-headless
 * gcj-4.9-jre-headless
 * openjdk-9-jre-headless
Try: apt install <selected package>
```

## 3. Completely remove the OpenJDK/JRE from your system and create a directory to hold your Oracle Java JDK/JRE binaries. ##
```sh
root@stan-virtual-machine:~# apt-get purge openjdk-\*
root@stan-virtual-machine:~# mkdir -p /usr/local/java
```
Go to link: http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html
download http://download.oracle.com/otn-pub/java/jdk/8u101-b13/jdk-8u101-linux-x64.tar.gz
```sh
root@stan-virtual-machine:/home/stan/Downloads# cp -r jdk-8u101-linux-x64.tar.gz /usr/local/java

# tar xvzf jdk-8u101-linux-x64.tar.gz
# vim /etc/profile     #add into end of the file
JAVA_HOME=/usr/local/java/jdk1.8.0_101
JRE_HOME=$JAVA_HOME/jre
PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
export JAVA_HOME
export JRE_HOME
export PATH

# update-alternatives --install "/usr/bin/java" "java" "/usr/local/java/jdk1.8.0_101/jre/bin/java" 1
# update-alternatives --install "/usr/bin/javac" "javac" "/usr/local/java/jdk1.8.0_101/bin/javac" 1
# update-alternatives --set java /usr/local/java/jdk1.8.0_101/jre/bin/java
# update-alternatives --set javac /usr/local/java/jdk1.8.0_101/bin/javac
source /etc/profile
```

## 4.Check##
```sh
root@stan-virtual-machine:/usr/local/java# java -version
java version "1.8.0_101"
Java(TM) SE Runtime Environment (build 1.8.0_101-b13)
Java HotSpot(TM) 64-Bit Server VM (build 25.101-b13, mixed mode)
root@stan-virtual-machine:/usr/local/java# javac -version
javac 1.8.0_101
```


-------
