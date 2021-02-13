---
title: MySQL分库分表备份脚本
tags: [mysql,backup]
date: 2016-08-28 16:46:20
---

Let's cut to the chase
```
[root@aofa-staging tools]# cat mysql_backup.sh
#!/bin/sh
MYUSER=root
MYPASS=yourpasswd
SOCKET=/tmp/mysql.sock
MYLOGIN="mysql -u$MYUSER -p$MYPASS -S $SOCKET"
MYDUMP="mysqldump -u$MYUSER -p$MYPASS -S$SOCKET -B"
DATABASE="$($MYLOGIN -e "show databases;"|egrep -vi "Data|_schema|mysql")"

for dbname in $DATABASE
  do
   MYDIR=/backup/mysql/$dbname
   [ ! -d $MYDIR ] && mkdir -p $MYDIR
 $MYDUMP $dbname|gzip >$MYDIR/${dbname}_$(date +%F).sql.gz
done
```
Output:
```
[root@aofa-staging backup]# tree mysql
mysql
├── allyoubank
│   └── allyoubank_2016-08-28.sql.gz
├── test
│   └── test_2016-08-28.sql.gz
└── wgzy
    └── wgzy_2016-08-28.sql.gz

3 directories, 3 files
```
- - -
```
[root@aofa-staging tools]# cat mysql_table.sh
#!/bin/sh
USER=root
PASSWD=xyz
SOCKET=/tmp/mysql.sock
MYLOGIN="mysql -u$USER -p$PASSWD -S$SOCKET"
MYDUMP="mysqldump -u$USER -p$PASSWD -S$SOCKET"
DATEBASE="$($MYLOGIN -e "show databases;"|egrep -vi "Data|_schema|mysql")"

for dbname in $DATEBASE
do
 TABLE="$($MYLOGIN -e "use $dbname;show tables;"|sed '1d')"
  for tname in $TABLE
   do
MYDIR=/backup/mysql02/$dbname/${dbname}_$(date +%F)
     [ ! -d $MYDIR ] && mkdir -p $MYDIR
 $MYDUMP $dbname $tname |gzip >$MYDIR/${dbname}_${tname}_$(date +%F).sql.gz
    done
done
```
Output:
```
[root@aofa-staging mysql02]# tree
.
├── allyoubank
│   └── allyoubank_2016-08-28
│       ├── allyoubank_account_2016-08-28.sql.gz
│       └── allyoubank_w_sign_in_2016-08-28.sql.gz
└── wgzy
    └── wgzy_2016-08-28
        ├── wgzy_account_2016-08-28.sql.gz
        └── wgzy_w_sign_in_2016-08-28.sql.gz
4 directories, 172 files
```

# my.cnf file
```
# cat /root/.my.cnf
[client]
host=hostname
user=foo
password=xxxxxxxxx



[client_replica]
host=hostname-replica
user=bar
password=xxxxxxxxxx

mysql -e 'show master status\G';
mysql --defaults-group-suffix=_replica -e 'show slave status\G'
```
