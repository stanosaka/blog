---
title: saltstack
tags: saltstack
date: 2016-09-15 11:28:58
---

## bug 01: ##
    [root@toad salt]# salt '*' state.sls upload_data
    salt-minion01:
    Data failed to compile:
    ----------
    Pillar failed to render with the following messages:
    ----------
    Error encountered while render pillar top file.
    macadamina:
    Data failed to compile:
    ----------
    Pillar failed to render with the following messages:
    ----------
    Error encountered while render pillar top file.
    
## Fix 01: ##
    [root@toad salt]# salt "*" saltutil.refresh_pillar
    macadamina:
    True
    salt-minion01:
    True

----------


## copy the file to spe. location: ##
    [root@toad salt]# tree
    .
    ├── contab.sls
    ├── createdir.sls
    ├── createuser.sls
    ├── del_cron.sls
    ├── files
    │   ├── hosts
    │   ├── wgzy.sql
    │   └── zfgtai.zip
    ├── host_file.sls
    ├── nginx_install.sls
    ├── top.sls
    ├── upload_data.sls
    └── upload_mysql.sls
    
    [root@toad salt]# cat upload_mysql.sls
    /usr/share/tomcat/webapps/wgzy.sql:
      file.managed:
    - source: salt://files/wgzy.sql
    - user: root
    - group: root
    - mode: 644
    
    [root@toad salt]# salt '*' state.sls upload_mysql
    salt-minion01:
    ----------
      ID: /usr/share/tomcat/webapps/wgzy.sql
    Function: file.managed
      Result: True
     Comment: File /usr/share/tomcat/webapps/wgzy.sql updated
     Started: 17:11:02.739156
    Duration: 298.114 ms
     Changes:
      ----------
      diff:
      New file
      mode:
      0644
    
    Summary
    ------------
    Succeeded: 1 (changed=1)
    Failed:0
    ------------
    Total states run: 1
    macadamina:
    ----------
      ID: /usr/share/tomcat/webapps/wgzy.sql
    Function: file.managed
      Result: True
     Comment: File /usr/share/tomcat/webapps/wgzy.sql updated
     Started: 17:11:05.274496
    Duration: 254.31 ms
     Changes:
      ----------
      diff:
      New file
      mode:
      0644
    
    Summary
    ------------
    Succeeded: 1 (changed=1)
    Failed:0
    ------------
    Total states run: 1
    
    

Modify /etc/tomcat/server.xml
 <Host name="localhost"  appBase="webapps"
            unpackWARs="true" autoDeploy="true">
            
to
 <Host name="localhost"  appBase="/deploy/source/aofa_pro"
            unpackWARs="true" autoDeploy="true">

----------

## saltstack run command: ##
    [root@toad salt]# cat modscript.sls
    sed -i '125s#appBase=".*"# appBase="/deploy/source/aofa_pro"#' /etc/tomcat/server.xml:
      cmd.run
    
    
    [root@toad salt]# salt '*' state.sls modscript
    macadamina:
    ----------
      ID: sed -i '125s#appBase=".*"# appBase="/deploy/source/aofa_pro"#' /etc/tomcat/server.xml
    Function: cmd.run
      Result: True
     Comment: Command "sed -i '125s#appBase=".*"# appBase="/deploy/source/aofa_pro"#' /etc/tomcat/server.xml" run
     Started: 11:06:15.245454
    Duration: 38.108 ms
     Changes:
      ----------
      pid:
      38516
      retcode:
      0
      stderr:
      stdout:
    
    Summary
    ------------
    Succeeded: 1 (changed=1)
    Failed:0
    ------------
    Total states run: 1
    salt-minion01:
    ----------
      ID: sed -i '125s#appBase=".*"# appBase="/deploy/source/aofa_pro"#' /etc/tomcat/server.xml
    Function: cmd.run
      Result: True
     Comment: Command "sed -i '125s#appBase=".*"# appBase="/deploy/source/aofa_pro"#' /etc/tomcat/server.xml" run
     Started: 11:06:16.505044
    Duration: 28.841 ms
     Changes:
      ----------
      pid:
      37702
      retcode:
      0
      stderr:
      stdout:
    
    Summary
    ------------
    Succeeded: 1 (changed=1)
    Failed:0
    ------------
    Total states run: 1

check result:
    [root@toad salt]# salt '*' cmd.run 'sed -n 125p /etc/tomcat/server.xml'
    macadamina:
      <Host name="localhost"   appBase="/deploy/source/aofa_pro"
    salt-minion01:
      <Host name="localhost"   appBase="/deploy/source/aofa_pro"

----------
copy the file form master to minion:
    [root@toad salt]# salt 'macadamina' cp.get_file salt://files/zfgtai.zip /deploy/source/aofa_pro/zfgtai.zip
    macadamina:
    /deploy/source/aofa_pro/zfgtai.zip
    
    
Append text into file:

    [root@salt-minion01 ~]# sed -i '34 a<Valve className="com.orangefunction.tomcat.redissessions.RedisSessionHandlerValve" /> <Manager className="com.orangefunction.tomcat.redissessions.RedisSessionManager" host="localhost" port="6379" database="0" maxInactiveInterval="60" />' context.xml
