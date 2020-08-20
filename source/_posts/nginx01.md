---
title: Nginx Deploy
tags:
  - nginx
  - centos
date: 2016-10-27 15:31:14
---


## Install Nginx ##
### Install Nginx Package
```sh
[root@huoq bin]# yum install pcre pcre-devel -y
check by:
[root@huoq bin]# rpm -qa pcre pcre-devel
pcre-devel-8.32-15.el7_2.1.x86_64
pcre-8.32-15.el7_2.1.x86_64
[root@huoq tools]# yum install -y openssl-devel openssl
```

### Install Nginx
```sh
wget http://nginx.org/download/nginx-1.6.3.tar.gz
useradd nginx -s /sbin/nologin -M
tar xf nginx-1.6.3.tar.gz
cd nginx-1.6.3/
./configure --user=nginx --group=nginx --prefix=/application/nginx-1.6.3 --with-http_stub_status_module --with-http_ssl_module
make
make install
ln -s /application/nginx-1.6.3 /application/nginx
vim /application/nginx/conf/nginx.conf
worker_processes  4;

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  logs/access.log  main;

    sendfile        on;
   
    keepalive_timeout  65;

    upstream web_pools {
        server 127.0.0.1:8081;
        server 127.0.0.1:8082;
        }

    server {
        listen       80;
        server_name  localhost;

        location / {
            root   html;
            index  index.html index.htm;
            proxy_pass http://web_pools;
        }
        location /nginx_status {
                stub_status on;
                access_log off;
                allow 127.0.0.1;
                deny all;
        }
       
        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
}
/application/nginx/sbin/nginx -t
vim /etc/init.d/nginx
#!/bin/bash
#
#chkconfig: - 85 15
#description: this script use to manage nginx process.
#

#set -x
. /etc/rc.d/init.d/functions

procnum=`ps -ef |grep "/application/nginx/sbin/nginx"|grep -v "grep"|wc -l`

start () {
  if [ "$procnum" -eq 1 -a -f /application/nginx/logs/nginx.pid ]; then
  echo -n "Starting nginx:"
  success
  echo
  else
  /application/nginx/sbin/nginx
  if [ "$?" -eq 0 ]; then
  echo -n "Starting nginx:"
  success
  echo
  else
  echo -n "Starting nginx:"
  failure
  echo
  exit 4
  fi
  fi
}

stop () {
  if [ "$procnum" -eq 1 -a -f /application/nginx/logs/nginx.pid ]; then
  /application/nginx/sbin/nginx -s stop
  if [ "$?" -eq 0 ]; then
  echo -n "Stopping nginx:"
  success
  echo
  else
  echo -n "Stopping nginx:"
  failure
  echo
  exit 3
  fi
  else
  echo -n "Stopping nginx:"
  success
  echo
  fi
}

case $1 in

  start)
  start
  ;;

  stop)
  stop
  ;;

  restart)
  stop
  sleep 1
  start
  ;;

  reload)
  if [ "$procnum" -eq 1 -a -f /application/nginx/logs/nginx.pid ]; then
  /application/nginx/sbin/nginx -s reload
  else
  echo "nginx is not running!please start nginx first..."
  exit 2
  fi
  ;;

  status)
  if [ "$procnum" -eq 1 -a -f /application/nginx/logs/nginx.pid ]; then
  echo "nginx is running..."
  else
  echo "nginx is not running..."
  fi
  ;;

  *)
  echo "Usage : nginx [ start|stop|reload|restart|status ]"
  exit 1
  ;;
esac
chmod +x /etc/init.d/nginx
chkconfig nginx on
/etc/init.d/nginx start
Starting nginx:                                      [  OK  ]
/etc/init.d/nginx status
nginx is running...
```

> Written with [StackEdit](https://stackedit.io/).
