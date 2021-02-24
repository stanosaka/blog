---
title: azure-mysql
date: 2021-02-25 10:41:47
tags: azure, mysql
---
# Create a read-only account with azure mysql 8.0
## Notes
1. When creating role and granting privileges, please use ‘%’ instead of ‘localhost’. For PaaS, ‘localhost’ means physical host machine that cannot be accessed in cloud considering security.
2. After creating role and granting role privileges to the target user, please run [SET DEFAULT ROLE ALL TO ‘{username}’]

```
create user 'ipscapereader'@'%' identified by 'StrongPassword!';
create role 'ipscape_read_only';

grant select on analytics_50198.* to 'ipscape_read_only';

mysql> grant 'ipscape_read_only' to 'ipscapereader'@'%';

mysql> set default role all to 'ipscapereader';

mysql> show grants for 'ipscapereader'@'%';

mysql> show grants for 'ipscapereader'@'%' using 'ipscape_read_only';

## check by login via new created user and test show databases
```

[Mysql 8 role permission control system](https://dev.mysql.com/doc/refman/8.0/en/set-default-role.html "role permission control")

