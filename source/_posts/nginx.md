---
title: optimize nginx performance
tags: 
  - centos
  - nginx
  - Linux
date: 2016-10-17 14:45:29
---

### 1.优化worker进程个数 ###
查看cpu总核数
```sh
# grep processor /proc/cpuinfo|wc -l
2 #表示为1颗CPU四核
```
查看CPU总颗数：
```sh
# grep 'physical id' /proc/cpuinfo|sort|uniq|wc -l
1 #对physical id去重计数，表示1颗cpu
```

```sh
# vim nginx.conf
worker_processes 2;
# /application/nginx/sbin/nginx -t
# /application/nginx/sbin/nginx -s reload
```

### 2.访问日志配置实战 ###
```sh
sed -n '21,23 s/#//gp' ../conf/nginx.conf.default
log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
'$status $body_bytes_sent "$http_referer" '
'"$http_user_agent" "$http_x_forwarded_for"';

vim nginx.con
error_log  logs/error.log;
...
http {
...
log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
		  '$status $body_bytes_sent "$http_referer" '
		  '"$http_user_agent" "$http_x_forwarded_for"';

access_log  logs/access.log  main;
   
...
   }
```
Nginx访问日志轮询切割
```sh
# cat /server/script/cut_nginx_log.sh
#!/bin/sh
Dateformat=`date +%Y%m%d`
Basedir="/application/nginx"
Nginxlogdir="$Basedir/logs"
Logname="access"
[ -d $Nginxlogdir ] && cd $Nginxlogdir || exit 1
[ -f ${Logname}.log ]|| exit1
/bin/mv ${Logname}.log ${Dateformat}_${Logname}.log
$Basedir/sbin/nginx -s reload

cat >> /var/spool/cron/root <<EOF
#cut nginx access log by cwzhou
00 00 * * * /bin/sh /server/script/cut_nginx_log.sh > /dev/null 2>&1
EOF

#check result
#crontab -l
```
### Nginx并发连接数量 ###
> Reference [ngx_http_limit_conn_module](http://nginx.org/en/docs/http/ngx_http_limit_conn_module.html)
1.限制单ip并发连接数
```sh
vim nginx.conf
...
http{
    ...
    limit_conn_zone $binary_remote_addr zone=addr:10m;
    server{
	...
	location / {
		...
		limit_conn addr 1; #限制单ip的并发连接为1
        }
   }
}
```
```sh
ab -c 1 -n 10 http://10.0.0.3/  #模拟并发连接1,访问10次服务器
# -c为并发数 -n为请求书 10.0.0.3为nginx的Ip地址
```
应用场景之一是服务器下载：
```sh
location /download/ {
	limit_conn add 1;
}
```

2.限制虚拟主机总连接数
```sh
vim nginx.conf
...
http{
    ...
    limit_conn_zone $binary_remote_addr zone=addr:10m;
    limit_conn_zone $server_name zone=perserver:10m;

    server{
	...
	location / {
		...
		limit_conn perserver 2; #限制虚拟主机连接数为2
        }
   }
}
```

