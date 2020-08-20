---
title: DevOps
date: 2019-07-17 21:23:02
tags: [Devops]
---
**What is DevOps?**
- Speed and agility enable organizations to better serve their customers and compete more effectively in the market
- Combination of cultural philosophies, practices, and tools
- Increases an organization's ability to deliver applications and services at high velocity
- Evolves and imprvoes products faster

**Why DevOps?**
- Antomate manual tasks, help teams manage complex environments at scale, and keep engineers in control of the velocity that is enabled by DevOps:
  - Speed
  - Rapid delivery
  - Reliability
  - Scale
  - Improved collaboration
  - Security

Standard Continuous delivery (CD) techniques
- Blue/Green deployment (where "live" and "last" deployments are maintained on live) 
Blue-green deployment is a technique that reduces downtime and risk by running two identical production environments called Blue and Green.

At any time, only one of the environments is live, with the live environment serving all production traffic. For this example, Blue is currently live and Green is idle.

As you prepare a new version of your software, deployment and the final stage of testing takes place in the environment that is not live: in this example, Green. Once you have deployed and fully tested the software in Green, you switch the router so all incoming requests now go to Green instead of Blue. Green is now live, and Blue is idle.

This technique can eliminate downtime due to app deployment. In addition, blue-green deployment reduces risk: if something unexpected happens with your new version on Green, you can immediately roll back to the last version by switching back to Blue.

- Phoenix deployment (where whole system are rebuilt on each release).
![devops tools](https://i.imgur.com/DXtJZOm.png)

**Goals**
- Culture for collaboration
> lack of collaboration is one of the root causes of the issues
- Automate
> Manual tests that consume resources are better left to machines. This frees time that is better spent elsewhere, and provides a better environment by relinquishing mundane tasks
- Optimize and reduce issue in SDLC(software development life cycle)
> The processes being comprised in a logical order allows for optimizations to be made from recorded metrics.
- Consistency in process
> This stems mostly from automation, and provides the foundation to ensure quality. It also provides a certain level of peace of mind, having confidence that the same process that successfully ran last time will run the same the next.
- Improve quality and security
> Automation combined with consistency in process, along with the tools and practices in place to perform the necessary testing and scans, removes the possibility of human error from subsequent processes
- Improve deployment frequency
> Agile methodologies proved effective for development, and sparked the idea to apply similar principles to other areas. Deployment frequency has always been a target for efficiency, as shown by the migration to waterfall, where deployments were seldom; to hybrid methodologies that produced releases four or five times per month; to agile, where deployments are dependent upon sprint links. With DevOps, there's a potential to release multiple times per day.

**DevOps Methodologies and Concepts**
- Automation
  - Not duplicate of goals
  - Automation in context of applying automation to deveopment, integration, and deployment process
- CI/CD/CD
  - CI: continuous integration
     - Focusses on sub process of SDLC to build features and fixes perform preliminary testing then merge to master if successful
  - CD: continuous delivery
     - tail end of CI
     - refers to archiving build artifacts
  - CD: continous deployment
     - deploy all necessary artifacts and perform any configuration
- Infrastructure as code, configuration as code: fail fast
  - Don't waste resources on something that will fail
  - Organize and optimize for efficiency
- Frequent feedback
  - Compilation of a series of small and frequent feedback loops

**DevOps engineer's role within a devops organization**
- Continually research, refine, and create conceopts, methodologies, practices, and tools in order to optimize the SDLC.
- Implement core standards, plicies, and tools based on the previous
- Asses infrastructure requirements"
  - Global and application sacle
- Crate and manage infrastructure
- Coordinate with other teams
- Troubleshoot issues with code and infrastructure
- Work with one or more teams to:
  - Assess their current state
  - Formulate end goals
  - Adapt to requirements
  - Develop a plan of implementation
  - Formulate milestones
  - Provide instruction on the core standards, policies, and tools
  - Develop a pipeline
  - Help to change their code and processes to work with the plan

**philosophy**
DevOps is not something you do. It is something you are. 

Devops culture is the one based on a set of principles, hierarchy of rules, by which each person operates.
A DevOps culture is one that allows more freedom but more responsibility.

**Devops lifecycle**
- Continuous integration
  - Central repository
  - Continuous compiling, testing
  - Code verification
  - Identifying bugs early
  - Software in smaller chunks
  - Easy integration
Continuous integration (CI) requires developers to integrate code into a centralized repository as they finish coding and successfully pass unit testing, several times per day. The end goal is to create small workable chunks of code that are validated and integrated back into the code repository as frequently as possible. 

- Configuration management
  - System changes
  - Multiple servers
  - Tracking changes
  - Types of CM tools
  - Scripted builds
  - Identical development and production environments

- Continous delivery
  - Deploying the application
  - Incremental or small changes
  - Compatible with schedule releases
  - Every change-ready to deploy
Continuous delivery simply means that every change is ready to be deployed to production as soon as automated testing validates it.
Note there are two manual checkpoints in this example. One is for a technical decision to approve the code before initiating activities in the CI environment. The second is a business decision to accept the changes and continue with the automated steps to production deployment.
![reference pipeline-continous delivery](https://i.imgur.com/XzLyd0g.png)
**Deploying to production**
- Canary releases
> Continuous delivery deploys many builds to production. In a canary deployment, the new code is delivered only to a percentage of the existing infrastructure. For example, if the system is running on 10 load-balanced virtual servers, you can define a canary cluster of one or two servers. This way, if the deployment is not successful due to an escaped defect, it can be caught before the build is deployed to all of the servers. Canary releases are also used for pilot features to determine performance and acceptance prior to a full rollout.

- Blue/green deployment
> This is a zero-downtime deployment technique that involves a gradual release to ensure uninterrupted service. The blue/green approach is effective in virtualized environments, especially if IaaS is used. Although a blue/green deployment is a deep topic that deserves an entire chapter on its own, simply put, it includes maintaining two identical development environments—Blue and Green. One is a live environment for production traffic, whereas the other is used to deploy the new release. In our example, let’s say that Green is the current live production environment and Blue is the idle identical production environment. After the code is deployed and tested in the Blue environment, we can begin directing traffic of incoming requests from Green (current production) to Blue. You can do this gradually until the traffic redirect is 100 percent to the Blue environment. If unexpected issues occur during this gradual release, we can roll back. When it is completed, the Green environment becomes idle and the Blue environment is now the live production environment.

- Continous monitoring
Continous monitoring is the practice that connects operations back to development,providing visibility and relevant data throughout the development lifecycle including production monitoring. continuous monitoring aims to reduce the time between identification of a problem and deployment of the fix.
Monitoring begins with **Sprint 1** and should be integrated into the development work. As the system is built, monitoring solutions are also designed. 

**four different types of continuous monitoring**
- Infrastructure monitoring
> Visualize infrastructure events coming from all computing resources, storage and network, and measure the usage and health of infrastructure resources. **AWS CloudWatch** and **CloudTrail** are examples of infrastructure monitoring tools.
- Application performance monitoring (APM)
> Target bottlenecks in the application’s framework. **Appdynamics** and **New Relic** are industry-leading APM tools.
- Log management monitoring
> Collect performance logs in a standardized way and use analytics to identify application and system problems. **Splunk** and **ELK** are two leading products in this area.
- Security monitoring
> Reduce security and compliance risk through automation. Security configuration management, vulnerability management, and intelligence to detect attacks and breaches before they do serious damage are achieved through continuous monitoring. For example, **Netflix’s Security Monkey** is a tool that checks the security configuration of your cloud implementation on AWS.

**three major setps**
1. monitoring
2. an alert system to warn the team about a problem
3. actions to take when an alert occurs
- Continous testing
  - Speed and quality
  - Testing incremental changes
  - Automated tests
  - Tests to be atomic: small test
     - Continous testing during development (e.g. use open source tools like Selenium for testing)
     - Confidence to release
     - Integration with CI
- Continous deployment
  - Superset of Continuous Delivery
  - Deploying to production
  - Automates the deployment pipeline
Unlike continuous delivery, which means that every change is deployable but might be held back because of business considerations or other manual steps, continuous deployment strives to automate production deployment end to end. With this practice, confidence in the automated tests is extremely high, and as long as the code has passed all the tests, it will be deployed.
![cd](https://i.imgur.com/Aq1VaJb.png)

DevOps: culture, automation, measurement, and sharing (CAMS)

**DevOps architecutres and practics**
From the DevOps movement, a set of software architectural patterns and practices have become increasingly popular. The primary logic behind the development of these architectural patterns and practices is derived from the need for scalability, no-downtime deployments, and minimizing negative customer reactions to upgrades and releases. Some of these you may have heard of (microservices), while others may be a bit vague (blue-green deployments).
