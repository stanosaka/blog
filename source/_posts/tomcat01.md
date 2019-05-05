---
title: tomcat deploy
tags: tomcat
date: 2016-10-27 11:10:37
---

## tomcat deploy on centos ##

## 软件准备
### JDK下载：
```sh
wget --no-check-certificate --no-cookies --header "Cookie: oraclelicense=accept-securebackup-cookie" http://download.oracle.com/otn-pub/java/jdk/7u79-b15/jdk-7u79-linux-x64.tar.gz
```
### [Tomcat下载](http://tomcat.apache.org/):
```sh
wget ftp://toad.cwzhou.win/pub/apache-tomcat-7.0.70.tar.gz
```

## 部署java环境
```sh
tar xf jdk-8u91-linux-x64.tar.gz -C /application/

ln -s /application/jdk1.8.0/ /application/jdk
cp /etc/profile /etc/profile.2016.6.29
# sed -i.ori '$a export JAVA_HOME=/application/jdk\nexport PATH=$JAVA_HOME/bin:$JAVA_HOME/jre/bin:$PATH\nexport CLASSPATH=.$CLASSPATH:$JAVA_HOME/lib:$JAVA_HOME/jre/lib:$JAVA_HOME/lib/tools.jar' /etc/profile

vimdiff /etc/profile /etc/profile.2016.6.29
source /etc/profile
java -version
```

## Install tomcat
```sh
tar xf apache-tomcat-8.0.27.tar.gz -C /application/
ln -s /application/apache-tomcat-8.0.27 /application/tomcat
echo  'export TOMCAT_HOME=/application/tomcat'>>/etc/profile

source /etc/profile
chown -R root.root /application/jdk/ /application/tomcat/
[root@tomcat ~]# tail -4 /etc/profile
export JAVA_HOME=/application/jdk
export PATH=$JAVA_HOME/bin:$JAVA_HOME/jre/bin:$PATH
export CLASSPATH=.$CLASSPATH:$JAVA_HOME/lib:$JAVA_HOME/jre/lib:$JAVA_HOME/lib/tools.jar
export TOMCAT_HOME=/application/tomcat
```
### Tomcat目录介绍
```sh
[root@tomcat ~]# cd /application/tomcat/
[root@tomcat tomcat]# tree -L 1
.
├── bin #→用以启动、关闭Tomcat或者其它功能的脚本（.bat文件和.sh文件）
├── conf #→用以配置Tomcat的XML及DTD文件
├── lib #→存放web应用能访问的JAR包
├── LICENSE
├── logs #→Catalina和其它Web应用程序的日志文件
├── NOTICE
├── RELEASE-NOTES
├── RUNNING.txt
├── temp #→临时文件
├── webapps #→Web应用程序根目录
└── work #→用以产生有JSP编译出的Servlet的.java和.class文件
7 directories, 4 files
[root@tomcat tomcat]# cd webapps/
[root@tomcat webapps]# ll
total 20
drwxr-xr-x 14 root root 4096 Oct 5 12:09 docs #→tomcat帮助文档
drwxr-xr-x 6 root root 4096 Oct 5 12:09 examples #→web应用实例
drwxr-xr-x 5 root root 4096 Oct 5 12:09 host-manager #→管理
drwxr-xr-x 5 root root 4096 Oct 5 12:09 manager #→管理
drwxr-xr-x 3 root root 4096 Oct 5 12:09 ROOT #→默认网站根目录
```

## Tomcat多实例及集群架构
### Tomcat多实例
复制Tomcat目录
``` sh
[root@tomcat ~]# cd /application/
 [root@yoshi application]# cp -a apache-tomcat-7.0.70 tomcat7_1
 [root@yoshi application]# cp -a apache-tomcat-7.0.70 tomcat7_2
```
修改配置文件
``` sh
[root@tomcat application]# mkdir -p /data/www/www/ROOT
[root@tomcat application]# cp /application/tomcat/webapps/memtest/meminfo.jsp /data/www/www/ROOT/
sed -i '22s#8005#8011#;71s#8080#8081#;125s#appBase=".*"# appBase="/data/www/www"#' /application/tomcat7_1/conf/server.xml

sed -i '22s#8005#8012#;71s#8080#8082#;125s#appBase=".*"# appBase="/data/www/www"#' /application/tomcat7_2/conf/server.xml
```
### jvm调优
``` sh
# vim catalina.sh insert
JAVA_OPTS="-Djava.awt.headless=true -Dfile.encoding=UTF-8 -server -Xms1024m -Xmx1024m -XX:NewSize=512m -XX:MaxNewSize=512m -XX:PermSize=512m -XX:MaxPermSize=512m"
```
### service startup script
```sh
cd /etc/init.d

vim tomcat
#!/bin/bash
# des: Tomcat Start Stop Restart
# processname: tomcat
# chkconfig: 234 20 80
JAVA_HOME=/application/jdk
export JAVA_HOME
PATH=$JAVA_HOME/bin:$PATH
export PATH
CATALINA_HOME=/application/tomcat

case $1 in
start)
sh /application/tomcat7_1/bin/startup.sh
sh /application/tomcat7_2/bin/startup.sh
;;
stop)
sh /application/tomcat7_1/bin/shutdown.sh
sh /application/tomcat7_2/bin/shutdown.sh
;;
restart)
sh /application/tomcat7_1/bin/shutdown.sh
sh /application/tomcat7_1/bin/startup.sh
sh /application/tomcat7_2/bin/shutdown.sh
sh /application/tomcat7_2/bin/startup.sh
;;
esac
exit 0

chkconfig --add tomcat
chkconfig --level 234 tomcat on
chkconfig --list tomcat
groupadd tomcat
useradd -g tomcat -d /home/tomcat -s /bin/sh tomcat
cd /etc/init.d/

chown tomcat:tomcat  tomcat

chmod +x tomcat
service tomcat start

check the log:
tail /application/tomcat/logs/catalina.out
```
### Remove default content:
```sh
rm -rf /root/apache-tomcat-7.0.70.tar.gz
vim /tmp/run01.sh
#!/bin/bash
for i in {1..2}
 do
   for j in docs examples host-manager manager ROOT/RELEASE-NOTES.txt 
        do
                rm -rf /application/tomcat7_$i/webapps/$j
        done
 done
```
### Change folder ownership and permissions:
```sh
for i in {1..2}; do chown -R tomcat.tomcat /application/tomcat7_$i; done
for i in {1..2}; do chmod g-w,o-rwx /application/tomcat7_$i; done

vim /tmp/run02.sh
#!/bin/bash
for i in {1..2}
 do
   for j in logs temp 
        do
                chmod o-rwx /application/tomcat7_$i/$j
        done
 done

vim /tmp/run03.sh
#!/bin/bash
for i in {1..2}
 do
   for j in bin webapps conf 
        do
                chmod g-w,o-rwx /application/tomcat7_$i/$j
        done
 done

for i in {1..2}; do chmod 770 /application/tomcat7_$i/conf/catalina.policy; done
vim /tmp/run04.sh
#!/bin/bash
for i in {1..2}
 do
   for j in catalina.properties context.xml logging.properties server.xml tomcat-users.xml web.xml
        do
                chmod g-w,o-rwx /application/tomcat7_$i/conf/$j
        done
 done
```

### Remove Server Banner
```sh
cd /application/tomcat/lib
mkdir -p org/apache/catalina/util/
jar xf catalina.jar org/apache/catalina/util/ServerInfo.properties
ll org/apache/catalina/util/ServerInfo.properties
vim org/apache/catalina/util/ServerInfo.properties
server.info=Secure Web server
server.number=1.0.0.0
server.built=Jan 01 2000 00:00:00 UTC
jar uf catalina.jar org/apache/catalina/util/ServerInfo.properties
rm -rf /application/tomcat/lib/org
```
