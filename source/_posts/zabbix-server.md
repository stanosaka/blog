---
title: Install zabbix server on aws ec2
tags: 'zabbix,ec2'
date: 2016-08-19 13:46:10
---

## 1. First install Mysql: ##

    vim install_mysql

### Insert the following lines: ###

    groupadd mysql
    useradd -s /sbin/nologin -g mysql -M mysql
    tail -1 /etc/passwd
    id mysql
    wget ftp://mysql.inspire.net.nz/mysql/Downloads/MySQL-5.5/mysql-5.5.53-linux2.6-x86_64.tar.gz    
    tar xf mysql-5.5.53-linux2.6-x86_64.tar.gz
    mkdir -p /application/
    mv mysql-5.5.53-linux2.6-x86_64 /application/mysql-5.5.53
    ln -s /application/mysql-5.5.53/ /application/mysql
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

### Running command: ###

    source install_mysql
    /application/mysql/bin/mysql_secure_installation
    
    
## 2. Installing repository configuration package ##
Zabbix 2.4 for RHEL6, Oracle Linux 6, CentOS 6:
    #  rpm -ivh http://mirrors.aliyun.com/epel/6/x86_64/epel-release-6-8.noarch.rpm
   ##  rpm -ivh http://repo.zabbix.com/zabbix/2.4/rhel/6/x86_64/zabbix-release-2.4-1.el6.noarch.rpm

Installing Zabbix packages

    # yum install zabbix-server-mysql zabbix-web-mysql

Creating initial database

    shell> mysql -uroot -p
    mysql> create database zabbix character set utf8 collate utf8_bin;
    mysql> grant all privileges on zabbix.* to zabbix@localhost identified by 'zabbix';
    mysql> quit;
    
Import initial schema and data.

    # cd /usr/share/doc/zabbix-server-mysql-2.4.8/create
    # mysql -uroot -p zabbix < schema.sql
    # mysql -uroot -p zabbix < images.sql
    # mysql -uroot -p zabbix < data.sql

Starting Zabbix server process

Edit database configuration in zabbix_server.conf

    # vim /etc/zabbix/zabbix_server.conf
    DBHost=localhost
    DBName=zabbix
    DBUser=zabbix
    DBPassword=zabbix
    DBSocket=/tmp/mysql.sock

Start Zabbix server process.

# service zabbix-server start

Editing PHP configuration for Zabbix frontend

Apache configuration file for Zabbix frontend is located in /etc/httpd/conf.d/zabbix.conf. Some PHP settings are already configured.

    # vim /etc/httpd/conf.d/zabbix.conf
    php_value max_execution_time 300
    php_value memory_limit 128M
    php_value post_max_size 16M
    php_value upload_max_filesize 2M
    php_value max_input_time 300
    php_value date.timezone Asia/Shanghai

It’s necessary to uncomment the “date.timezone” setting and set the right timezone for you. After changing the configuration file restart the apache web server.

    # service httpd restart
    
Zabbix frontend is available at http://zabbix-frontend-hostname/zabbix in the browser. Default username/password is Admin/zabbix.

## 3. Troubleshooting ##
![](http://obsavus1p.bkt.clouddn.com/error02.PNG)

`mkdir -p /var/lib/mysql;
ln -s /tmp/mysql.sock /var/lib/mysql/mysql.sock`

----------

![](http://obsavus1p.bkt.clouddn.com/error03.PNG)
```shell
vim /etc/sysconfig/clock

ZONE="Aisa/Shanghai"

ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
echo '#time sync by stan zhou at `date`'>> /var/spool/cron/root
echo '*/5 * * * * /usr/sbin/ntpdate 3.cn.pool.ntp.org > /dev/null 2>&1' >> /var/spool/cron/root
reboot
```

read more [Changing the Time Zone](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/set-time.html)
----------
checking for mysql_config... /usr/bin/mysql_config
checking for main in -lmysqlclient... no
configure: error: Not found mysqlclient library

#yum -y install mysql-devel
