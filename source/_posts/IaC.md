---
title: IaC (Infrastructure as Code)
date: 2020-12-08 12:07:49
tags:
  - IaC
  - devops
  - SRE
---
# Infrastructure automation tool registries
- Chef Infra Server
- PuppetDB
- Ansible Tower
- Salt Mine

# General-purpose configuration registry products
- Zookeeper
- etcd
- Consul
- doozerd

# Handling secrets as parameters
## Encrypting secrets
- [git-crypt](https://oreil.ly/-ZBmS) 
- [blackbox](https://oreil.ly/ZT99R) 
- [sops](https://oreil.ly/7md-J) 
- [transcript](https://oreil.ly/2os70)

## Disposable secrets
- HashiCorp Vault

# Continuously Test and Deliver
## Delivery pipeline software dnd services
### Build server
- Jenkins
- Team City
- Bamboo
- Github Actions
### CD software
- GoCD
- ConcourseCI
- BuildKite
### SaaS services
- CircleCI
- TravisCI
- AppVeyor
- Drone
- BoxFuse
### Cloud platform services
- AWS CodeBuild(CI)
- AWS CodePipeline(CD)
- Azure Pipelines
### Source code repository services
- Github Actions
- GitLab CI and CD
### Evaluating tools
- Atlantis (manage pull requests for Terraform projects)
- Terraform Cloud
- WeaveWorks(managing Kubernetes clusters)

## Deployment packages
| Target runtime          | Example packages                                                     |
|-------------------------|----------------------------------------------------------------------|
| Server oprating system  | Red Hat RPM files, Debian .deb files, Windows MSI installer packages |
| Language runtime engine | Ruby gems, Python pip packages, Java .jar, .war, and .ear files      |
| Container runtime       | Docker images                                                        |
| Application clusters    | Kubernetes Deployment Descriptors, Helm charts                       |
| FaaS serverless         | Lambda deployment package                                            |
|                         |                                                                      |

