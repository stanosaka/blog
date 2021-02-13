---
title: Jenkins For Professionals
tags: [Jenkins, Devops, CD/CI]
date: 2019-05-08 21:26:40
---

# Jenkins for Professionals
## Jenkins's architecture

Github repository --> Master -> slave
                             -> slave

Why distributed architecture for Jenkins?
- Sometime, several different environments needed
- If a larger project gets build, a single server cannot handle entire load
Jenkins Master:
- Scheduling jobs
- Communicating with the slaves
- Monitor the slaves
- Present the results
- Master can also execute build jobs

Jenkins Slave:
- Communicates with Jenkins Master
- Can run in different OS
- Execute jobs
- Flexibility

## Jenkins Freestyle job
- Central feature of Jenkins
- Using jenkins UI to create a CI Pipeline

https://github.com/ravikiran-srini/springExample

## Scheduling a Jenkins job
- Scheduling is one of the options of a Build Trigger
- Use CRON expressions to schedule a job
   - Each line consists of 5 fields sparated by TAB or whitespace
   - MINUTE HOUR DOM MONTH DOW
   0 refers to Sunday
- To specify multiple values, use the following:
   - * for all valid values
   - 0-9 specify a range of values
   - 2,3,4,5 to enumerate multiple values
- Use Hash system for automatic balancing
   - Use H H * * * instead of 0 0 * * *
- Use aliases like @yearly, @monthly, @weekly
   - @hourly is the same as H * * * *

Under Source Code Management of Project,
in Build Triggers, tick Build periodically, put five starts.

## Triggering builds remotely
Build Triggers-> tick 'Tigger build remotely', put Authenication Token value.

## Parameterizing build
- Allow you to prompt users for one or more inputs
- Each parameter has a name and a value
- Can be accessed using $paramater or %parameter%
- 'Build Now' will be replaced by "Build with Parameters"

Types of Parameters
- Boolean Parameter
- Choice Parameter
- Credentials Parameter
- File Parameter
- List Subversion tags
- Password Parameter
- Run Parameter
- String Parameter

## Creating a user
## Installing Plugins
Plugin name: Role-based Authorization Strategy

## Implementing Role based access
Enabling role based access
Manage Jenkins-> Configure Global Security, tick 'Enable security'
Authorization select 'Role-Based Strategy'

Under Manage Jenkins you will have a new 'Manage and Assign Roles'-> Manage roles

In 'Global roles' add role 'team' with overall read permission.
In "Project roles" add role 'dev' with 'Pattern' 'Dev.*', and tick all the boxes.
Same for 'test' and 'ops'.
![ManageRoles](https://i.imgur.com/MSJOKSL.png)
### Assign Roles 
Click "Assign Roles", in 'Global roles' add 'dev', 'test' and 'ops' into group 'team'.
Under 'Item roles' add user 'dev','test' and 'ops' and assign to its own role. 
![AssignRoles](https://i.imgur.com/ml63iH6.png)
---
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

## Introduction to Jenkins Pipeline
- A suite of plugins that support continuous delivery pipelines
- They provide tools for modeling delivery pipelines as
- Definition is written into a text file called 'Jenkinsfile'

Benefits of pipelines
- Automatically creates a Pipeline build process
- Code review of the Pipeline
- Audit trail of the Pipeline 
- Single source of truth

Declarative vs Scripted Pipeline 
- Jenkinsfile can be written using:
   - Declarative
   - Scripted
- Declarative is a more recent feature
   - provides rich syntactical feature
   - designed to make reading & writing Pipelines easier

Why pipeline?
- Pipelines add a powerful set of automation tolls onto Jenkins
- Features of Pipeline are:
   - Pipelines are implemented in code 
   - Pipelines support complex real-world CD requirements
   - Pipelines can survive both planned and unplanned restarts
   - Pipeline plugin supports custom extensions
   - Pipelines can stop and wait

## Pipeline concepts
- Pipeline
- Node
- Stage: build, test, deploy
- Step

Declarative Pipeline syntax
```
pipeline{
  agent any
  stages{
    stage('Build'){
	steps {
	   //
        }
    }
    stage('Test'){
	steps {
	   //
        }
    }
    stage('Deploy'){
	steps {
	   //
        }
    }
  }
}
```
Declarative Pipeline syntax
```
  node {
    stage('Build'){
      //
    }
    stage('Test'){
      //
    }
    stage('Deploy'){
     //
    }
  }
```

## Creating a simple Pipeline
- Pipeline can be created in any of the following ways:
   - Blue Ocean
   - Through Classic UI
   - In SCM(Source Code Managment)

## Building a project with jenkins pipeline
pipeline script:
```
pipeline{
  agent any
  stages{
    stage('Build'){
        steps {
           bat "rm -rf springExample"
           bat "git clone https://github.com/ravikiran-srini/springExample.git"
           bat "mvn clean -f springExample"
        }
    }
    stage('Test'){
        steps {
           bat "mvn test -f springExample"
        }
    }
    stage('Deploy'){
        steps {
           bat "mvn package -f springExample"
        }
    }
  }
}
```
## Building a Pipeline with Jenkinsfile
Jenkinsfile
- Complex Pipelines are difficult to write and maintain
- You can write Jenkinsfile in an IDE and commit to source control
1. In https://github.com/szhouchoice/springExample add a Jenksinsfile
```
pipeline {
  agent any
  stages {
    stage('Build') {
      steps {
        sh 'mvn clean'
      }
    }
    stage('Test') {
      steps {
        sh 'mvn test'
      }
    }
    stage('Deploy') {
      steps {
        sh 'mvn package'
      }
    }
  }
}
```
2. Pipeline
![Pipeline script from SCM](https://i.imgur.com/EsJCtWv.png)

## Using environment variables
**Environment variables**
- Jenkins exposes environment variables through the variable 'env'
- Entire list of environment variables is accessible from [env variables](localhost:8080/pipeline-syntax/globals#env)

example:
```
pipeline {
    agent any
    stages {
        stage('stage1') {
            steps {
                echo "Build ID: ${env.BUILD_ID}, Jenkins URL: ${env.JENKINS_URL}"
            }
        }
    }
}
```

Console Ouput:
```
Started by user Stan Zhou
Running in Durability level: MAX_SURVIVABILITY
[Pipeline] Start of Pipeline
[Pipeline] node
Running on Jenkins in /Users/szhou/.jenkins/workspace/stanpipeline
[Pipeline] {
[Pipeline] stage
[Pipeline] { (stage1)
[Pipeline] echo
Build ID: 1, Jenkins URL: http://localhost:8080/
[Pipeline] }
[Pipeline] // stage
[Pipeline] }
[Pipeline] // node
[Pipeline] End of Pipeline
Finished: SUCCESS
```
## Setting environment variables
- Declarative Pipeline
   - use 'environment' directive
- Scripted Pipeline
   - use 'withEnv' step
- environment directive in top-level pipeline block will apply to all steps
- environment directive within a stage will only apply with the stage

example:
```
pipeline {
    agent any
    environment {
        mainenv = 'test'
    }
    stages {
        stage('stage1') {
            environment {
                subenv = 'test1'
            }
            steps {
                echo mainenv
                echo subenv
            }
        }
        stage('stage2') {
            steps {
                echo mainenv
                echo subenv  # will pro error
            }
        }
    }
}
```

# Jenkins Blue Ocean

## Getting started with blue ocean

What is blue ocean?
- Blue Ocean refefines the user experience of Jenkins
- Features of Blue Ocean are:
   - Visualization
   - Pipeline editor
   - Personalization
   - Precision
   - Native integration for branch

