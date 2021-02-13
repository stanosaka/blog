---
title: aws_sysops_admin
tags: [aws certified]
---
# Cloudformation Basics
Flagship service
- Infrastructure as Code (JSON, YAML)
- Integrated with most of AWS
- Exceptions are documented
- Deploy infrastructure and changes
- Does NOT operate on data

# CloudFormation template
- Description
- Metadata
- **Parameters**
- **Mappings**
- **Conditions**
- Transform
- **Resources**
- **Output**

**Cloudformation study hints**
1. How can your tempate be used in multiple regions?
2. What happens when the template creation fails?
3. What happens when the template update fails?
4. Which designs require explicit dependencies?
5. Which resources are replaced upon stack update?
6. Which resources are retained upon stack delete?
7. What are change

# Opsworks basics
configuration management + IAC
- Chef automate
- puppet enterprise
- Stacks consist of layers
  - EC2
  - Elastic Load Balancing
  - RDS
  - ECS
  - Custom
# Elastic beanstalk key concepts
- Manages platform
  - ELB
  - Auto scaling
  - RDS
- Used for resource create, update, and delete actions
- Still requires OS management
- Does not address backups

# SSM run command basic
- Runs manual or scheduled tasks
- Works in hybrid environments
- Parallelized
- Track results and erros
- Easier to troubleshoot in bulk than manual operations
- Requires agent

# ECS basic
- Deploy containers without managing infrastructure
- Supports Docker and Windows containers
- Choice of deployment via EC2 or Fargate
# EKS basic
- Similar to ECS, but uses Kubernetes
- Deploy Docker containers without managing infrastructure
- Choice of deployment via EC2 or Fargate
- Supports existing VPC infrastructure
- Hybrid infrastructure support
# CodeDeploy basics
- Deploy to ec2,lambda, or on-premises
- File and command-based framework
- Rolling updates
- Blue/green deployments
- Stop and rollback
- Does not provision network or compute infrastructure
![Blue-green deployment](https://i.imgur.com/HgZZxfn.png)
How many other possibilities?
- ECS
- Opsworks
- CodeDeploy

# Manage backups
- Cli
- SSM run command
- Lambda functions

# DR processes
Learn the 4 DR scenarios
- Backup and restore
- Pilot light
- Warm standby
- Multi-site solution
