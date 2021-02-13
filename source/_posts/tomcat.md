---
title: tomcat troubleshooting
tags: tomcat
date: 2016-08-12 15:19:32
---

### Issue 1: ###
   ![](http://obsavus1p.bkt.clouddn.com/outmemoryissue.PNG)
### Solve: ###
vim /tomcat home dir/bin/catalina.sh
add the following line:

   ` JAVA_OPTS="-Djava.awt.headless=true -Dfile.encoding=UTF-8 -server -Xms1024m -Xmx1024m -XX:NewSize=512m -XX:MaxNewSize=512m -XX:PermSize=512m -XX:MaxPermSize=512m"`
  

----------
  
### Issue 2: ###
    `SEVERE: The web application [/name] appears to have started a thread named [Prototyper] but has failed to stop it. This is very likely to create a memory leak.`
    
### Solve: ###
tomcat did not shutdown properly

`vim /tomcat home dir/bin/catalina.sh`
add the following line after PRGDIR=`dirname "$PRG"`:
```
if [ -z "$CATALINA_PID" ]; then
      CATALINA_PID=$PRGDIR/CATALINA_PID
      cat $CATALINA_PID
fi
```
==============================bin/shutdown.sh   
`exec "$PRGDIR"/"$EXECUTABLE" stop  -force "$@"  #add -force `
