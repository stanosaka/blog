---
title: aws ec2 init setup
tags: 
  - aws
  - ec2
date: 2016-08-17 14:07:13
---

#### ASW ec2 init setup

Managing User Accounts on Your Linux Instance
```
[ec2-user@ip-172-31-17-193 ~]$ sudo -i
[root@ip-172-31-17-193 ~]# adduser stan
[root@koopa ~]# passwd stan
[root@ip-172-31-17-193 ~]# su - stan
[stan@ip-172-31-17-193 ~]$ mkdir .ssh
[stan@ip-172-31-17-193 ~]$ chmod 700 .ssh
[stan@ip-172-31-17-193 ~]$ vim .ssh/authorized_keys
[stan@ip-172-31-17-193 ~]$ chmod 600 .ssh/authorized_keys
[root@ip-172-31-17-193 ~]# groupadd auegg
[root@ip-172-31-17-193 ~]# usermod -a -G auegg stan
[root@ip-172-31-17-193 ~]# visudo
#add the following line

%auegg ALL=(ALL) ALL
[root@ip-172-31-17-193 ~]# service sshd restart

```

#### ssh to aws without key pairs
```
vim /etc/ssh/sshd_config
PasswordAuthentication yes
Reload ssh daemon
service sshd reload
```

#### set root login
```
vi /root/.ssh/authorized_keys
Delete the lines at the begining of the file until you get to the words ssh-rsa
vim /etc/ssh/sshd_config
PermitRootLogin yes
```

# the 5 pillars of aws weel-architected framework
1. Operationa Excellence
Design Principles:
- Perform operations as code
- Annotate documentation
- Make frequent, small, reversible changes
- Refine operations procedures frequently
- Anticipate failure
- Learn from all operation failures
Best practices: 
Operations teams need to understand their business and customer needs so they can support business outcomes.
Ops creates and uses procedures to repond to operational events, and validates their effectiveness to support
business needs. Ops also collects metrics that are used to measure the achievement of desired business outcomes.

2. Security
Design Principles:
- Implement a strong identity foundation
- Enable traceability
- Apply security at all layers
- Automate security best practices
- Protect data in transfit and at rest
- Prepare for security events

The [AWS Shared Responsibility Model](https://aws.amazon.com/compliance/shared-responsibility-model/) enables organizations to achieve
security and compliance goals.

3. Reliability
Design Principles:
- Test recovery procedures
- Automatically recover from failure
- Gtop guessing capacity
- Manage change in automation

4. Performance efficiency
Design principles:
- Democratize advanced technologies
- Go global in minutes
- Use serverless architectures
- Experiment more often
- Mechanical sympathy

5. Cost optimization
- Adopt a consumption model
- Measure overall efficiency
- Stop spending monty on data center operations
- Analyze and attribute expenditure
- Use managed services to reduce cost of ownership

# Identity and access management
**IAM overview**
AWS API
Access ID + secret key

CLI, SDKs and Management console call API.

Users
- Are created and exist within IAM service
- Login to management console
- Can have long-term access keys


