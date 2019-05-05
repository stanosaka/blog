---
title: saltstack formula
tags: saltstack
date: 2016-09-18 14:23:21
---

### 1. Get formulat:
```sh
[root@toad formulas]# cd /srv/formulas
    
[root@toad formulas]# git clone https://github.com/saltstack-formulas/nginx-formula
```
### 2. Setup top file:
```sh
[root@toad salt]# cat top.sls
base:
   '*':
       - nginx
```       
Add the new directory to file_roots:
```sh
[root@toad salt]# vim /etc/salt/master
file_roots:
  base:
    - /srv/salt
    - /srv/formulas/tomcat-formula
    - /srv/formulas/mysql-formula
    - /srv/formulas/nginx-formula
```
Restart the Salt Master.

### 3. Run:
```sh
[root@toad formulas]# salt 'macadamina' state.highstate
macadamina:
----------
          ID: nginx
    Function: pkg.installed
      Result: True
     Comment: Package nginx is already installed.
     Started: 11:45:55.019357
    Duration: 3498.284 ms
     Changes:
----------
          ID: /var/www
    Function: file.directory
      Result: True
     Comment: Directory /var/www is in the correct state
     Started: 11:45:58.518393
    Duration: 1.548 ms
     Changes:
----------
          ID: /usr/share/nginx
    Function: file.directory
      Result: True
     Comment: Directory /usr/share/nginx is in the correct state
     Started: 11:45:58.520140
    Duration: 0.699 ms
     Changes:
----------
          ID: /etc/nginx/conf.d/default.conf
    Function: file.absent
      Result: True
     Comment: File /etc/nginx/conf.d/default.conf is not present
     Started: 11:45:58.520983
    Duration: 0.554 ms
     Changes:
----------
          ID: /etc/nginx/conf.d/example_ssl.conf
    Function: file.absent
      Result: True
     Comment: File /etc/nginx/conf.d/example_ssl.conf is not present
     Started: 11:45:58.521693
    Duration: 0.486 ms
     Changes:
----------
          ID: /etc/nginx
    Function: file.directory
      Result: True
     Comment: Directory /etc/nginx is in the correct state
     Started: 11:45:58.522309
    Duration: 1.041 ms
     Changes:
----------
          ID: /etc/nginx/nginx.conf
    Function: file.managed
      Result: True
     Comment: File /etc/nginx/nginx.conf updated
     Started: 11:45:58.524655
    Duration: 132.936 ms
     Changes:
              ----------
              diff:
                  ---
                  +++
                  @@ -41,15 +41,7 @@
                               allow 127.0.0.1;
                               deny all;
                           }
                  -     location / {
                  -             root html;
                  -             index index.jsp index.html index.htm;
                  -             proxy_pass http://web_server;
                  -     }
                       }
                  -    upstream web_server {
                  -     server 127.0.0.1:8080;
                  -     }


                       include /etc/nginx/conf.d/*.conf;
----------
          ID: /etc/nginx/sites-enabled
    Function: file.directory
      Result: True
     Comment: Directory /etc/nginx/sites-enabled is in the correct state
     Started: 11:45:58.657818
    Duration: 1.969 ms
     Changes:
----------
          ID: /etc/nginx/sites-available
    Function: file.directory
      Result: True
     Comment: Directory /etc/nginx/sites-available is in the correct state
     Started: 11:45:58.659985
    Duration: 1.662 ms
     Changes:
----------
          ID: /var/log/nginx/access.log
    Function: file.absent
      Result: True
     Comment: File /var/log/nginx/access.log is not present
     Started: 11:45:58.661843
    Duration: 0.709 ms
     Changes:
----------
          ID: nginx-logger-access
    Function: file.managed
        Name: /etc/init/nginx-logger-access.conf
      Result: True
     Comment: File /etc/init/nginx-logger-access.conf is in the correct state
     Started: 11:45:58.662778
    Duration: 26.583 ms
     Changes:
----------
          ID: nginx-logger-access
    Function: service.running
      Result: True
     Comment: Service nginx-logger-access is already enabled, and is running
     Started: 11:45:58.712813
    Duration: 32.548 ms
     Changes:
              ----------
              nginx-logger-access:
                  True
----------
          ID: /var/log/nginx/error.log
    Function: file.absent
      Result: True
     Comment: Removed file /var/log/nginx/error.log
     Started: 11:45:58.745946
    Duration: 2.21 ms
     Changes:
              ----------
              removed:
                  /var/log/nginx/error.log
----------
          ID: nginx-logger-error
    Function: file.managed
        Name: /etc/init/nginx-logger-error.conf
      Result: True
     Comment: File /etc/init/nginx-logger-error.conf is in the correct state
     Started: 11:45:58.748406
    Duration: 26.15 ms
     Changes:
----------
          ID: nginx-logger-error
    Function: service.running
      Result: True
     Comment: Service nginx-logger-error is already enabled, and is running
     Started: 11:45:58.776206
    Duration: 28.481 ms
     Changes:
              ----------
              nginx-logger-error:
                  True
----------
          ID: /etc/logrotate.d/nginx
    Function: file.absent
      Result: True
     Comment: File /etc/logrotate.d/nginx is not present
     Started: 11:45:58.805766
    Duration: 1.142 ms
     Changes:
----------
          ID: htpasswd
    Function: pkg.installed
        Name: httpd-tools
      Result: True
     Comment: Package httpd-tools is already installed.
     Started: 11:45:58.807193
    Duration: 1.447 ms
     Changes:
----------
          ID: nginx-old-init
    Function: file.rename
        Name: /usr/share/nginx/init.d
      Result: True
     Comment: Source file "/etc/init.d/nginx" has already been moved out of place
     Started: 11:45:58.809509
    Duration: 0.872 ms
     Changes:
----------
          ID: nginx-old-init-disable
    Function: cmd.run
        Name: chkconfig --del nginx
      Result: True
     Comment: onlyif execution failed
     Started: 11:45:58.822906
    Duration: 10.906 ms
     Changes:
----------
          ID: nginx-old-init
    Function: module.wait
        Name: cmd.run
      Result: True
     Comment:
     Started: 11:45:58.835556
    Duration: 1.147 ms
     Changes:
----------
          ID: nginx
    Function: file.managed
        Name: /etc/init/nginx.conf
      Result: True
     Comment: File /etc/init/nginx.conf is in the correct state
     Started: 11:45:58.839171
    Duration: 41.314 ms
     Changes:
----------
          ID: nginx
    Function: service.running
      Result: True
     Comment: Service nginx is already enabled, and is running
     Started: 11:45:58.886379
    Duration: 53.53 ms
     Changes:
              ----------
              nginx:
                  True

Summary
-------------
Succeeded: 22 (changed=5)
Failed:     0
-------------
Total states run:     22

```    
### 4. Test
``` sh
[root@macadamina ~]# /usr/share/nginx/init.d status
nginx (pid  2092) is running...
[root@macadamina ~]# lsof -i :80
COMMAND  PID  USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
nginx   2092  root    6u  IPv4  13922      0t0  TCP localhost:http (LISTEN)
nginx   2093 nginx    6u  IPv4  13922      0t0  TCP localhost:http (LISTEN)
[root@macadamina ~]# wget 127.0.0.1
--2016-09-18 14:19:32--  http://127.0.0.1/
Connecting to 127.0.0.1:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 3698 (3.6K) [text/html]
Saving to: “index.html”

100%[========================================================================================================>] 3,698       --.-K/s   in 0.007s

2016-09-18 14:19:32 (545 KB/s) - “index.html” saved [3698/3698]
[root@macadamina ~]# curl 127.0.0.1
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.1//EN" "http://www.w3.org/TR/xhtml11/DTD/xhtml11.dtd">

<html xmlns="http://www.w3.org/1999/xhtml" xml:lang="en">
    <head>
        <title>Test Page for the Nginx HTTP Server on EPEL</title>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
        <style type="text/css">
            /*<![CDATA[*/
            body {
                background-color: #fff;
                color: #000;
                font-size: 0.9em;
                font-family: sans-serif,helvetica;
                margin: 0;
                padding: 0;
            }
            :link {
                color: #c00;
            }
            :visited {
                color: #c00;
            }
            a:hover {
                color: #f50;
            }
            h1 {
                text-align: center;
                margin: 0;
                padding: 0.6em 2em 0.4em;
                background-color: #294172;
                color: #fff;
                font-weight: normal;
                font-size: 1.75em;
                border-bottom: 2px solid #000;
            }
            h1 strong {
                font-weight: bold;
                font-size: 1.5em;
            }
            h2 {
                text-align: center;
                background-color: #3C6EB4;
                font-size: 1.1em;
                font-weight: bold;
                color: #fff;
                margin: 0;
                padding: 0.5em;
                border-bottom: 2px solid #294172;
            }
            hr {
                display: none;
            }
            .content {
                padding: 1em 5em;
            }
            .alert {
                border: 2px solid #000;
            }

            img {
                border: 2px solid #fff;
                padding: 2px;
                margin: 2px;
            }
            a:hover img {
                border: 2px solid #294172;
            }
            .logos {
                margin: 1em;
                text-align: center;
            }
            /*]]>*/
        </style>
    </head>

    <body>
        <h1>Welcome to <strong>nginx</strong> on EPEL!</h1>

        <div class="content">
            <p>This page is used to test the proper operation of the
            <strong>nginx</strong> HTTP server after it has been
            installed. If you can read this page, it means that the
            web server installed at this site is working
            properly.</p>

            <div class="alert">
                <h2>Website Administrator</h2>
                <div class="content">
                    <p>This is the default <tt>index.html</tt> page that
                    is distributed with <strong>nginx</strong> on
                    EPEL.  It is located in
                    <tt>/usr/share/nginx/html</tt>.</p>

                    <p>You should now put your content in a location of
                    your choice and edit the <tt>root</tt> configuration
                    directive in the <strong>nginx</strong>
                    configuration file
                    <tt>/etc/nginx/nginx.conf</tt>.</p>

                </div>
            </div>

            <div class="logos">
                <a href="http://nginx.net/"><img
                    src="nginx-logo.png"
                    alt="[ Powered by nginx ]"
                    width="121" height="32" /></a>

                <a href="http://fedoraproject.org/"><img
                    src="poweredby.png"
                    alt="[ Powered by Fedora EPEL ]"
                    width="88" height="31" /></a>
            </div>
        </div>
    </body>
</html>
```
