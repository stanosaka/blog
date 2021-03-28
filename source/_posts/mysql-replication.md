---
title: mysql-replication
date: 2021-02-25 11:20:35
tags: 
  - mysql
  - replication
---
# Myql replicatoin
## Process
1. Clean the salve db
```
DROP_DB_LIST=`mysql -Nse "SET group_concat_max_len = 81920; SELECT GROUP_CONCAT(SCHEMA_NAME SEPARATOR ';\nDROP DATABASE ') FROM information_schema.SCHEMATA WHERE SCHEMA_NAME NOT IN ('mysql','information_schema','performance_schema','sys');"`
echo -e $DROP_DB_LIST > /tmp/delete.sh 
vim /tmp/delete.sh
mysql < /tmp/delete.sh 
```

## dump on the 1st slave 
```
DB_LIST=`mysql -Nse "SELECT GROUP_CONCAT(SCHEMA_NAME SEPARATOR ' ') FROM information_schema.SCHEMATA WHERE SCHEMA_NAME NOT IN ('
mysql','information_schema','performance_schema','sys');"`
mysqldump --dump-slave --apply-slave-statements --include-master-host-port --events --triggers --routines --single-transaction --opt --tz-utc --databases $DB_LIST > mysql_au3_db_02_26Feb2021.sql
```

## restore on the 2nd slave
```
mysql < restore_file.sql
change master to master_host='master_ip_add'
, master_user='replication'
, master_password='strongpasswd'
mysql: show slave status\G;
```

## common problems
### dropping a database with a special character in the databsename
error message:
```
[root@au3-db-03 mysql]# mysql < /tmp/deletenewnew.sh
ERROR 1064 (42000) at line 267: You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near '-123' at line 1
```
fix by:
```
mysql> drop database `supd-123`;
Query OK, 0 rows affected (0.01 sec)
```
