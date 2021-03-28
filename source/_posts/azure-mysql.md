---
title: azure mysql
date: 2021-02-25 10:41:47
tags: 
  - azure
  - mysql
  - PaaS
---
# Create a read-only account with azure mysql 8.0
## Notes
1. When creating role and granting privileges, please use ‘%’ instead of ‘localhost’. For PaaS, ‘localhost’ means physical host machine that cannot be accessed in cloud considering security.
2. After creating role and granting role privileges to the target user, please run [SET DEFAULT ROLE ALL TO ‘{username}’]

```
create user 'foobarreader'@'%' identified by 'StrongPassword!';
create role 'foobar_read_only';

grant select on foobardb.* to 'foobar_read_only';

mysql> grant 'foobar_read_only' to 'foobarreader'@'%';

mysql> set default role all to 'foobarreader';

mysql> show grants for 'foobarreader'@'%';

mysql> show grants for 'foobarreader'@'%' using 'foobar_read_only';

## check by login via new created user and test show databases
```

[Mysql 8 role permission control system](https://dev.mysql.com/doc/refman/8.0/en/set-default-role.html "role permission control")

