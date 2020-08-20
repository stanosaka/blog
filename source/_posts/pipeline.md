---
title: pipeline
tags: [pipeline]
---
Pipeline

Represents a part of:
- Software delivery
- Quality assurance process

Benefits:
- Operation grouping
- Visibility
- Feedback
**Automated deployment pipeline**
3 stages:
code change-> Continuous Integration-> Automated acceptance testing-> Configuration management
1. Continuous Integration
The continuous integration phase provides the first feedback to the developers. It checks out the code from the repository, compiles it, runs unit tests and verifies the code quality. If any step fails the pipeline execution is stopped and the first thing the developer should do is fix the continuous integration build. The continuous integration pipeline is usually the starting point.
- first feedback
- checks code
- starting point
- simple to setup

2. Automated acceptance testing
- Suites of tests
- A quality gate
- Pipeline execution is stopped if test fails
- Prevents movement
- Lot of confusion

![Agile testing matrix](https://i.imgur.com/KGWFQPx.png)
1. Acceptance testing
- Represent functional requirements
- Wrtten in the form of stories or examples
2. Unit Testing
- Provide the high-quality software
- Minimize the number of bugs
3. Exploratory Testing
- Manual black-box testing
- Breaks or improves the system
4. Non-functional testing
- Represent properties:
  - Performance
  - Scalability
  - Security

3. Configuration management
- Replaces the manual operations
- Responsible for tracking and controlling changes
- Solution to the problems
- Enable storing configuration files
Configuration management is a solution to the problems posed by manually deploying and configuring applications on the production. Configuration management tools enables storing configuration files in the version control system and tracking every change that was made on the production servers.

**Technical and development prerequisites**
- Automated build, test, package, and deploy operations
- Quick pipeline execution
- Quick failure recovery
- Zero-downtime deployment
- Trunk-based deployment

**Building the continuous delivery process**
Jenkins
- Popular automation server
- Helps create automated sequence of scripts
- Plugin-oriented

Ansible
Helps with:
- Software provisioning
- Configuration management
- Application deployment

Java
- Most popular programming language
- Develop with the Spring framework
- Gradle-build tool
Jenkins is by far the most popular automation server on the market. It helps to create continuous integration and continuous delivery pipelines and in general any other automated sequence of scripts. Highly plug-in oriented it has a greater community which constantly extends it with new features. Another one is Ansible. Ansible is an automation tool that helps with software provisioning configuration management and application deployment. Next comes Java. Java has been the most popular programming language for years, that is why it is being used for most code examples. Together with Java, most companies develop with the Spring framework so we used it to create a simple web service needed to explain some concepts. Gradle is used as a build tool. It is less popular than Maven however trending much faster.


**Pipeline elements**
- Basic elements
- Stage - Logical separation
- Setp - Used to visualize process
![elements](https://i.imgur.com/J7xJ8KG.png)

```
pipeline {
   //agent any
   agent {
      label 'linux'
   }
   stages {
      stage('First Stage') {
          steps {
              echo 'Step 1. Hello World'
          }
      }
      stage('Second Stage') {
          steps {
              echo 'Step 2. Second time Hello'
              echo 'Step 3. Third time Hello'
          }
      }
   }
   post {
     always {
        echo "Pipeline executed!"
     }
   }
}
```
- Define pipeline structure
A declarative pipeline is always specified inside the pipeline block and contains sections, directives and steps. 

A declarative pipeline has a simplified and opinionated syntax on top of the pipeline sub-system.

**Sections**
- Keywords are:
  - Agent-defines on which agent the pipeline or the stage will be executed
  - Stages-defines the stages of the pipeline
  - Step
  - Post-defines the post build steps

Sections define the pipeline structure and usually contain one or more directives or steps. They are defined with the key words, stages, step and post.

Stages defines a series of one or more stage directives.

Steps defines a series of one or more step instructions.

Steps are the most fundamental part of the pipeline they define the operations that are executed so they actually tell Jenkins what to do.

Defined using:
- sh: executes the shell command
- custom
- script

Posts defines a series of one or more step instructions that are run at the end of the pipeline build.

**Directives**
- Expresses the configuration of a pipeline or its parts
- Defined by:
  - Agent: specifies where the execution takes place
  - Triggers: define automated ways to trigger the pipeline and can use cron to set the time based scheduling
  - Options: specified pipeline specific options
  - Environment: deinfes a set of key values used a s environment variables
  - Parameters: define a list of user input parameters
  - Tools: defines the tools configured on Jenkins
  - Stage: allows for logical grouping of steps; contains steps and agent 
  - When: determines whether the stage should be executed depending on the given condition

```
pipeline {
   agent any
   tools {
      maven 'M3'
   }
  
   parameters {
      string(name: 'VERSION',
          defaultValue: '1.0.0',
          description: 'What is the version to build?')
   }
  
   stages {
      stage('Build'){
          steps {
             sh "./build.sh ${params.VERSION}"
          }
      }
   }
}
```
          
## Commit Pipeline
- Basic continuous integration process
- Results in a report about the build
- Runs after each change in the code
- Consume a reasonable amount of resources
- Starting point

Since it runs after every change in the code, the build should take no more than five minutes and should consume a reasonable amount of resources. The commit phase is always the starting point of the continuous delivery process and it provides the most important feedback cycle in the development process.

In the commit phase a developer checks in the code to the repository. 
The continuous integration server detects the change and the build starts.
The most fundamental commit pipeline contains three stages 
- checkout
- compile
- unit test

## Code Quality Stages
**Extending continuous integration**
- The most widely used are: Code coverage and Static analysis

**Code coverage**
- Scenario:
   - Nobody writes unit tests
   - Passes all the builds
- Solution:
   - Add coverage code tools
- Creates report
- Make build fail
**Available tools**
- JaCoCo
- Clover
- Cobertura
**Static Code Analysis**
- Checks without execution
- Checks number of rules
- Rules apply to wide range of aspects
- Popular tools are Checkstyle, FindBugs, and PMD

**SonarQube**
- Quality management tool
- An alternative
- Aggregates different code analysis frameworks
- Has own dashboards
- User friendly web interface

## Triggers and notifications
**Triggers**
- Automatic action to start the build
- Manay options to choose from
- Three types:
  - External
  - Polling SCM
  - Scheduled build
**External Trigger**
- Natural to understand
- Starts the build after it's called by the notifier

GitHub -> trigger-> Jenkins

**Polling SCM**
- Periodically calls Github
- Sound counter-intuitive
Polling SCM trigger is less intuitive.
![Polling SCM figure](https://i.imgur.com/s6L2bbo.png)
Jenkins periodically calls GitHub and checks if there were any push to the repository. Then it starts the build.

**Scheduled build**
- Jenkins runs the build periodically
- No communication with any system
- Implementation of scheduled build is same as polling SCM
- Used for the commit pipeline

**Notifications**
- Lot

## Team development strategies
**Development workflows**
- Put the code into the repostitory
- Depends on many factors
- Classify into three types:
  - ![Trunk-based workflow](https://i.imgur.com/WXIl9qU.png)
  - ![Branching workflow](https://i.imgur.com/Oiury7J.png)
  - ![Forking workflow](https://i.imgur.com/5Uo3Ac9.png)
**Feature toggle**
Feature toggle is a technique that is an alternative to maintaining multiple source code branches such that the feature can be tested before it is completed and ready for use. It is used to disable the feature for users but enables it for developers while testing. Feature toggles are essentially variables used in conditional statements the simplest implementation of feature toggles are flags and the if statements.

**acceptance testing**
Acceptance testing is a test performed to determine if business requirements or contracts are met. 
- Invovles black-box testing
- Imply the acceptance of the software delivery
- Also called UAT
- Rely on manual steps
- Reaonable to run them as programmed repeatable operations
**Artifact repository**
While the source control management stores the source code the artifact repository is dedicated for storing software binary artifacts, for example, compiled libraries or components later used to build a complete application.
- Store binaries on a separate server due to:
 - File size
 - Versions
 - Revision mapping
 - Packages
 - Access Control
 - Clients
 - Use cases
![working](https://i.imgur.com/5gDLWLc.png)
**private docker registry**
```
mkdir -p certs

openssl req \\n  -newkey rsa:4096 -nodes -sha256 -keyout certs/domain.key \\n  -x509 -days 365 -out certs/domain.crt

docker run -d -p 5000:5000 --restart=always --name registry -v `pwd`/auth:/auth -e "REGISTRY_AUTH=htpasswd" -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" -e REGISTRY_AUTH_HTPASSWD_PATH=/auth/passwords -v `pwd`/certs:/certs -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/domain.crt -e REGISTRY_HTTP_TLS_KEY=/certs/domain.key registry:2

docker tag ubuntu_with_python stanosaka/ubuntu_with_python:1
docker push stanosaka/ubuntu_with_python:1
```

**Acceptance test in pipeline**
![process](https://i.imgur.com/06nJoTO.png)
1. The developer pushes a code change to Github.
2. Jenkins detects the change, triggers the build and checks out the current code.
3. Jenkins executes the commit phase and builds the Docker image
4. Jenkins pushes the image to Docker Registry
5. Jenkins runs the Docker container in the staging environment

**Configuration management**
- Process of controlling configuration changes
- Used to refer to the software and the hardware

Application Configuration| Infrastructre Configuration
Decides how the system works| Server infrastructure and environment configuration
Expressed in the form of flags| Takes care of the deployment process
![working](https://i.imgur.com/L4k4cb0.png)
**Traits**
- Automation
- Version Control
- Incremental changes
- Server provisioning
- Security
- Simplicity

