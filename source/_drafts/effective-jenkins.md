---
title: Effective Jenkins
date: 2019-07-26 22:19:23
tags:
  - Jenkins
  - Devops
  - CD/CI
  - Pipeline
---
# Devops and the delivery pipeline
**How did we get here to DevOps?**
- At the 2008 Agile Toronto conference, Andrew Shafer and Patrick Debois introduced the term DevOps in their talk on Agile intrastructure
- Until 2009, most of the IT operations and development teams were used to work in silos
**What is DevOps?**
Devops is a culture of collaboration between software engineering and IT operations as well as the practice of using
software enginerring concepts where applicable in IT operations -Lars Fronius

The continuous delivery pipeline: enables a constant flow of changes into production by means of an automated
software production line, by breaking down the software delivery process into stages.

**Typical delivery/deployment pipeline** contains three parts:
- Build automation: responsible for compiling and packaging the software
- Test automation: responsible for executing unity, integration, and functional or non-functional tests.
- Deployment automation: responsible for installing and provisioning new environments for the application.

Once someone has committed something and all of the three setps has passed successfully, we should have a new
version of the application available in a production like environment.

**Default interaction with Jenkins**
- Historically, the default interaction model with Jenkins has been using web UI, requiring users to create
jobs and fill in the details manually throught a web browser
- Disadvantages:
  - Huge effort to create and manage jobs
  - CI configuration separated from the code

**Pipleline as Code**
- After the introduction of the Pipeline plugin, now it's possible to implement an entire delivery/deployment
pipeline in a Jenkinsfile and store that alongside the software code, treating their pipeline as another piece 
of code checked into source control
- Advantages:
  - Checking the pipeline definition into source control
  - Support for extending the dsl by means of the shared libraries feature


