---
title: Jenkins Pipeline
date: 2019-07-29 11:00:54
tags:
  - Jenkins
  - Devops
  - CD/CI
  - Pipeline
  - Blue Ocean
---
# An Example of Declarative Pipeline:
```bash
pipeline {
    agent {
       node {
           label 'master'
       }
    }
    stages {
       stage('Build') {
           steps {
               sh 'mvn clean verify -DskipITs=true'
           }
       }
       stage ('Static Code Analysis') {
           steps {
               sh 'mvn clean verify sonar:sonar'
           }
       }
       stage ('Publish to Artifactory') {
           steps {
               script {
                   def server = Artifactory.server 'Default Artifactory'
                   def uploadSpec = """{
                       "files": [
                           {
                               "pattern": "target/*.war",
                               "target": "helloworld/${BUILD_NUMBER}/",
                               "pattern": "Unit-Tested=Yes",
                           }
                        ]
                    }"""
                    server.upload(uploadSpec)
		}
            }
        }
    }
}
```
**Jenkinsfile**
A Jenkinsfile is a file that contains Pipeline code.

## Declarative Pipeline Syntax
**Sections**

Sections in Declarative Pipeline Syntax can contain one or more Directive or Steps.
### Agent
- The *agent* section defines where the whole Pipleline, or a specific stage, runs. It is positioned right at the beginnning of the *pipeline { }* block the is compolsory.
- can also use it inside the *stage* directive, but that's optional.
The *agent* section allow specific parameters to suit various use-cases.

**any**

Use *agent any* inside your *pipleine {}* block, or *stage* directive, to run them on any agent available in Jenkins.
![A pipleline{}Block with agent any](https://i.imgur.com/6ByXXdT.png)

**none**

Use *agent node* inside your *pipeline {}* block to stop any global agent from running your pipeline. In such case, each *stage* directive inside your pipeline must have its *agent* section.
![A Pipeline{}Block with Agnet None](https://i.imgur.com/MQIQ9ID.png)

**label**

Run your complete pipeline, or stage, on an agent available in Jenkins with specific label.
![A pipeline{} block with agent { label '....'} section](https://i.imgur.com/fw2xBh7.png)

**node**

Run your complete pipeline, or stage, on an agent available in Jenkins with s specific label. The behavior is similar to the *label* parameter. Howerver, the *node* parameter allows you to specify additional options such as *customWorkspace*. Workspace, or Jenkins Pipeline workspace, is a reserved directory on your Jenkins agent, where all your source code is downloaded and where your builds are performed.

![agent node customWorkspace section](https://i.imgur.com/jtj39sB.png)

**docker**

Run your complete pipeline, or stage, inside a docker container. The container will be dynamically spawned on a pre-configured docker host.

The *docker* option also takes an *args* parameter that allows you to pass arguments directly to the *docker run* command.

![An agent section with docker parameter](https://i.imgur.com/oINFaFW.png)

**dockerfile**

Run your pipeline, or stage, inside a conatiner build from a *dockerfile*. To use this option, the *jenkinsfile* and the *dockerfile* should be present inside the root of your source code.

![agent section with dockerfile](https://i.imgur.com/M2e1Xq0.png)

**post**

The *post* section allows you to run additional *setps* after running a pipeline or a stage.
![a post section at the end of a pipeline](https://i.imgur.com/xctaUYY.png)

![add specific conditions to post section](https://i.imgur.com/wmfuirh.png)

**stages**

The *stages* section contains a sequence of one or more *stage* directives. It's used to segregate a sinle or a collection of *stage* directives form the rest of the code.

The *stages* section has no parameters and gets used at least once in the pipeline.

**steps**

The *steps* section contains one or more steps that should run inside a *stage* directive. The *steps* section is used to segrate a single or a collection of steps from the rest of the code inside a *stage* directive.
![a pipeline{} block with steps section](https://i.imgur.com/FxgpLwC.png)

### Directives
Directives are supporting actors that give direction, set conditions, and provide assistance to the steps of your pipeline in order to archieve the required purpose. In the following section we will see all the *Directives* that are available to you with the Declarative Pipeline Syntax.

**environment**
The *environment* directive is used to specify a set of key value pairs that are available as environment variables for all the steps of your pipeline, or only for the steps of a specific stage, depending on where you place the *environment* directive.

![pipeline block with environment section](https://i.imgur.com/g07NWDy.png)

**options**
The *options* directive allows you to define pipeline-specific *options* from within the pipeline.

1. buildDiscarder
Allows you to keep a certain number of recent builds run.
```
// Will keep the last 10 Pipeline logs and artifacts in Jenkins.
options {
   biuldDiscarder(logRotator(numToKeepStr: '10'))
}
```
2. disableConcurrentBuilds
Disallows concurrent executions of the Pipeline.
To understand the usage of this option, imagine a situation. You have a Pipeline in Jenkins Blue Ocean that’s triggered on every push on your source code repository. Moreover, you have a single build agent to run your Pipeline.

Now, if there are multiple pushes to the repository in a short duration of time (say three different pushes at once), then there is a pipeline for every push (say, Pipelines #1, #2, and #3). However, since we have a single build agent, Pipelines #2 and #3 wait in a queue with Pipeline #1 running. Pipelines #2 and #3 run whenever the build agent is free again.

To disable this functionality, and discard everything that’s in the queue, you should use the disableConcurrentBuilds option. The following option is useful to perform incremental builds.
```
options {    
   disableConcurrentBuilds ()
}
```

3. newContainerPerStage

The *newContainerPerStage* option is used inside an agent section that employs the docker or dockerfile parameters.

When specified, each stage runs in a new container on the same docker host, as opposed to all stages utilizing the same container.

This option only works with an *agent* section defined at the top-level of your *Pipeline {}* block.

4. preserveStashes
In Jenkins, you can pass artifacts between stages. You do it using stash. The *preserveStashes* option allows you to preserve stashes of the completed Pipelines. It’s useful if you have to re-run a stage from a completed Pipeline.
```
// Preserver stashes from the most recent completed Pipeline.
options {
   perserveStashes()
}
```
or
```
// Preserve stashes from the ten most recent completed Pipelines.
options {
   perserveStashes(10)
}
```

5. retry
The retry option allows you to retry a Pipeline, or a stage, on failure.
![retry](https://i.imgur.com/ifB8mza.png)

6. skipDefaultCheckout
The *skipDefaultCheckout* option is another handy choice. In Jenkins Declarative Pipeline, the source code by default gets checked out in every *stage* directive of a Pipeline.

Use this option to skip checking out source code in a given *stage* directive.

![block with skipDefaultCheckout options](https://i.imgur.com/hLjjGBF.png)

7. timeout
The *timeout* option allows you to set a timeout period for your Pipeline run. If the pipeline run teimes out the duration defined using a *timeout*, Jenkins aborts the Pipeline.

It is possible to use the timeout option at the Pipeline or stage level.

![timeout option](https://i.imgur.com/Kv5XzMK.png)

There are times when a pipeline that’s waiting for a process runs forever, generating huge logs and bringing down Jenkins server. The timeout option comes handy in such cases.

8. timestamps
The *timestamps* option prepend all lines of the console log with a timestamp. The following option is useful in debugging issues where you would need the time of execution of a specific command as evidence.

It is possible to use the *timestamps* option at the Pipeline or stage level.

![timestamps](https://i.imgur.com/QBq3HBf.png)

### Parameters
The *parameters* directive allows a user to provide a list of specific parameters when triggering a Pipeline.

Following are the types of available parameters that I find useful.

1. string

Use the *string* parameter to pass a string.
![string](https://i.imgur.com/QIBJjWM.png)

2. text

Use the *text* parameter to pass a multiline text.
![text](https://i.imgur.com/vSrmP5T.png)

3. booleanParam

Use the *booleanParam* parameter to pass true for false.
![booleanParam](https://i.imgur.com/AVmdPvS.png)

4. choice

Use the *choice* parameter to allow you to choose from a set of values, and them pass it to the pipeline
![choice](https://i.imgur.com/bzZ8FsZ.png)

5. file

A *file* parameter allows you to specify a file to upload that your pipeline might require.
![file](https://i.imgur.com/zXmC4BJ.png)

### Triggers
The *triggers* directive defines the various ways to rigger a pipeline.

Pipelines created using Jenkins Blue Ocean may not require triggers, since such Pipelines are triggered using webhooks configured on the source code repository (Git/GitHub/GitLab).

Following are the two triggers directives that might be useful, in some scenarios, for your Continuous Delivery pipeline.
1. cron
The *cron* trigger accepts a cron-style string to define a regular interval at which the piepleine shoulde be re-triggered, for example:
```
pipeline {
    agent any
    triggers {
       cron('H */4 * * 1-5')
    }

    stages {
       stage('Long running test') {
          steps {
             // Do something
          }
       }
    }
}
```
2. upstream
The *upstream* trigger allows you to define a comma-sparated list of jobs and a threshold. When any of the jobs from the list finishes with the defined threshold, your pipeline gets triggererd. This feature is useful if your CI and CD Pipelines are two separate Jenkins Pipelines.
![upstream](https://i.imgur.com/TOK46DZ.png)

### Stage
A pipeline stage contains one or more *steps* to achieve a task, for example: Build, Unit-Test, Staic Code Analysis, etc. The *stage* directive contains one or more *steps* sections, an *agent* section (optional), and other stage-related directives.
In the previous sections, you have already seen many examples demonstrating the stage directive. It is possible to run multiple stage directives in sequence, in parallel, or a combination of both. You’ll see it shortly.

### tools
A *tools** directive allows you to install a tool on your agent automatically. It can be defined inside a *pipeline {}* or a *stage {}* block.
![](https://i.imgur.com/NtZHRhQ.png)

For the tools directive to work, the respective tool must be pre-configured in Jenkins under **Manage Jenkins**-> **Global Tool Configuraton**. However, the tools directive gets ingnored if you put an *agent none* section in the *pipeline {}* block.

As of today, only the following tools work in Declarative Pipeline:
- maven
- jdk
- gradle

### input
The *input* directive allows the Pipleine to prompt for input. The *input* directive works with the *stage* directive. When used, the *stage* directive pauses and wait for your input, and when you provide the input, the stage then continues to run.
![input options table](https://i.imgur.com/qrWOh2L.png)

![input code](https://i.imgur.com/AQuF2fa.png)

### when
The *when* directive allows your Pipeline to decide whether a *stage* directive should be run based on some conditions.

The *when* directive must contain at least one condition. If the *when* directive contains more than one condition, all the chould conditions must return true for the stage to execute.

1. branch
Run a *stage* directive only if the branch that’s being built matches the branch pattern provided.
![branch condition](https://i.imgur.com/ofJR2Gl.png)

The *Performance Testing* stage in this Pipeline runs for all branches that start with a *release*-string. Example: *release-beta, release-1.0.0.1*.

2. buildingTag
The *buildingTag* option allows you to run a stage only if the Pipeline is building a tag.
![buildingTag](https://i.imgur.com/378lFdP.png)

3. tag
The *tag* option allows you to run a *stage* directive only if the *TAG_NAME* variable matches the pattern provided.

If you provide an empty pattern, and your Pipeline is building some tag, then the *tag* option behaves the same as a *buildingTag()*.

![tag](https://i.imgur.com/JZ4zMRT.png)

To the *tag* option, you can add a *comparator* parameter to define how a given pattern must get evaluated. The options for the *comparator* parameter include *EQUALS* for simple string comparison, *GLOB* (the default) for an ANT style path glob, *orREGEXP* for regular expression matching.

![comarator](https://i.imgur.com/JbP9Liu.png)

The Publish stage in the above Pipeline runs if the tags being built match anything between release-0.0.0.0-nightly and release-9.9.9.9-nightly.

4. changelog
The *changelog* option allows you to execute a *stage* directive only if the Pipeline’s SCM changelog contains a specific regular expression pattern. The SCM changelog contains information about the list of modified files, the branch of the repository where the respective files were modified, the user who made the changes, and more.

![changelog](https://i.imgur.com/9uUBGHD.png)

5. changeset
The *changeset* option allows you to execute a *stage* directive if the Pipeline’s SCM changeset contains one or more files matching the given string or glob. The SCM changeset is a list of modified files relating to a commit.

![changeset](https://i.imgur.com/13dSS1R.png)

By default the matching is case-insensitive; to make case-sensitive, use the *caseSensitive* parameter.
```
when { changeset glob: "Simlulator_SAS.*", caseSensitive: true }
```
6. environment
The *environment* option allows you to execute a *stage* directive only when a specific environment variable is set to the given value. To give you an example use-case, let us assume that you have multiple geographical regions where you are required to deploy your successfully tested artifacts—let’s say, apac and japac. And, for all these environments, the deployment steps are different. So, using the *when* directive with *environment* condition, you can design your pipeline as shown here.

![environment](https://i.imgur.com/9vXmWEx.png)

7. equals
The *equals* option allows you to execute a *stage* directive only when an expected value equals the real value.

![equals](https://i.imgur.com/nT7HPpR.png)

8. expression
The *expression* option allows you to run a stage only when the specified Groovy expression holds true.
![expression](https://i.imgur.com/xRBayPL.png)

9. not
Use the *not* option to execute a *stage* directive when all of the nested conditions are true and must contain at least one condition.
![not](https://i.imgur.com/QOlR9f3.png)

10. allOf
Use the *allOf* option to execute a *stage* directive when all of the nested conditions are true and must contain at least one condition.
![](https://i.imgur.com/JKFI18a.png)

11. anyOf
Use the *anyOf* option to execute a *stage* directive when at least one of the nested conditions are true and must contain at least one condition.

![](https://i.imgur.com/r9Lcscp.png)

12. Evaluate When Before Entering the Stage’s Agent
By default the when condition for a stage directive gets evaluated after entering the agent defined for the stage.To change this behavior, use the beforeAgent option within the when {} block.

![](https://i.imgur.com/pcRUia1.png)

## Sequentail stages
Multiple *stage* directives in Declarative Pipeline can be defined in sequence. A *stage* directive can contain only one *steps {}, parallen {},* or *stages {}* block.
1. Simple Sequential Stages
![Simple sequential stages figure](https://i.imgur.com/HqdwQaB.png)

![Simple sequential stages code](https://i.imgur.com/lbHccu0.png)

The *pipeline {}* block compulsory has a *stages {}* block. All the *stage* directives that should run in the sequence are defined inside the *stages {}* block, one after the other.

2. Nested sequential stages
In Declarative Pipeline, it is possible to nest multiple stage directives that are in sequence inside another stage directive.
![Nested sequential stages figure](https://i.imgur.com/yLSxMZI.png)
![Nested sequential stages code](https://i.imgur.com/8iAt69F.png)
Notice the two *stage* directives that are in sequence (highlighted in bold) are not placed directly inside the parent *stage ('Stage 1.2') {...}* block. But, they are first added to a *stages {}* block, which is them placed inside the *stage ('Stage 1.2') {...}* block.

## Parallel stages
It is possible to define multiple *stage* directives in Declarative Pipeline to run in parallel with one another.
A *stage* directive can contain only one *steps {}, parallen {},* or *stages {}* block. Also, the nested stages cannot contain further parallel stages.

A *stage* directive that contains a *parallen {}* block cannot contain *agent* or *tools* sections since they are not applicable without the *steps {}* block.
1. Simple parallel stages
![Simple parallel stages figure](https://i.imgur.com/7fqNbdX.png)
![Simple parallel stages code](https://i.imgur.com/rc05gg4.png)
The pipeline {} block compulsory has a stages {} block. All the stage directives that should run in parallel are defined one after the other inside a parallel {} block. The parallel {} block is then defined inside the parent stage {} block.
2. Nested parallel stages
In Declarative Pipeline, it is possible to nest multiple stage directives that are in parallel inside another stage directive. 
![Nested parallel stages figure](https://i.imgur.com/82ksxy9.png)
![Nested parallel stages code ](https://i.imgur.com/KIxaHqF.png)

## STEPS
There are a large number of steps available for Jenkins. You can find them at https://jenkins.io/doc/pipeline/steps/

Every new Jenkins Plugin that’s compatible with the Scripted Pipeline adds a new step to the list of steps. However, not all of the steps available for Jenkins are compatible with the Declarative Pipeline. The ones that are compatible are available as a step inside the Visual Pipeline Editor.

To make the incompatible steps work with Declarative Pipeline, you need to enclose them inside the script {} block, also called the script step.

However, if you are creating a pipeline using the Visual Pipeline Editor, the script step is available as Run Arbitrary Pipeline Script inside the list of Steps.

## Script
In simple terms, the script {} block allows you to execute a Scripted Pipeline inside a Declarative Pipeline.

![script](https://i.imgur.com/Gm6EcNy.png)

There is no limit to the size of Scripted Pipeline you can run inside the script {} block. However, it is recommended to move anything bigger than a few lines to Shared Libraries. Also, any piece of code that is common across multiple pipelines must be moved to Shared Libraries.In the final chapter, you’ll learn to extend your Jenkins Blue Ocean pipelines or, shall we say, the Declarative Pipeline, using the script step and Shared Libraries.

# Shared Library
A Jenkins Shared Library is an external source control repository containing your complex Groovy code. It acts like a function that could be used on-demand inside your Declarative Pipeline.

*A Groovy Script (example.groovy)Inside Jenkins Shared Library Repository:*

```
def greet(message) {
   echo "Hello ${message}, welcome to Jenkins Blue Ocean."
}
```
*Jenkins Declarative Pipeline Utilizing the Shared Library:*
```
@Library('Example_shared_Library') _

pipeline {
  agent none
  stage ('Example') {
     steps {
        script {
           example.greet 'Readers'
        }
     }
  }
}
```
# Setting up Jenkins Blue Ocean
## Setting up Blue Ocean using docker
**Downloading the latest Jenkins Blue Ocean**
1. docker pull jenkinsci/blueocean

2. To list the downloaded docker image: docker images

Docker containers generate and use data. When a container gets deleted, its relevant data also gets lost.
To make the data persistent, we use docker volumes.
```bash
docker volume create jenkins_home
docker volume ls
docker volume inspect jenkins_home #get detailed info about docker volume
```
## Run Jenkins blue ocrean behind a reverse proxy
1. Run a Jenkins container
```bash
docker run -d --name jenkins -v jenkins_home:/var/jenkins_home jenkinsci/blueocean
```
2. Download the docker image for Nginx
```bash
docker pull nginx
```
3. Spawn a Dockr container for Nginx. Also, link the nginx container to the jenkins container using the *--link* option.
```bash
docker run -d --name ngingx -p 80:80 --link jenkins nginx
```
4. Get inside the Ningx container using the *docker exec* command
```bash
docker exec -it ningx /bin/bash
```
5. Update the Ubuntu package lists
`apt-get update`
6. Install vim text editor
`apt-get install vim`
7. Take the backup of the default.conf file inside /etc/nginx/conf.d/
```bash
cp etc/ningx/conf.d/default.conf etc/nginx/conf.d/default.conf.backup
```
8. Next, replace the content of the default.conf file with the following:
```bash
upstream jenkins {
  server jenkins:8080;
}

server {
  listen 80;
  server_name jenkins.example.com;
  
  location / {
     proxy_pass         http://jenkins;
     proxy_set_header   Host $host;
     proxy_set_header   X-Real-IP $remote_addr;
     proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
     proxy_set_header   X-Forwarded-Proto $scheme;
  }
}
```
9. Exit the Nginx container.
`exit`
10. Restart the Nginx container.
`docker restart nginx`
11. Run the following docker command to fetch the content of initialAdminPassword file.
```bash
docker exec -it jenkins /bin/bash -c \ "cat /var/jenkins_home/secrets/initialAdminPassword"
```
# Creating Pipeline
**Prerequisites**     
1. fork [example maven project](https://github.com/Apress/beginning-jenkins-blue-ocean/tree/master/Ch03/example-maven-project)
2. pulling the docker image for jenkins agent
```
docker pull nikhilpathania/jenkins_ssh_agent
```
The docker image is based on Ubuntu and comes with Git,Java JDK, Maven, and sshd installed. 
The image also contains a user account named *jenkins*.

**Creating credentials for the docker image in Jenkins**
Add credentials inside Jenkins that allow it to interact with the docker image nikhilpathania/jenkins_ssh_agent
1. Classic Jenkins dashboard -> Credentials-> System-> Global credential (unrestricted).
2. Add Credentials
3. Options
   1. Kind: Username with password
   2. Username: the username to interact with the docker image: jenkins
   3. Password: the password to interact with the docker image: jenkins
   4. ID: add a meaningful name to recognize these credentials.
   5. Descritpion: Add a meaningful description for these credentials.
![configuring the credential](https://i.imgur.com/pFaHQuZ.png)
**Installing the docker plugin**
To spawn on-demand docker containers serving as Jenkins agents, need to install the docker plugin
![Installing the Docker Plugin for Jenkins](https://i.imgur.com/nUi1uKk.png)   

**Configuring the docker plugin**
Manage Jenkins-> Configure System-> Cloud-> Add a new cloud-> Docker

Options to configure:
1. Docker Host URI: This is the URL used by Jenkins to talk to the docker host.
---
**ENABLING DOCKER REMOTE API (CRITICAL)**
The Docker remote API allows external applications to communicate with the Docker server using REST APIs . Jenkins (through the Docker Plugin) uses the docker remote API to communicate with a docker host.

To enable the Docker remote API on your Docker host, you’ll need to modify Docker’s configuration file. Depending on your OS version and the way you have installed Docker on your machine, you might need to choose the right configuration file to modify. Shown here are two methods that work on Ubuntu. Try them one by one.

**Modifying the docker.conf File**
Follow these steps to modify the docker.conf file :
1.	
Log in to your docker server; make sure you have sudo privileges.

 
2.	
Execute the following command to edit the file docker.conf :

``` 
sudo nano /etc/init/docker.conf
```
3.	
Inside the docker.conf file, go to the line containing "DOCKER_OPTS=" .

 
You’ll find "DOCKER_OPTS=" variable at multiple places inside the docker.conf file. Use the DOCKER_OPTS= that is available under the pre-start script or script section.

4.	
Set the value of DOCKER_OPTS as shown here. Do not copy and paste; type it all in a single line.
```
DOCKER_OPTS='-H tcp://0.0.0.0:4243 -H unix:///var/run/docker.sock'
```
 
The above setting binds the Docker server to the UNIX socket as well on TCP port 4243.

"0.0.0.0" makes Docker engine accept connections from anywhere. If you would like your Docker server to accept connections only from your Jenkins server, then replace "0.0.0.0" with your Jenkins Server IP.

5.	
Restart the Docker server using the following command:
```
sudo service docker restart 
```
 
6.	
To check if the configuration has worked, execute the following command. It lists all the images currently present on your Docker server.
```
curl -X GET http://<Docker Server IP>:4243/images/json
```
 
7.	
If this command does not return a meaningful output, try the next method.

 
**Modifying the docker.service File**
Follow these steps to modify the docker.service file :
1.	
Execute the following command to edit the docker.service file.
```
sudo nano /lib/systemd/system/docker.service
```
 
2.	
Inside the docker.service file, go to the line containing ExecStart= and set its value as shown here. Do not copy and paste; type it all in a single line.
```
ExecStart=/usr/bin/dockerd -H fd:// -H tcp://0.0.0.0:4243
```
 
The above setting binds the docker server to the UNIX socket as well on TCP port 4243.

"0.0.0.0" makes the Docker engine accept connections from anywhere. If you would like your Docker server to accept connections only from your Jenkins server, then replace "0.0.0.0" with your Jenkins Server IP.

3.	
Execute the following command to make the Docker demon notice the modified configuration:
```
sudo systemctl daemon-reload
```
 
4.	
Restart the Docker server using the below command:
```
sudo service docker restart
```
 
5.	
To check if the configuration has worked, execute the following command. It lists all the images currently present on your Docker server.
```
curl -X GET http://<Docker Server IP>:4243/images/json
```
---
2. Server Credentials: If your docker host requireds a login, you need to add the credentials to Jenkins using the **Add** button. However, do nothing if you are using a docker host that's running your Jenkins server container.
3. Test Connection: Click on this to test the communication between your Jenkins server and the docker host. You should see the docker version and the API version [4] if the connection is successful.
4. Enabled: A checkbox to enable/disable the current configuration.
![Configuring the Docker host URI and testing the connection](https://i.imgur.com/RKOm2V2.png)

**Add Docker Template** button. Click on it to configure the docker image that Jenkins shoudl use to spawn container.
![docker template figure](https://i.imgur.com/6NOb5kY.png)
1. Labels: The label that you type in under the Labels field gets used inside your Pipeline to define agents for your stages. In this way, Jenkins knows that it has to use docker to spawn agents. 
2. Enabled: This checkbox is used to enable/disable the current configuration. 
3. Docker Image: Add the name of the docker image that should be used to spawn agents containers. 
4. Remote File System Root: This is the directory inside the container that holds the workspace of the Pipeline that runs inside it. 
5. Usage: We would like only to build pipelines that have the right agent label, in our case it is docker.
6. Connect method : Choose to Connect with SSH option to allow Jenkins to connect with the container using the SSH protocol.
7. SSH Key: Choose use configured SSH credentials from the options to use the SSH credentials as the preferred mode of authentication.
8. SSH Credentials: From the list of options choose the credentials that you have created earlier, in the section: Creating Credentials for the Docker Image in Jenkins.
9. Host Key Verification Strategy: Choose Non verifying Verification Strategy to keep things simple. However, this is not the recommended setting for a production Jenkins server.
# Using the pipeline creation wizard, configure Jenkins Blue Ocean with various types of source code repositories.
# Using the Visual Pipeline Editor to design Pipeline.
- Downloads the source code from the Github repository
- Performs a build and some testing
- Publishes the testing results under the Pipeline's Test page
- Uploads the built artifact to Henkins Blue Ocean

**Assigning a global agent**
The pipeline that you are going to create should have two stages, and each stage is supposed to run inside a docker
container. You'll define the agents for each stage sparately in the stage's settings.
Therefore, let's keep the Pipeline's global agent setting to none.
![Assigning a global agent](https://i.imgur.com/fBKsY4k.png)

**Creating a build & test stage**
Type in the name **Build & Test** for your stage.
![Naming your stage](https://i.imgur.com/SOXKlCZ.png)

**Adding steps**
Let's add some steps to our **Build & Test** stage.
![Adding a new step](https://i.imgur.com/FR5koOy.png)
**Adding a shell script setp**
Our source code is a Maven project, and we would like to build and test it using an mvn command, which eventually
gets executed inside a shell on the Jenkins agent.
![Adding a shell script step](https://i.imgur.com/zdRO91W.png)
Paste the below code which is a maven command to build, test, and create a package out of your source code.
```bash
mvn -Dmaven.test,failure,ignore clean package
```
**Adding a stash step to pass artifact between stages**
Add another step to stash the build package and the testing report generated by the maven command.
Look for the step **Stash some files to be used later in the build**
![Adding a Stash step](https://i.imgur.com/yguB0hB.png)

Add the name "build-test-artifacts" fro your stash using the Name* field, which is mandatory.
Add the following to the Included field: **/target/surefire-reports/TEST-*.xml,target/*.jar

With this configuration you are telling Jenkins to stash any .jar file (build package) from the target directory,
and the TEST-*.xml file(test report) from the `**/target/surefire-reports/` directory on the build agent.
![configuring a Stash step](https://i.imgur.com/ZtwyY00.png)

piple line code so far:
```
pipeline {
    agent none
    stages {
        stage('Build & Test') {
            steps {
                sh 'mvn -Dmaven.test.failure.ignore clean package'
                stash(name: 'build-test-artifacts', \
                includes: '**/target/surefire-reports/TEST-*.xml,target/*.jar')
            }
        }
    }
}
```
**Assigning an agent for the build & test stage**
You'll assign a build agent for your **Build & Test** stage. The agent is going to be a docker container that will be spawn automatically by Jenkins. ONce the stage is complete, Jenkins will destroy the container.
![assigning an agent to a stage](https://i.imgur.com/8QGv9eH.png)
With the following configuration, Jenkins looks for an agent with the label **docker**. Remember the section, wherein you configured the docker plugin in Jenkins. You specified the label **docker** while configure the **Docker Agent
Template**.

**Creating a report & publish stage**
Add another stage named **Report & Publish** that will publish the testing results on the **Test** page of the Pipeline and that will publish the build package on the **Artifacts** page of the pipeline.
![Naming your stage](https://i.imgur.com/1wyyhNT.png)

**Adding an un-stash step**
Before we do anything in the **Report & Publish** stage. it is first crucial to un-stash the files that were stashed
in the previous stage. So let's add step to un-stash a stash from the previous stage.
![Adding a restore files previously stashed step](https://i.imgur.com/e0XDMmW.png)
You'll see a text field Name* where you should paste the name of your stash precisely as it was defined during its
creation.
![Configuring the Restore files previously stashed step](https://i.imgur.com/O5Do1D7.png)

**Report testing results**
The stash contains a JUnit test results .xml file that you'll publish on the pipeline's Test page. For this, we need
to add a step named **Archive Junit-formatted test results**
 
Use the TestResults* field to provide Jenkins with the path to your JUnit test result file. In our case, it is 
`**/target/surefire-reports/TEST-*.xml`

![Configuring an Archive Junit-formatted test results step](https://i.imgur.com/Skz6Yg7.png)

**Upload artifacts to blue ocean**
Add a step that will upload the build package to the Pipleine's Artifacts page. From the un-stashed files, you also
have a .jar file that is the build package.

To upload it to the Pipeline Artifacts page, use the **Archive the artifacts** step.
Use the Artifacts* filed to provide Jenkins with the path to your build package file. In our case, it is target/*.jar.Also, tick the option OnlyIfSuccessful to upload the artifacts only if the Pipeline status is green or yellow.
![Configuring the Archive the Artifacts step](https://i.imgur.com/X1ogakY.png) 

**Assigning an aget for the report & publish stage**
you’ll assign a build agent for your Report & Publish stage. The agent is going to be a docker container that will be spawn automatically by Jenkins. Once the stage is complete, Jenkins will destroy the container.

![Assigning an agent to a stage](https://i.imgur.com/zg4Jc5G.png)

You are new done with creating the pipeline. To save the changes, click on the Save button.

When you click on the **Save** button, Jenkins in the back end converts your UI configurations into a Jenkinsfile that follows the Declarative Pipeline Syntax.

![Committing your pipeline configurations changes](https://i.imgur.com/2HYkBP5.png)

**Run an artifactory server**
To spawn an Artifactory server with docker. Artifactory is a popular tool to manage and version control software build artifacts. 
1. Log in to your docker host
2. docker volume create --name artifactory_data
3. docker pull docker.bintray.io/jfrog/artifactory-oss:latest  # downloading the latest version of Artifactory community edition
4. docker run --name artifactory -d -v artifactory_data:/var/opt/jfrog/ -p 8081:8081 docker.bintray.io/jfrog/artifactory-oss:latest
5. access your Artifactory server: http://<Docker_host_ip>:8081/artifactory/webapp/#/home
6. using the admin credentials (username: admin, and password as password). Note the default example repository in Artifactory named example-repo-local.

**Installing the artifactory plugin for jenkins**
Manage Jenkins-> Manage Plugins-> Artifactory

**Configuring the artifactory plugin in jenkins**
![configuring the artifactory plugin](https://i.imgur.com/2HYkBP5.png)

**Creating a Publish to Artifactory Stage (Parallel Stage)**
Add a stage in parallel to existing **Report & Publish** stage.
![creating a new stage](https://i.imgur.com/HAq6sYq.png)
![naming your stage](https://i.imgur.com/vXUpngZ.png)
Your new stage first downloads the stash files from the previous stage, and then it publishes the built artifacts to
the Artifactory server.

**Adding a scripted pipeline step**
does two things
1. it fetches the stash files from the previous stage.
2. it runs a filespec that uploads the build package to the Artifactory server.
![add a scripted pipeline step](https://i.imgur.com/9W9fTUQ.png)
Script to Un-Stash Built Artifacts and Upload Them to the Artifactory Server
```
unstash 'build-test-artifacts'
def server = Artifactory.server 'Artifactory'
def uploadSpec = """{
  "files": [
    {
      "pattern": "target/*.jar",
      "target": "example-repo-local/${BRANCH_NAME}/${BUILD_NUMBER}/"
    }
  ]
}"""
server.upload(uploadSpec)
```
```bash

███████╗██╗  ██╗██████╗ ██╗      █████╗ ██╗███╗   ██╗
██╔════╝╚██╗██╔╝██╔══██╗██║     ██╔══██╗██║████╗  ██║
█████╗   ╚███╔╝ ██████╔╝██║     ███████║██║██╔██╗ ██║
██╔══╝   ██╔██╗ ██╔═══╝ ██║     ██╔══██║██║██║╚██╗██║
███████╗██╔╝ ██╗██║     ███████╗██║  ██║██║██║ ╚████║
╚══════╝╚═╝  ╚═╝╚═╝     ╚══════╝╚═╝  ╚═╝╚═╝╚═╝  ╚═══╝
```
The code line unstash 'build-test-artifacts' downloads the previously stashed package. The rest of the code is a Filespec that uploads the target/*jar file, which is our built package file, to the Artifactory server on the repository example-repo-local.

Notice that the target path contains Jenkins Global variables, ${BRANCH_NAME} and ${BUILD_NUMBER}, representing the branch name and build number, respectively. Doing so uploads the built artifacts to a unique path on Artifactory every single time a pipeline runs.

**Assigning an Agent for the Publish to Artifactory Stage**
you’ll assign a build agent for your Publish to Artifactory stage. The agent is going to be the docker container that gets spawned automatically by Jenkins. Once the stage is complete, Jenkins destroys the container.
![Assigning an agent to a stage](https://i.imgur.com/l3VMpHu.png)

Final pipeline code:
```
pipeline {
  agent none
  stages {
    stage('Build & Test') {
      agent {
        node {
          label 'docker'
        }

      }
      steps {
        sh 'mvn -Dmaven.test.failure.ignore clean package'
        stash(name: 'build-test-artifacts', includes: '**/target/surefire-reports/TEST-*.xml,target/*.jar')
      }
    }
    stage('Report & Publish') {
      parallel {
        stage('Report & Publish') {
          agent {
            node {
              label 'docker'
            }

          }
          steps {
            unstash 'build-test-artifacts'
            junit '**/target/surefire-reports/TEST-*.xml'
            archiveArtifacts(artifacts: 'target/*.jar', onlyIfSuccessful: true)
          }
        }
        stage('Publish to Artifactory') {
          agent {
            node {
              label 'docker'
            }

          }
          steps {
            script {
              unstash 'build-test-artifacts'

              def server = Artifactory.server 'Artifactory'
              def uploadSpec = """{
                "files": [
                  {
                    "pattern": "target/*.jar",
                    "target": "example-repo-local/${BRANCH_NAME}/${BUILD_NUMBER}/"
                  }
                ]
              }"""

              server.upload(uploadSpec)
            }

          }
        }
      }
    }
  }
}
```
![Committing your pipeline configurations changes](https://i.imgur.com/id9vL50.png)
![build artifact uploaded to Artifactory](https://i.imgur.com/Ec4LinF.png)

**Running a pipeline for a pull requrest**
Jenkins Blue Ocean can detect pull requests on your Github repositories and run a pipeline with it for you. The 
pipeline run result (fail/pass/canceled) gets reported back to your source code repository.

The person who is reponsible for accepting the pull request can then decide based on the pipeline run result whether
he should merge the new changes into the destination branch or not.
![pull request](https://i.imgur.com/kUvGGZM.jpg)

## Issue
pull requests not showing
fix by: ![](https://i.imgur.com/63ETSbk.png)
