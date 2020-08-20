---
title: devops101
tags: devops
---
# Indtroduction
## What is Devops?

Devops is a new Philosophy that can help organizations innovate faster and be more productive and reponsive to busesiness needs. It promotes collaboration between devlopers and operations by automating infrastructure, automation workflows and continuously measuring application performance.

## Devops goals and benefits
- Increase rate of software devliery
- Imporoves company's time to market potentially from months and weeks to days and hours.
- Provide huge competitive advantage due to shorter release cycles.
- Help them automatin the releases and their infrasturctre so that they can foucus on increasing business value
- Reduce manual processes leading to happier emplyees and customers.
- Lower failure rate of releases.
- Faster mean time to recovery.
- Shortened lead time between fixes.

## Devops adoption interest
- Use of Agile development processes
- Demand for frequent realeases to production
- Availability of cloud infrastructre and tools
- Increased usage of data center automation and configuration management tools.
- Incrased focus on test automation and continous integration methods.
- A critical mass of publicly available best practices in the indestry.

## Devops toolchain
Devops is a set ot tool chain that will help organizations achieve their goals. The tools fit into several catogories that are:
- Code - SCM, workflow, development, review and merge.
- Build - Continous integration and builds
- Test - Continous testing that provided feeback
- Package - Packaging code and release to artifact repository
- Release - Release, promotion and change management
- Configure - Automated infrastructure and configuration management
- Monitor - Application performance and log monitoring.

## Acronyms:
- SDLC - Software Development Lifecycle
- SCM - Software Configuration Management
- ELK - Elasticsearch, Logstach and Kibana.
- GitFlow - Git Workflow.
- SSH - Secure Shell
- GIT - Version COntrol System.
- CI - Continous Integration
- Repository - Place where artificats are stored.

## Section 1
1. Setup Artifactory:
  a. Startup Artifactory
  b. Setup Admin account
  c. Port: 8081
2. Setup Tomcat:
  a. Startup tomcat
  b. Update tomcat-users.xml
```
  <role rolename="tomcat"/>
  <role rolename="manager-gui"/>
  <role rolename="manager-script"/>
  <user username="tomcat" password="tomcat" roles="tomcat"/>
  <user username="manager" password="tomcat" roles="tomcat,manager-gui"/>
  <user username="jenkins" password="jenkins" roles="manager-script"/>
```
  c. Port: 8080
3. Setup Jenkins:
  a. Setup Jenkins
  b. Port: 8090
```
for windows:
Go to the Program Files/Jenkins directory where you installed Jenkins
Open the Jenkins.xml in the editor
Find "--httpPort=8080" and replace the port 8080 with the new port number
to restart:
jenkins.exe restart
```
4. Setup Maven:
  a. Startup maven home.
  b. Generate maven security.xml and settings-security.xml.
```
mvn -emp password
copy string to settings-security.xml
cop two xml to ~/.m2 folder
mvn -ep password
mvn clean
mvn help:describe
```
## Section 4
1. Setup SSH for GitHub
2. Create a Github repository
3. Create a Spring BOot project
4. Configure for SCM and Artifactory
5. Commit changes to the feature branch and push to Github
6. Create a pull reuquest to develop branch and merge.
## Section 5
1. Setup SSH and Maven settings for Jenkins User.
2. Install and configure Jenkins Plugins.
  a. Conditinal Buildstep plugin
  b. Deploy to container plugin
  c. Environment injector plugin
  d. Git parameter plugin
  e. Github branch Source Plugin
3. Build SNAPSHOT version using Jenkins.
4. Configure Jenkins to Deploy to Tomcat.
5. Configure for SCM and Artifactory
6. Commit changes to the feature brach and push to Github
7. Create a pull request to develop branch and merge.
