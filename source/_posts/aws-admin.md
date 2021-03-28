---
title: aws administration
date: 2019-07-17 18:02:52
tags: 
  - aws 
  - Codecommit
  - CICD
---
# CodeCommit
Codecommit is a source control service provided by aws(hosts private git repositories)
**why choose CodeCommit**
- Easy integration with other AWS services like CodePipeline
- Repositories are private by default
- Can use IAM for fine-graned authorization

**Creating a private code repo on CodeCommit**
IAM-> HTTPS Git credentials for AWS CodeCommit

```
git clone https://git-codecommit.ap-southeast-2.amazonaws.com/v1/repos/SampleRepo
cd SampleRepo
wget https://s3.amazonaws.com/aws-codedeploy-us-east-1/samples/latest/SampleApp_Linux.zip
unzip SampleApp_Linux.zip
rm SampleApp_Linux.zip
git add -A
git commit -m "Initial commit"
git push
```
**Migrating your project to CodeCommit**
```
### - - mirror here means that we are not interested in cloning the application, however we are interested in downloading the Git binary files that make up the repository in the first place. 
git clone --mirror https://github.com/awslabs/aws-demo-php-simple-app.git aws-codecommit-demo
git push https://git-codecommit.ap-southeast-2.amazonaws.com/v1/repos/myrepo
```


```
git init
git add .
git commit -m "initial commit to demp rep"
git remote add demo https://git-codecommit.ap-southeast-2.amazonaws.com/v1/repos/demo
git remote -v
git push demo master
```
[CodeCommit tutorials](https://docs.aws.amazon.com/codecommit/latest/userguide/getting-started-cc.html)
[Using IAM policies with CodeCommit](https://docs.aws.amazon.com/codecommit/latest/userguide/auth-and-access-control-iam-identity-based-access-control.html)
[Using AWS CodeCommit Pull Requests](https://aws.amazon.com/blogs/devops/using-aws-codecommit-pull-requests-to-request-code-reviews-and-discuss-code/)
[GitFlow development release model](https://datasift.github.io/gitflow/IntroducingGitFlow.html)

# Deploying Jenkins 
Amazon linux 2 AMI
```
sudo yum -y update
sudo yum -y install java-1.8.0
# remove java 7
sudo yum remove java-1.7.0-openjdk
java -version
sudo wget -O /etc/yum.repos.d/jenkins.repo https:/pkg.jenkins-ci.org/redhat/jenkins.repo 
sudo rpm --import https://pkg.jenkins.io/redhat/jenkins.io.key
sudo yum -y install jenkins
sudo service jenkins start
```

# CodeDeploy
![codedeploy ec2](https://i.imgur.com/ZRCFrZP.png)

# Cloud9
**What is Cloud9**
- Cloud9 integrates with other AWS DevOps tools such as CodeCommit, CodePipeline, and CodeStar to enable a rich development pipeline with continuous delivery
- AWS Cloud9 contains a collection of tools that you use to code, build, run, test, debug, and release software on the cloud
- Use the AWS Cloud9 Integrated Development Environment (IDE) to work with these tools
**What can i do with aws Cloud 9**
- Supported languages:
  - C++
  - Java
  - Python
  - .NET
  - Node.js
  - PHP
  - Pearl
  - Ruby
  - Go
  - JavaScript
  - CoffeeScript
- Supported integrations:
  - CodeCommit
  - CodePipeline
  - CodeStar
  - API gateway
  - Lambda
  - Lightsail
  - DynamoDB
  - RDS
  - AWS CLI
  - Docker
  - GitHub
- Supported environments
  - EC2
  - SSH
  - Single-user environment
  - Shared team environment
  - virtualenv
# CodeBuild
# CodePipeline
# CodeStart

# Resources
- [What is DevOps](https://aws.amazon.com/devops/what-is-devops/)
- [AWS pricing](https://aws.amazon.com/pricing/)
- [TUtorial-create a simple pipeline](https://docs.aws.amazon.com/codepiepline/latest/userguide/tutorials-simple-codecommit.html)
