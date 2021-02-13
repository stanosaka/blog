---
title: practical jenkins
date: 2019-06-24 23:57:33
tags: [Jenkins, Devops, CD/CI]
---
# Practical Jenkins for Production Deployments
- Installation, configuration, and automation of Jenkins and its dependencies
- Attention to high-availability, monitoring, management, and security
- Utilizing distributed architectures and ability to work with diverse infrastructure platforms
- Understanding and implementing pipeline as code with special attention to multi-branch pipeline
- Being able to deploy code continously
- Integration with extenal services for optimal workflow desgin

# The Development Stages
- Feature branch forked from devlop branch
- Code added/modified and then build/test is executed against it
- On success, the node is merged to the develop branch
- More builds/tests are executed against the develop branch including integration tests
- On success, the code is merged to the master branch
- A tag is created and (optionally) a package is prepared for deployment
- Next setps:
  - The package is deployed to target systems by Jenkins
  - Jenkins triggers an event in another deployment tool
  - Acceptance testing is performed on the target systems (optional)

# What is Continuous Integration?
- It is a development practice
- A new feature is added by forking an integration branch
- The new feature branch is tested before merging the code to the integration branch
- Errors and problems are detected early
- Testing and merging happens automatically without manual intervention
- New code is added, modified, and merged regularly to the integration branch

# What is Continous Delivery?
- It is the practice of getting code into a deployable state
- This is achieved irrespective of the size of the application or number of developers making commits
- Includes new features, changes, bug fixes, etc
- Makes sure that all changes are ready to be deployed at any time
- Makes releases low risk and of higher quality
- Depends on Continous Integration

# What is Continous Deployment?
- It is the process of automatic releases to production
- It is the next step to Continous Delivery
- The level of testing decides how good the release will be 
- Release happends in small batches and continuously
- Gradual product improvement and increase in quality
- The processes of Continous Integration and Continuous Delivery need to be perfect to ensure that releases are without issues
- Relieves developers and administrators from the periodic taks of releasing

# Setting up git, code repositories, and dependencies for Jenkins
## Storage for Jenkins Data
- Setting up a dedicated storage
- Run on physical hardware-rate configuration with multiple disks
- Run on virtualized cloud platform-dedicated elastic volumes
- Home directory - /var/lib/jenkins

Create a new volume in aws ec2 Elastic Block Store, attach volume to ec2 jenkins master instance, run following commmands:
```
fdisk -l
mkfs.ext4 /dev/xvdf
mkdir -p /var/lib/jenkins
vim /etc/fstab
/dev/xvdf /var/lib/jenkins ext4 defaults 0 0
mount /var/lib/jenkins
df -h
yum -y install java-1.8.0-openjdk
```
## Installation of Jenkins from Packages and WAR Files
1. from Packages
```
yum -y install java-1.8.0-openjdk
wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
yum install -y jenkins
systemctl start jenkins
```
If want to change the jenkins' home directory vim /etc/sysconfig/jenkins
JENKINS_HOME=

2. from WAR files
```
yum intall -y tomcat
cd /usr/share/tomcat.webapps/
wget http://mirrors.jenkins.io/war-stable/latest/jenkins.war
systemctl start tomcat
access by ipaddress:8080/jenkins
systemctl stop tomcat
mkdir -p /opt/jenkins_home
chown -R tomcat:tomcat /opt/jenkins_home
vim /etc/tomcat/contxt.xml
# At the end of file add a new line
<Environment name="JENKINS_HOME" vaule="/opt/jenkins_home" type="java.lang.String" />
systemctl start tomcat
```

3. use docker containers
```
yum install -y yum-utils device-mapper-persistent-data lvm2
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum install -y docker-ce
systemctl start docker
docker pull jenkins/jenkins:lts
docker run --name jenkins_master -d -p 8080:8080 -v jenkins_home:/var/jenkins_home jenkins/jenkins:lts
```
## Configuring reverse proxy and setting up user interface for jenkins
Install nginx:
```
rpm -ihv https://dl.fedoraproject.org/pub/epel/7/x86_64/Packages/e/epel-release-7-11.noarch.rpm
yum install -y nginx
vim /etc/nginx/nginx.conf delete server { part }
vim /etc/nginx/conf.d/jenkins.conf 
upstream jenkins {
    server 127.0.0.1:8080;
}

server {
    listen      80 default;
    server_name jenkins.course;

    access_log  /var/log/nginx/jenkins.access.log;
    error_log   /var/log/nginx/jenkins.error.log;

    proxy_buffers 16 64k;
    proxy_buffer_size 128k;

    location / {
        proxy_pass  http://jenkins;
        proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504;
        proxy_redirect off;

        proxy_set_header    Host            $host;
        proxy_set_header    X-Real-IP       $remote_addr;
        proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header    X-Forwarded-Proto $scheme;
    }

}
```
In Jenkins configuration Git plugin put git username and email address
![setupcredentials](https://i.imgur.com/vZuSUgf.png)

## Automating the Jenkins Installation and Configuration Process
- Disabling the manual setup wizard
- Automating the setup wizard steps
- Creating puppet configuration to automate the installation and configuration process
- Automating the process by applying the puppet module
```
systemctl stop jenkins
vim /etc/system/jenkins
modify JENKINS_JAVA_OPTIONS="-Djava.awt.headless=true -Djenkins.install.runSetupWizard=false"
mkdir -p /var/lib/jenkins/init/groovy.d
vim /var/lib/jenkins/init/groovy.d/installPlugins.groovy
#!groovy

import jenkins.model.*
import java.util.logging.Logger

def logger = Logger.getLogger("")
def installed = false
def initialized = false
def pluginParameter = "ws-cleanup timestamper credentials-binding build-timeout antisamy-markup-formatter cloudbees-folder pipeline-stage-view pipeline-github-lib github-branch-source workflow-aggregator gradle ant mailer email-ext ldap pam-auth matrix-auth ssh-slaves github git"

def plugins = pluginParameter.split()
logger.info("" + plugins)
def instance = Jenkins.getInstance()
def pm = instance.getPluginManager()
def uc = instance.getUpdateCenter()
plugins.each {
  logger.info("Checking " + it)
  if (!pm.getPlugin(it)) {
    logger.info("Looking UpdateCenter for " + it)
    if (!initialized) {
      uc.updateAllSites()
      initialized = true
    }
    def plugin = uc.getPlugin(it)
    if (plugin) {
      logger.info("Installing " + it)
        def installFuture = plugin.deploy()
      while(!installFuture.isDone()) {
        logger.info("Waiting for plugin install: " + it)
        sleep(3000)
      }
      installed = true
    }
  }
}
if (installed) {
  logger.info("Plugins installed, initializing a restart!")
  instance.save()
  instance.restart()
}
vim /var/lib/jenkins/init/groovy.d/security.groovy
#!groovy

import jenkins.model.*
import hudson.security.*
import jenkins.security.s2m.AdminWhitelistRule
import hudson.security.csrf.DefaultCrumbIssuer
import jenkins.model.Jenkins

def instance = Jenkins.getInstance()

def hudsonRealm = new HudsonPrivateSecurityRealm(false)
hudsonRealm.createAccount("admin", "admin")
instance.setSecurityRealm(hudsonRealm)

def strategy = new FullControlOnceLoggedInAuthorizationStrategy()
strategy.setAllowAnonymousRead(false)
instance.setAuthorizationStrategy(strategy)
instance.save()

Jenkins.instance.getInjector().getInstance(AdminWhitelistRule.class)

def j = Jenkins.instance
if(j.getCrumbIssuer() == null) {
    j.setCrumbIssuer(new DefaultCrumbIssuer(true))
    j.save()
    println 'CSRF Protection configuration has changed.  Enabled CSRF Protection.'
}
else {
    println 'Nothing changed.  CSRF Protection already configured.'
}
chown -R jenkins:jenkins /var/lib/jenkins/init/groovy.d/*.groovy
systemctl start jenkins
tail -f /var/log/jenkins/jenkins.log
```
## install puppet
go to [puppet](http://yum.puppetlabs.com/), copy url link [puppetlabs-release-pc1-el-7.noarch.rpm](http://yum.puppetlabs.com/puppetlabs-release-pc1-el-7.noarch.rpm).
on the sever install the package by the command:
```
rpm -ihv http://yum.puppetlabs.com/puppetlabs-release-pc1-el-7.noarch.rpm

yum -y install puppet-agent
```

```
mkdir puppet
cd puppet
mkdir {modules, manifests}
mkdir modules/jenkins
mkdir modules/jenkins/{manifests,files}
vim modules/jenkins/manifests.install.pp
class jenkins::install {
    file { 'jenkins_repo':
        path   => '/etc/yum.repos.d/jenkins.repo',
        source => 'https://pkg.jenkins.io/redhat-stable/jenkins.repo',
        ensure => present,
        mode   => '0644'
    }

    exec { 'jenkins_repo_key':
        command   => '/bin/rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key',
        unless    => '/bin/rpm -q jenkins',
        subscribe => File['jenkins_repo'],
        require   => File['jenkins_repo']
    }

    package { 'epel-repo':
        name   => 'epel-release',
        ensure => present,
        source => 'https://dl.fedoraproject.org/pub/epel/7/x86_64/Packages/e/epel-release-7-11.noarch.rpm'
    }

    $packages = [ 'jenkins', 'java-1.8.0-openjdk', 'nginx', 'git' ]

    package { $packages:
        ensure    => present,
        require   => [ File['jenkins_repo'], Package['epel-repo'] ]
    }
}

vim modules/jenkins/cofig.pp
class jenkins::config {
    file { 'groovy_script_directory':
        path   => '/var/lib/jenkins/init.groovy.d',
        owner  => 'jenkins',
        group  => 'jenkins',
        mode   => '0755',
        ensure => directory
    }

    file { 'security_groovy_script':
        path    => '/var/lib/jenkins/init.groovy.d/security.groovy',
        owner   => 'jenkins',
        group   => 'jenkins',
        source  => 'puppet:///modules/jenkins/security.groovy',
        mode    => '0644',
        require => File['groovy_script_directory']
    }

    file { 'plugins_groovy_script':
        path    => '/var/lib/jenkins/init.groovy.d/installPlugins.groovy',
        owner   => 'jenkins',
        group   => 'jenkins',
        source  => 'puppet:///modules/jenkins/installPlugins.groovy',
        mode    => '0644',
        require => File['groovy_script_directory']
    }

    file { 'nginx_config_jenkins':
        path   => '/etc/nginx/conf.d/jenkins.conf',
        owner  => 'root',
        group  => 'root',
        source => 'puppet:///modules/jenkins/jenkins.conf',
        mode   => '0644'
    }

    file { 'nginx_config':
        path   => '/etc/nginx/nginx.conf',
        owner  => 'root',
        group  => 'root',
        source => 'puppet:///modules/jenkins/nginx.conf',
        mode   => '0644'
    }

    file { 'jenkins_sysconfig':
        path   => '/etc/sysconfig/jenkins',
        owner  => 'root',
        group  => 'root',
        source => 'puppet:///modules/jenkins/jenkins',
        mode   => '0644'
    }
}

vim modules/jenkins/service.pp
class jenkins::service {
    service { 'jenkins':
        ensure => running
    }

    service { 'nginx':
        ensure => running
    }
}

vim modules/jenkins/init.pp
class jenkins {
    include jenkins::install
    include jenkins::config
    include jenkins::service

    Class['jenkins::install'] -> Class['jenkins::config'] ~> Class['jenkins::service']
}

vim modules/jenkins/files/installPlugins.groovy
#!groovy

import jenkins.model.*
import java.util.logging.Logger

def logger = Logger.getLogger("")
def installed = false
def initialized = false
def pluginParameter = "ws-cleanup timestamper credentials-binding build-timeout antisamy-markup-formatter cloudbees-folder pipeline-stage-view pipeline-github-lib github-branch-source workflow-aggregator gradle ant mailer email-ext ldap pam-auth matrix-auth ssh-slaves github git"

def plugins = pluginParameter.split()
logger.info("" + plugins)
def instance = Jenkins.getInstance()
def pm = instance.getPluginManager()
def uc = instance.getUpdateCenter()
plugins.each {
  logger.info("Checking " + it)
  if (!pm.getPlugin(it)) {
    logger.info("Looking UpdateCenter for " + it)
    if (!initialized) {
      uc.updateAllSites()
      initialized = true
    }
    def plugin = uc.getPlugin(it)
    if (plugin) {
      logger.info("Installing " + it)
        def installFuture = plugin.deploy()
      while(!installFuture.isDone()) {
        logger.info("Waiting for plugin install: " + it)
        sleep(3000)
      }
      installed = true
    }
  }
}
if (installed) {
  logger.info("Plugins installed, initializing a restart!")
  instance.save()
  instance.restart()
}

vim modules/jenkins/files/jenkins.conf
upstream jenkins {
    server 127.0.0.1:8080;
}

server {
    listen      80 default;
    server_name jenkins.course;

    access_log  /var/log/nginx/jenkins.access.log;
    error_log   /var/log/nginx/jenkins.error.log;

    proxy_buffers 16 64k;
    proxy_buffer_size 128k;

    location / {
        proxy_pass  http://jenkins;
        proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504;
        proxy_redirect off;

        proxy_set_header    Host            $host;
        proxy_set_header    X-Real-IP       $remote_addr;
        proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header    X-Forwarded-Proto $scheme;
    }

}

vim modules/jenkins/files/nginx.conf
# For more information on configuration, see:
#   * Official English Documentation: http://nginx.org/en/docs/
#   * Official Russian Documentation: http://nginx.org/ru/docs/

user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

# Load dynamic modules. See /usr/share/nginx/README.dynamic.
include /usr/share/nginx/modules/*.conf;

events {
    worker_connections 1024;
}

http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 2048;

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;

    # Load modular configuration files from the /etc/nginx/conf.d directory.
    # See http://nginx.org/en/docs/ngx_core_module.html#include
    # for more information.
    include /etc/nginx/conf.d/*.conf;

}

vim modules/jenkins/files/security.groovy
#!groovy

import jenkins.model.*
import hudson.security.*
import jenkins.security.s2m.AdminWhitelistRule
import hudson.security.csrf.DefaultCrumbIssuer
import jenkins.model.Jenkins

def instance = Jenkins.getInstance()

def hudsonRealm = new HudsonPrivateSecurityRealm(false)
hudsonRealm.createAccount("admin", "admin")
instance.setSecurityRealm(hudsonRealm)

def strategy = new FullControlOnceLoggedInAuthorizationStrategy()
strategy.setAllowAnonymousRead(false)
instance.setAuthorizationStrategy(strategy)
instance.save()

Jenkins.instance.getInjector().getInstance(AdminWhitelistRule.class)

def j = Jenkins.instance
if(j.getCrumbIssuer() == null) {
    j.setCrumbIssuer(new DefaultCrumbIssuer(true))
    j.save()
    println 'CSRF Protection configuration has changed.  Enabled CSRF Protection.'
}
else {
    println 'Nothing changed.  CSRF Protection already configured.'
}
[root@ip-172-31-2-108 ~]# tree puppet/
puppet/
├── manifests
│   └── site.pp
└── modules
    └── jenkins
        ├── files
        │   ├── installPlugins.groovy
        │   ├── jenkins
        │   ├── jenkins.conf
        │   ├── nginx.conf
        │   └── security.groovy
        └── manifests
            ├── config.pp
            ├── init.pp
            ├── install.pp
            └── service.pp
cd puppet
puppet apply --modulepath ./modules manifests/site.pp
```
## Creating build jobs from user interface and scripts
1.Create a new freestyle jenkins job called python-job, configure 'Source Code Management"->'Git'-> Repo URL git@github.com:szhouchoice/python-project.git.
"Build" -> Execute shell, command `python *test.py`

2.`mkdir jenkins_cmd; cd jenkins_cmd`
```
cp /var/lib/jenkins/jobs/python-job/config.xml .
vim config.xml
<publishers>
    <hudson.plugins.ws__cleanup.WsCleanup plugin="ws-cleanup@0.34">
      <patterns class="empty-list"/>
      <deleteDirs>true</deleteDirs>
      <skipWhenFailed>false</skipWhenFailed>
      <cleanWhenSuccess>true</cleanWhenSuccess>
      <cleanWhenUnstable>true</cleanWhenUnstable>
      <cleanWhenFailure>true</cleanWhenFailure>
      <cleanWhenNotBuilt>true</cleanWhenNotBuilt>
      <cleanWhenAborted>true</cleanWhenAborted>
      <notFailBuild>false</notFailBuild>
      <cleanupMatrixParent>false</cleanupMatrixParent>
      <externalDelete></externalDelete>
    </hudson.plugins.ws__cleanup.WsCleanup>
curl -s 'http://localhost:8080/crumbIssuer/api/xml?xpath=concat(//crumbRequestField,":",//crumb)' -u admin:admin

curl -s -XPOST 'http://localhost:8080/createItem?name=python-project-new' -u admin:admin --data-binary @config.xml -H "Jenkins-Crumb:384576fea44a8984fd1afe2474c8e951" -H "Content-Type:text/xml"
```
# High-availability, monitoring, and management of jenkins deployments
- High-availability scenario of Jenkins ecosystem and support
- Creating multiple Jenkins master nodes with shared data directory
- Configuring HAProxy as a load balancer for the Jenkins masters
- Testing the setup by failing nodes

## Setting up multiple Jenkins Master with Load Balancer for high-availability
![High-Availability Setup](https://i.imgur.com/YnYwCzR.png)
### 1. On aws ec2 create 3 instances with Centos 7 OS. 1 for haproxy 2 for jenkins
### 2. Create a efs on aws, setup up efs on two jenkins ec2 instances. efs security group need create a new one open nfs port to everywhere
```
yum -y install nfs-utils
vim /etc/fstab
ap-southeast-2a.fs-foobar.efs.ap-southeast-2.amazonaws.com:/ /var/lib/jenkins/jobs nfs defaults 0 0
systemctl stop jenkins
mount /var/lib/jenkins/jobs
df -h
chown -R jenkins:jenkins /var/lib/jenkins/jobs
ls -l /var/lib/jenkins/
systemctl start jenkins
```
### 3. On Jenkins master passive node
``` 
vim /opt/jenkins_reload.sh
#!/bin/bash
crumb_id=$(curl -s 'http://localhost:8080/crumbIssuer/api/xml?xpath=concat(//crumbRequestField,":",//crumb)' -u admin:admin)
curl -s -XPOST 'http://localhost:8080/reload' -u admin:admin -H "$crumb_id"
chmod +x /opt/jenkins_reload.sh
vim /etc/cron.d/jenkins_reload.sh
*/1 * * * * root /bin/bash /opt/jenkins_reload.sh #run every min
```
### 4. On HA Proxy server
```
yum -y install haproxy
cat /dev/null> /etc/haproxy/haproxy.cfg
vim /etc/haproxy/haproxy.cfg
defaults
  log  global
  maxconn  2000
  mode  http
  option  redispatch
  option  forwardfor
  option  http-server-close
  retries  3
  timeout  http-request 10s
  timeout  queue 1m
  timeout  connect 10s
  timeout  client 1m
  timeout  server 1m
  timeout  check 10s

frontend ft_jenkins
  bind *:80
  default_backend  bk_jenkins
  reqadd  X-Forwarded-Proto:\ http

backend bk_jenkins
 server jenkins1 172.31.2.108:8080 check
 server jenkins2 172.31.2.132:8080 check backup

systemctl start haproxy
ps-ef|grep haproxy
```
## Backing up and restoring Jenkins data
### Steps of backup and resotre
- Decide on what to back up (specific directories or everything)
- Set up a dedicated local directory where backups would be stored
- Decide the archiving method to use for the backup (.zip, tar.gz, and so on)
- Set up a schedule for backups to take place
- Configure a highly-available remove location where old backups can be transferred for archiving
- Keep a certain number of the most recent backups to be readily available for restores
- When restoring, unarchive the most recent backup and copy over files
### Backup and Restore Methods
- Use periodic Backup plugin available in Jenkins
- Manually copy and archive files using scripts
- Select a remote location for storing data such as S3, EFS, and so on
- Use local tools such as scp, rsync, s3cmd, and son on to transfer data
- Use specialized open source tools such as Amanda or Bacula to perform backups or enterprise tools such as Netbackup

```
mkdir -p /opt/jenkins_backups
chown -R jenkins:jenkins /opt/jenkins_backups/
```
Install plugin called Periodic Backup Manager, configure it as following:
![periodicbackupconf](https://i.imgur.com/fvgXK7C.png)

click button Backup Now!

## Monitoring Jenkins components and data
### Install plugin monitoring, after that under 'Manage Jenkins' will have Monitoring of Jenkins master:
![Monitoring](https://i.imgur.com/88b0cnX.png)

### install configure graphite grafana
```
#Install EPEL repo
rpm -ihv https://dl.fedoraproject.org/pub/epel/7/x86_64/Packages/e/epel-release-7-11.noarch.rpm

#Install Graphite and dependencies
yum -y install graphite-web mysql mariadb-server MySQL-python net-tools mlocate wget python-carbon python-pip gcc python-devel libffi python-cffi cairo cairo-devel fontconfig freetype*

#Start database server
systemctl start mariadb

#Configure database and perms
mysql -e "CREATE DATABASE graphite;"
mysql -e "GRANT ALL PRIVILEGES ON graphite.* TO 'graphite'@'localhost' IDENTIFIED BY 'j3nk1nsdb';"
mysql -e 'FLUSH PRIVILEGES;'

#Edit file
vi /etc/graphite-web/local_settings.py

#Add content
DATABASES = {
    'default': {
        'NAME': 'graphite',
        'ENGINE': 'django.db.backends.mysql',
        'USER': 'graphite',
        'PASSWORD': 'j3nk1nsdb',
        'HOST': 'localhost',
        'PORT': '3306'
    }
}

#Change perms
chown -R apache:apache /var/lib/graphite-web /usr/share/graphite/

#Edit file
vi /etc/httpd/conf.d/graphite-web.conf

#Add content

<VirtualHost *:80>
    ServerName graphite-web
    DocumentRoot "/usr/share/graphite/webapp"
    ErrorLog /var/log/httpd/graphite-web-error.log
    CustomLog /var/log/httpd/graphite-web-access.log common

    # Header set Access-Control-Allow-Origin "*"
    # Header set Access-Control-Allow-Methods "GET, OPTIONS"
    # Header set Access-Control-Allow-Headers "origin, authorization, accept"
    # Header set Access-Control-Allow-Credentials true

    WSGIScriptAlias / /usr/share/graphite/graphite-web.wsgi
    WSGIImportScript /usr/share/graphite/graphite-web.wsgi process-group=%{GLOBAL} application-group=%{GLOBAL}

    <Location "/content/">
        SetHandler None
    </Location>

    Alias /media/ "/usr/lib/python2.7/site-packages/django/contrib/admin/media/"
    <Location "/media/">
        SetHandler None
    </Location>

   <Directory "/usr/share/graphite/">
    Require all granted
    Order allow,deny
    Allow from all
   </Directory>
</VirtualHost>

#Edit file
vi /etc/graphite-web/local_settings.py

#Add content
SECRET_KEY = 'jenkinsapp'

#Run command
/usr/lib/python2.7/site-packages/graphite/manage.py syncdb
# user admin password admin

#Edit file
vi /etc/yum.repos.d/grafana.repo

#Add content
[grafana]
name=grafana
baseurl=https://packagecloud.io/grafana/stable/el/6/$basearch
repo_gpgcheck=1
enabled=1
gpgcheck=1
gpgkey=https://packagecloud.io/gpg.key https://grafanarel.s3.amazonaws.com/RPM-GPG-KEY-grafana
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt

#Install grafana
yum -y install grafana

#Start services
systemctl start carbon-cache
systemctl start httpd
systemctl start grafana-server
```
1.Install jenkins plugin call 'Metrics Graphite Reporting'
2.Go to Manage Jenkins -> Configure Sytem-> Graphite metrics report input Hostname ipaddress Port 2003 Prefix jenkins.
3.Access grafana server ipaddress:3000 -> Add Graphite -> http settings url http://localhost
4.Import grafana dashboard: go to grafana.com/dashboards search for Jenkins: Performance and health overview-> copy id 
5.![import dashboard](https://i.imgur.com/jeagEZ9.png) Options grahite choose jenkins-test

### Install monit
```
yum install -y monit
vim /etc/monitrc
set mailserver smtp.gmail.com port 587
    username "EMAIL" password "PASSWORD"
    using tlsv12
vim /etc/monit.d/jenkins
check system $HOST
    if loadavg (5min) > 3 then alert
    if loadavg (15min) > 1 then alert
    if memory usage > 80% for 4 cycles then alert
    if swap usage > 20% for 4 cycles then alert
    if cpu usage (user) > 80% for 2 cycles then alert
    if cpu usage (system) > 20% for 2 cycles then alert
    if cpu usage (wait) > 80% for 2 cycles then alert
    if cpu usage > 200% for 4 cycles then alert

check process jenkins with pidfile /var/run/jenkins.pid
    start program = "/bin/systemctl start jenkins"
    stop program  = "/bin/systemctl stop jenkins"
    if failed host 192.168.33.200 port 8080 then restart

check process nginx with pidfile /var/run/nginx.pid
    start program = "/bin/systemctl start nginx"
    stop program  = "/bin/systemctl stop nginx"
    if failed host 192.168.33.200 port 80 then restart

check filesystem jenkins_mount with path /dev/sda2
    start program  = "/bin/mount /var/lib/jenkins"
    stop program  = "/bin/umount /var/lib/jenkins"
    if space usage > 80% for 3 times within 5 cycles then alert
    if space usage > 99% then stop
    if inode usage > 30000 then alert
    if inode usage > 99% then stop

check directory jenkins_home with path /var/lib/jenkins
    if failed permission 755 then exec "/bin/chmod 755 /var/lib/jenkins"
```
Enable gmail account Allow less secure apps to 'on'
```
systemctl start monit
tail -f /var/log/monit.log
# test by systemctl stop jenkins
```
![monit](https://i.imgur.com/gG5igvY.png)
## Implementing security and roles for Jenkins
### Jenkins Security best practices
- Disable job execution on the mater and use slave nodes for builds
- Use job restrictions plugin to confine specific jobs to specific nodes irrespective of the label used
install plugin job restrictions-> Manage Jenkins-> Manage Nodes
![restrict jobs execution at node](https://imgur.com/1gwJ4gM.png)
- Enable CSRF protection and update scripts to use crumbs
![configure csrf](https://i.imgur.com/9q0FIpP.png)
- Enable the slave to master access control
![config](https://i.imgur.com/lPoxSai.png)
- For large encironments, use role and matrix based authorization strategies to isolate project access
install plugin Role-based Authorization Strategy
![Project-based Matrix Authorization Strategy](https://i.imgur.com/H0bz6rU.png)
![Enable project-based security](https://i.imgur.com/YkFDXf5.png)
### use LDAP server and role-based strategy
1.ldap server config on centos 7
```
yum -y install openldap-servers openldap-clients
cp /usr/share/openldap-servers/DB_CONFIG.example /var/lib/ldap/DB_CONFIG
chown ldap. /var/lib/ldap/DB_CONFIG
systemctl start slapd

slappasswd -> Enter new password twice -> GENERATED_PASSWORD

#New file
config.ldif

#Add content
dn: olcDatabase={0}config,cn=config
changetype: modify
add: olcRootPW
olcRootPW: GENERATED_PASSWORD

#Run commands
ldapadd -Y EXTERNAL -H ldapi:/// -f config.ldif
ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/cosine.ldif
ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/nis.ldif
ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/inetorgperson.ldif

#New file
domain.ldif

#Add content
dn: olcDatabase={1}monitor,cn=config
changetype: modify
replace: olcAccess
olcAccess: {0}to * by dn.base="gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth"
  read by dn.base="cn=Manager,dc=practicaljenkins,dc=com" read by * none

dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcSuffix
olcSuffix: dc=practicaljenkins,dc=com

dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcRootDN
olcRootDN: cn=Manager,dc=practicaljenkins,dc=com

dn: olcDatabase={2}hdb,cn=config
changetype: modify
add: olcRootPW
olcRootPW: {SSHA}mviTD7um1R+LygfAN01MzQOtK4ezm1ob

dn: olcDatabase={2}hdb,cn=config
changetype: modify
add: olcAccess
olcAccess: {0}to attrs=userPassword,shadowLastChange by
  dn="cn=Manager,dc=practicaljenkins,dc=com" write by anonymous auth by self write by * none
olcAccess: {1}to dn.base="" by * read
olcAccess: {2}to * by dn="cn=Manager,dc=practicaljenkins,dc=com" write by * read


#Run command
ldapmodify -Y EXTERNAL -H ldapi:/// -f domain.ldif

systemctl restart slapd

#New file
base.ldif

#Add content
dn: dc=practicaljenkins,dc=com
objectClass: top
objectClass: dcObject
objectclass: organization
o: practicaljenkins
dc: practicaljenkins

dn: cn=Manager,dc=practicaljenkins,dc=com
objectClass: organizationalRole
cn: Manager
description: Directory Manager

dn: ou=users,dc=practicaljenkins,dc=com
objectClass: organizationalUnit
ou: users

dn: ou=groups,dc=practicaljenkins,dc=com
objectClass: organizationalUnit
ou: groups

#Run command - enter password of slappasswd command when asked

ldapadd -x -D cn=Manager,dc=practicaljenkins,dc=com -W -f base.ldif

## configure phpldapadmin
yum -y install httpd php php-mbstring php-pear
systemctl start httpd
yum -y install epel-release
yum clean all && yum makecache fast && yum update
vim /etc/phpldapadmin/config.php
<?php
$config->custom->session['blowfish'] = 'd7458abe84f9622a42ce3f9e45dfc457';  # Autogenerated for ip-172-31-4-228.ap-southeast-2.compute.internal

$config->custom->commands['cmd'] = array(
	'entry_internal_attributes_show' => true,
	'entry_refresh' => true,
	'oslinks' => true,
	'switch_template' => true
);

$config->custom->commands['script'] = array(
	'add_attr_form' => true,
	'add_oclass_form' => true,
	'add_value_form' => true,
	'collapse' => true,
	'compare' => true,
	'compare_form' => true,
	'copy' => true,
	'copy_form' => true,
	'create' => true,
	'create_confirm' => true,
	'delete' => true,
	'delete_attr' => true,
	'delete_form' => true,
	'draw_tree_node' => true,
	'expand' => true,
	'export' => true,
	'export_form' => true,
	'import' => true,
	'import_form' => true,
	'login' => true,
	'logout' => true,
	'login_form' => true,
	'mass_delete' => true,
	'mass_edit' => true,
	'mass_update' => true,
	'modify_member_form' => true,
	'monitor' => true,
	'purge_cache' => true,
	'query_engine' => true,
	'rename' => true,
	'rename_form' => true,
	'rdelete' => true,
	'refresh' => true,
	'schema' => true,
	'server_info' => true,
	'show_cache' => true,
	'template_engine' => true,
	'update_confirm' => true,
	'update' => true
);
$servers = new Datastore();
$servers->newServer('ldap_pla');
$servers->setValue('server','name','Stan Local LDAP Server');
$servers->setValue('server','host','172.31.4.228');
$servers->setValue('appearance','password_hash','');
$servers->setValue('login','attr','dn');
$servers->setValue('login','class',array('dc=practicaljenkins,dc=com'));
?>

systemctl restart httpd
vim /etc/httpd/conf.d/phpldapadmin.conf
#
#  Web-based tool for managing LDAP servers
#

Alias /phpldapadmin /usr/share/phpldapadmin/htdocs
Alias /ldapadmin /usr/share/phpldapadmin/htdocs

<Directory /usr/share/phpldapadmin/htdocs>
  <IfModule mod_authz_core.c>
    # Apache 2.4
    #Require local
    Require all granted
  </IfModule>
  <IfModule !mod_authz_core.c>
    # Apache 2.2
    Order Deny,Allow
    Deny from all
    Allow from 127.0.0.1
    Allow from ::1
  </IfModule>
</Directory>
cat /etc/selinux/config

# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
SELINUX=disabled
# SELINUXTYPE= can take one of three values:
#     targeted - Targeted processes are protected,
#     minimum - Modification of targeted policy. Only selected processes are protected.
#     mls - Multi Level Security protection.
#SELINUXTYPE=targeted

systemctl restart httpd
systemctl enable slapd
systemctl enable httpd
netstat -atnulp
```
Login to ipaddress/phpldapadmin
Ligin DN: cn=Manager,dc=practialjenkins.dc=com
Password: 

Under ou=groups create a child entry-> Generic: Posix Group  
3 groups: admins, developers, devops

Under ou=users create Generic: User Account a b c

Under cn=admins Add new attribue mumberUid a
same for other groups

Configure Global Security
![ldap config](https://i.imgur.com/JM1wDEf.png)
Group membership filter -> (| (member={0}) (uniqueMember={0}) (memberUid={1}))

Under Authorization choose Role-Based Strategy

Under Manage Jenkins click Mange and Assign Roles -> Manage Role-> Role to add "admins, developers and devops"
![managerole](https://i.imgur.com/l1wUQY3.png)

Assign Roles-> User/group to add-> admins, developers, and devops
![assignrole](https://i.imgur.com/10Cbmz6.png)

User different user for login:
![login](https://i.imgur.com/R8fVL74.png)

## Using the jenkinss api and automating plugin management
get API token from 'People'> admin-> configure->API Token

Automatically tirgger the build by:
```
curl -u admin:20fda2a6ed2ff7a8cf9db9dd50a0c2b0 "http://localhost:8080/api/json?pretty=true"
curl -s 'http://localhost:8080/crumbIssuer/api/xml?xpath=concat(//crumbRequestField,":",//crumb)' -u admin:admin
curl -X POST -u admin:20fda2a6ed2ff7a8cf9db9dd50a0c2b0 "http://localhost:8080/job/python-job/build" -H "Jenkins-Crumb:f40b9f1084de9e60cca87085b30e6159"```
Jenkins CLI:
```
wget --auth-no-challenge "http://localhost:8080/jnlpJars/jenkins-cli.jar"
java -jar jenkins-cli.jar -auth admin:20fda2a6ed2ff7a8cf9db9dd50a0c2b0 -s http://localhost:8080 help
java -jar jenkins-cli.jar -auth admin:20fda2a6ed2ff7a8cf9db9dd50a0c2b0 -s http://localhost:8080 install-plugin aws-codebuild -deploy
java -jar jenkins-cli.jar -auth admin:20fda2a6ed2ff7a8cf9db9dd50a0c2b0 -s http://localhost:8080 install-plugin http://updates.jenkins-ci.org/latest/ansicolor.hpi -deploy
java -jar jenkins-cli.jar -auth admin:20fda2a6ed2ff7a8cf9db9dd50a0c2b0 -s http://localhost:8080 install-plugin http://updates.jenkins-ci.org/download/plugins/beaker-builder/1.6/beaker-builder.hpi -deploy
```
# Integrating Jenkins with external services
## Integrating with Github
![workflow](https://i.imgur.com/hkwPMqG.png)

- Preparing Github and configuring Jenkins to work together
Github->Setting->Developer setting->Personal access tokens-> Genere a new token-> select scopes (repo:all |admin:repo_hook all| admin:org_hook)->copy token

Jenkins->Add credentials-> Kind username with password |scope global| username szhouchoice| password access token|id jenkins-secret
add another credentials-> kind secret test-> secret token|id jenkins-token->save

Username with password will use for multi-branch pipeline
secret text will be use in Global configuration for github server
![github server setting](https://i.imgur.com/u4OVbH2.png)

Install GitHub Branch Source Plugin

- Configuring the Pipeline to create a code build workflow
[project](https://github.com/practicaljenkins/sample-php-project)
```
git add -A
git commit -m "adding code"
git push origin master
git checkout -b develop
git push orgin develop
```
In Jenkins-> Create new Multibranch Pipeline->
![Branch sources setting](https://i.imgur.com/Tn2maPQ.png)

- Testing the pipelines with different scenarios
```
git checkout -b feature-001
vim src/ConnectTest.php
```
push feature branch to github
![merge in github](https://i.imgur.com/JOkzC9O.png)
will trigger the build in Jenkins
## Integrating with Sonarqube
- setting up Sonarqube prerequisites for Jenkins
- Install and configure Sonarqube plugin
- Configure Jenkins pipeline for Sonarqube action
- Generate Sonarqube analysis report from Jenkins pipeline
