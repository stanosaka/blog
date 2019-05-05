---
title: backup script
tags: backup
date: 2016-08-28 16:13:25
---

## weekly backup script

```
# vim /etc/backup/backupwk.sh
#!/bin/bash
# ====================================================================
# user input backup location
# basedir=backup dir
basedir=/backup/weekly

# ====================================================================
#
PATH=/bin:/usr/bin:/sbin:/usr/sbin; export PATH
export LANG=C

# backup folders
#named=$basedir/named
#postfixd=$basedir/postfix
#vsftpd=$basedir/vsftp
sshd=$basedir/ssh
#sambad=$basedir/samba
wwwd=$basedir/wwwd
#others=$basedir/others
userinfod=$basedir/userinfo
# check if the file exists, if didn't create a dir
for dirs in $sshd $wwwd $userinfod
do
        [ ! -d "$dirs" ] && mkdir -p $dirs
done

# 1. service config file
#cp -a /var/named/chroot/{etc,var}      $named
#cp -a /etc/postfix /etc/dovecot.conf   $postfixd
#cp -a /etc/vsftpd/*                    $vsftpd
cp -a /etc/ssh/*                        $sshd
#cp -a /etc/samba/*                     $sambad
cp -a /etc/my.cnf       $wwwd
cp -a /application/nginx/conf/nginx.conf        $wwwd

cd /application
  tar -jpc -f $wwwd/tomcat.tar.bz2     tomcat
  tar -jpc -f $wwwd/nginx.tar.bz2     nginx

# 2. about user par
cp -a /etc/{passwd,shadow,group}        $userinfod
cd /var/spool
  tar -jpc -f $userinfod/mail.tar.bz2   mail
cd /var/spool
  tar -jpc -f $userinfod/cron.tar.bz2   cron at
```

Every Sun 3.30 run weekly backup script:
```
# vim /etc/crontab
30 3 * * 0 root /backup/backupwk.sh
```


