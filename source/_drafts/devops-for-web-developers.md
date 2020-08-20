---
title: devops-for-web-developers
tags:
---
# DevOps for web developers
## Understanding the Devops movement
Devops life cycle: all about continuous: continuous integration, configuration management, continous delivery, or continous deployment, continous monitoring, continuous feedback tools and technologies

**Possible issues**
- Artifacts are verifed on the development environment
- Each team take a different approach for setting up the runtime environment
- The deployment process needs to be documented for future usage
- Maintaining the documentation is a time-consuming task
- Both team can use different automation techniques
- Both team are unaware of the challenges faced by each other
- The development team get another request for a feature implementation for bug fix
- Poor collaboration causes many issues in the application's deployment to different environments

Agile deveopment , or the Agile methodology is a method of building an application by empowering individuals, and encouraging interactions, giving importance to working software, customer collaboration using feeback for improvement in subsequent steps, and responding to change in an efficient manner. It emphasizes customer satifaction through continuous delivery in small interfactions for specific features, in short timelines or sprints.

**Collaboration**
- Empasizes communication, collaboration, and integration between software developers and IT operations.
- Promotes collaboration, & collaboration is facilitated by automation
- It is a combination of agile practices & processes leveraging the benefits of cloud solutions.
- Testing methodologies help us meet the golas fo continously integrating, developing, building, deploying, testing, and releasing applications
- Provides transparency in the form of a platform for collaboration across teams, such as usiness analysts, developers, and testers

**Cloud computing**
- The cloud helps us overcome this hurdle by providing flexible on-demand resources and environments
- The cloud provides a repository of software tools
- The entire development, test, and production environments can be monitored
- Easy to recreate the production environment exactly in an automated fashion
- Provide a distributed agile environment in the cloud, leading to continous accelerated delivery

![devops lifecycle](https://i.imgur.com/SG8RUPl.png)
**Build Automation**:  An automated build helps us create an application build using build automation tools such as Apache Ant and Apache Maven.
- Compiling source code into class files or binary files
- Providing references to third-party library files
- Providing the path of configuratin files
- Packaging class files or binary files into WAR files in the case of Java
- Executing automated test cases
- Deploying WAR files on local or remote machines
- Reducing manual effort in creating the WAR file
![build atomation pipeline](https://i.imgur.com/XltBSmR.png)

**Continuous Integration**:CI is a software engineering practice where each check-in made by a developer is verified by two mechanisms.
- Pull mechanism: executing an automated build at a scheduled time
- Push mechanism: executing an automated build when changes are saved in the repository
![ci pipeline](https://i.imgur.com/OSwLu72.png)

**Needs for CI**
- It helps us identify bugs or errors in the code at a very early stage of development
- It is far easier to cure and fix issues
- CI is a development practice that requires developers to integrate code into a shared repository several times a day
- CI is a significant part for the release-management strategy of any organization that wants to develop a DevOps culture

**Benefits CI**
- Automated integration with pull or push mechanism
- Repeatable process without any manual intervention
- Automated test case execution
- Coding standard verification
- Execution of scripts based on requirement
- Quick feedback: build status notification to stakehoulders via e-mail
- Teams focused on their work and not in the managing processes

Jenkins, Apache Continuum, Buildbot, GitLab CI, and so on, are some examples of open-source CI tools. Anthill Pro, Atlassian Bamboo, TeamCity, Team Foundation Server, and so on, are some examples of commercial CI tools.

**Best practices**
- Maintain a code repository such as Git or SVN
- Check-in third-party JAR files, build scripts, other artifacts
- Execute builds fully from the code repository
- Automate the build using Maven or Ant for Java
- Make the build self-testing
- Commit all changes at least once a day per feature
- Every commit should be built to verify the integrity of changes
- Authentication and authorization
- Use alphanumeric characters for build names and avoid symbols
- Helps to assign build execution to slave instances
- Backup the home directory of the CI server regularly
- Make sure the CI server has enough free disk space
- Do not schedule multiple jobs to start at the same time, or use a master-salve concept
- Set up an e-mail, SMS, or Twitter notification to specific stakeholders of a project

**Configuration managment**
- CM is keeping track or versions of details related to the state of specific nodes
- A centralized change can trigger this or nodes can communicate with the CM server
- CM tools make this process efficient when only changed behavior is updated
- The entire installation and modification isn't applied again to the server nodes
- Chef, Puppet, Ansible, and Salt are the popular CM tools.
