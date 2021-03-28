---
title: Jenkins Pipeline
date: 2019-07-29 11:00:54
tags:
  - Jenkins
  - devops
  - CICD
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
