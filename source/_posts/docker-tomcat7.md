---
title: docker tomcat7
tags: 'docker,tomcat'
date: 2016-08-24 11:15:59
---

# docker container with centos 6 and tomcat 7 #

    # docker search centos
    # docker pull centos
    
Download the Oracle JDK

    wget --no-cookies --no-check-certificate --header "Cookie: gpw_e24=http%3A%2F%2Fwww.oracle.com%2F; oraclelicense=accept-securebackup-cookie" "http://download.oracle.com/otn-pub/java/jdk/7u72-b14/jdk-7u72-linux-x64.tar.gz"
    
Create a working folder and place both Tomcat tarball and Oracle JDK files in the same folder.

    [root@docker centos6]# ls
    apache-tomcat-7.0.70.tar.gz  Dockerfile  jdk-7u72-linux-x64.tar.gz

    [root@docker centos6]# cat Dockerfile
    FROM centos:latest
    MAINTAINER lreeder
    
    #Helpful utils, but only sudo is required
    #RUN yum -y install tar
    #RUN yum -y install vim
    #RUN yum -y install nc
    RUN yum -y install sudo
    
    ######## JDK7
    
    #Note that ADD uncompresses this tarball automatically
    ADD jdk-7u72-linux-x64.tar.gz /opt
    WORKDIR /opt/jdk1.7.0_72
    RUN alternatives --install /usr/bin/java java /opt/jdk1.7.0_72/bin/java 1
    RUN alternatives --install /usr/bin/jar jar /opt/jdk1.7.0_72/bin/jar 1
    RUN alternatives --install /usr/bin/javac javac /opt/jdk1.7.0_72/bin/javac 1
    RUN echo "JAVA_HOME=/opt/jdk1.7.0_72" >> /etc/environment
    
    ######## TOMCAT
    
    #Note that ADD uncompresses this tarball automatically
    ADD apache-tomcat-7.0.70.tar.gz /usr/share
    WORKDIR /usr/share/
    RUN mv  apache-tomcat-7.0.70 tomcat7
    RUN echo "JAVA_HOME=/opt/jdk1.7.0_72/" >> /etc/default/tomcat7
    RUN groupadd tomcat
    RUN useradd -s /bin/bash -g tomcat tomcat
    RUN chown -Rf tomcat.tomcat /usr/share/tomcat7
    EXPOSE 8080

Build image

    # docker build --rm=true -t centos6/tomcat7 .

Setup port forwarding and start tomcat

    [root@docker centos6]# docker run -p 80:8080 --rm=true -t -i --name tomcat7 centos6/tomcat7 /usr/bin/sudo -u tomcat /usr/share/tomcat7/bin/catalina.sh run

Find out the IP address for the running container:

    [root@docker centos6]# docker inspect --format '{{ .NetworkSettings.IPAddress }}' tomcat7
    172.17.0.22

