---
title: zabbix
tags:
  - zabbix
  - centos
date: 2016-10-12 15:54:53
---

## zabbix ##

|   IP address     | Roles                        | notes              |
 ----------------- | ---------------------------- | ------------------
 | 192.168.0.49(chatswood)| zabbix server          | Centos 6.8 Zabbix 2.4.8 |
 | 192.168.0.80(macadamina)| zabbix agent          | Centos 6.8 Zabbix 2.4.8 |


 ## server setup ##
 ```sh
 yum install mysql-server -y
 tar xf zabbix-2.4.8.tar.gz
 cd zabbix-2.4.8
 ./configure --prefix=/opt/zabbix --enable-server --enable-agent --with-mysql
 make && make install
 yum install httpd php-devel php php-mbstring -y
 vim /etc/hosts
 192.168.0.80    macadamina
 192.168.0.49    chatswood
 cd /opt/zabbix/
 mv etc{,_bak}
 /bin/cp -rf etc_bak/ etc
 [root@chatswood ~]# cat /opt/zabbix/etc/zabbix_server.conf|egrep -v "^#|^$"
 LogFile=/tmp/zabbix_server.log
 PidFile=/tmp/zabbix_server.pid
 DBSocket=/tmp/mysql.sock
 DBHost=chatswood
 DBName=zabbix
 DBUser=zabbix
 DBPassword=123456
 Timeout=30
 [root@chatswood ~]# cat /opt/zabbix/etc/zabbix_agentd.conf|egrep -v "^#|^$"
 PidFile=/tmp/zabbix_agentd.pid
 LogFile=/tmp/zabbix_agentd.log
 Server=chatswood
 ServerActive=chatswood
 Hostname=chatswood
 Timeout=30

 /etc/init.d/mysqld start
 # mysql
 mysql> create database zabbix character set utf8;
 mysql> grant all on zabbix.* to zabbix@'192.168.0.49' identified by '123456';
 mysql> flush privileges;
 mysql> use zabbix;
 mysql> show databases;
 mysql> exit;
 cd zabbix-2.4.8/database/mysql
 mysql zabbix < ./schema.sql -p
 mysql zabbix < ./images.sql -p
 mysql zabbix < ./data.sql -p
 ## test database
 mysql -u zabbix -p -h192.168.0.49
 mysql> use zabbix;
 mysql> show databases;
 mysql> exit;

 # create a zabbix user
 groupadd -g 508 zabbix
 useradd -g 508 -u 508 zabbix -M -s /sbin/nologin

 /opt/zabbix/sbin/zabbix_server -c /opt/zabbix/etc/zabbix_server.conf
 ps aux|grep zabbix
 /opt/zabbix/sbin/zabbix_agentd -c /opt/zabbix/etc/zabbix_agentd.conf

 # test
 /opt/zabbix/bin/zabbix_get -s chatswood -k proc.num[]

 cd zabbix-2.4.8
 mkdir /var/www/html/zabbix
 /bin/cp -rf frontends/php/* /var/www/html/zabbix/
 vim /etc/php.ini
 date.timezone = Asia/Shanghai
 post_max_size = 18M
 max_execution_time = 350
 max_input_time = 350
 
 yum install php-gd php-xml php-mysql php-bcmath -y
 /etc/init.d/httpd restart
 ps aux|grep apache
 /usr/sbin/ntpdate 3.cn.pool.ntp.org
 chown -R apache /var/www/html/zabbix/
 ```
 ----------
## agent setup ##
```sh
vim /etc/hosts
192.168.0.80    macadamina
192.168.0.49    chatswood
./configure --prefix=/opt/zabbix --enable-agent --with-net-snmp
make && make install
groupadd -g 508 zabbix
useradd -g 508 -u 508 zabbix -M -s /sbin/nologin
cat /opt/zabbix/etc/zabbix_agentd.conf|egrep -v "^#|^$"
LogFile=/tmp/zabbix_agentd.log
Server=chatswood
ServerActive=chatswood
Hostname=macadamina

/opt/zabbix/sbin/zabbix_agentd -c /opt/zabbix/etc/zabbix_agentd.conf
ps aux|grep zabbix
lsof|grep zabbix
```
Access http://192.168.0.49/zabbix,
default username: admin 
password: zabbix

 ----------
## issue ##
----------
checking for mysql_config... /usr/bin/mysql_config
checking for main in -lmysqlclient... no
configure: error: Not found mysqlclient library

#yum -y install mysql-devel

> Written with [StackEdit](https://stackedit.io/).
