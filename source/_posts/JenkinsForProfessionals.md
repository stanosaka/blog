---
title: Jenkins For Professionals
tags: 'Jenkins, Devops, CD/CI'
date: 2019-05-08 21:26:40
---

# Jenkins for Professionals
## Jenkins in Docker
### Running jenkins in docker
```
docker pull jenkins
docker run -p 8080:8080 -p 50000:50000 jenkins
```
### Persisting Jenkins data in a Volume
```
docker volume create volume1
docker volume volume ls
docker run -p 8080:8080 -p 50000:50000 -v  volume1:/var/jenkins_home jenkins #v means bond a volume
```
`docker stop $(docker ps -aq) #stop all the containers`
`docker rm $(docker ps -aq) #remove the container`
### Running multiple Jenkins instances in Docker
![demo](https://i.imgur.com/0yAeBkU.png)
```
docker run -p 8080:8080 -p 50000:50000 -v  volume1:/var/jenkins_home jenkins
docker run -p 8081:8080 -p 50001:50000 -v  volume1:/var/jenkins_home jenkins
```
## Jenkins Plugins
### Create build monitor view
Install pugin: build monitor view
### Using [catlight](https://catlight.io)

**Catlight**
- A notification app for developers
- Available for wind, mac & Linux os
- Monitor Busg, tasks, builds
- See status in the tray and get notified
- This is a jenkins plugin

### Using Jenkins CLI
- Jenkins has a built-in CLI
Manage Jenkins-> Jenkins CLI-> Download jenkins-cli.jar

`java -jar jenkins-cli.jar -s http://localhost:8080/ build pipeline -f --username stanadmin --password mario54321`

Change 'Configure Global Security' -> 'Authorization'-> Logged-in users can do anything-Allow anonymous read access

`java -jar jenkins-cli.jar -s http://localhost:8080/ build Parameterized-build -f -p environment='dev'`

### Jenkins Multibranch Pipeline
- Implement different Jenkinsfile for different branches
New iteam-> Select Multibranch Pipline-> Scan multibranch pipleline Now
```
git remote add proj1 https://github.com/szhouchoice/springExample.git
git branch
git branch -a
git branch feature2
git checkout feature2 # Switched to branch 'feature2'
git push origin feature2
```
### Integrating Jenkins with Slack
Install: Jenkins->Plugin Manager-> Slack Notification
### Jenkins Metrics and Trends
**Build History Metrics Plugin**
- Mean Time To Failure (MTTF)
- Mean Time To Recovery (MTTR)
- Standard Deviation
Install: Plugin Manager->Build history metrics
![Build History Metrics](https://i.imgur.com/C9aaE74.png)
**Global Build Stats**
Install: Plugin Manager->global-build-stats
Manage Jenkins->Global Build Stats->Initalize stats->Create new chart->
Title: foobar
600*400
Hourly
24 hours
![Global Build Stats](https://i.imgur.com/tRfhyNR.png)
