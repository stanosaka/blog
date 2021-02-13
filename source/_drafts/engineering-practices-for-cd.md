---
title: engineering-practices-for-cd
tags:
---
by Neal Ford
# Deployment pipelines
**Deployment pipeline**
Fast, automated feedback on the production readiness of your application every time there is a change-
to code, infrastructure, or configuration.

**Prerequisites**
comprehensive
configuration management
excellent automated testing at all levels
continuous integration
![commit stage](https://i.imgur.com/RYgasA1.png)
Run against each check-in
![Pipeline construction](https://i.imgur.com/3k3gXqg.png)
![UAT stage](https://i.imgur.com/HvD88Wv.png)
End-to-end tests in production-like environment

Triggered when upstream stage passes

First DevOps-centric build

![Manual stage](https://i.imgur.com/tAmoFi3.png)
UAT, staging, integration, production, ...
Push versus Pull model
Deployments self-serviced throught push-button process

![whole workflow](https://i.imgur.com/U3b7TrA.jpg)

Machinery
continous integration ++
Jenkins
gocd.org
![gpcd](https://i.imgur.com/pGhuFdv.png)
![deployment pipeline](https://i.imgur.com/UmHIC6X.png)
![engineering practices](https://i.imgur.com/bZ6QNiX.png)

# Continuous integration for databases
![Fast/slow tests](https://i.imgur.com/MqSSffd.png)
![pattern](https://i.imgur.com/c0s6npf.png)

Question:
What charact

What does agility means as an architecture characteristic?
- agility
- deployability
- modularity
- testability
- controlled coupling




**continous integration**
everyone commits to trunk at least once per day

*teams using feature branching aren't doing continous integration*

- canonical integration point
- useful platform to do other stuff
- unit testing
- functional testing
- integration testing
- automated machine provisioning
- deployment to production?



**Continuous delivery metrics**
- Lead time
the time between the initation and completion of a productin process
- cycle time
the total elapsed time to move a unit of work from the beginning to end of a phsical process
