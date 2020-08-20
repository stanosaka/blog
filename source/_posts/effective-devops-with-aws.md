---
title: effective devops with aws
date: 2019-08-16 00:15:29
tags:
---
**Code**

[source code ansible](https://github.com/stanosaka/ansible)
[source code helloworld](https://github.com/00root/helloworld)

# Continuous Delivery
![codepipline01](https://i.imgur.com/QOYWSYb.png) 
![codepipline02](https://i.imgur.com/an0PPjD.png)
![Source](https://i.imgur.com/P7gdZBo.png)
![Test](https://i.imgur.com/O6LNT3M.png)
![Deploy](https://i.imgur.com/gD0EYZ7.png)
![Approval](https://i.imgur.com/uAnMDQk.png)
![Production](https://i.imgur.com/OoeILa2.png)
![Jenkins01](https://i.imgur.com/Mz3RM0I.png)
![Jenkins02](https://i.imgur.com/88C8zGu.png)
![Jenkins03](https://i.imgur.com/7YXnt5L.png)
```
npm config set registry http://registry.npmjs.org/ 
npm install
./node_modules/mocha/bin/mocha    
```
**Creating the new cloudformation stack for production**
```
aws cloudformation create-stack --capabilities CAPABILITY_IAM --stack-name helloworld-production --template-body file://nodeserver-cf.template --parameters ParameterKey=KeyPair,ParameterValue=EffectiveDevOpsAWS
aws cloudformation wait stack-create-complete --stack-name helloworld-production
arn=$(aws deploy get-deployment-group --application-name helloworld --deployment-group-name staging --query 'deploymentGroupInfo.serviceRoleArn')
aws deploy create-deployment-group --application-name helloworld --ec2-tag-filters Key=aws:cloudformation:stack-name,Type=KEY_AND_VALUE,Value=helloworld-production --deployment-group-name production --service-role-arn $arn
aws deploy create-deployment-group --application-name helloworld --ec2-tag-filters Key=aws:cloudformation:stack-name,Type=KEY_AND_VALUE,Value=helloworld-production --deployment-group-name production --service-role-arn arn:aws:iam::012152306932:role/CodeDeployRole
aws deploy list-deployment-groups --application-name helloworld
aws sns create-topic --name production-deploy-approval
aws sns subscribe --topic-arn arn:aws:sns:ap-southeast-2:012152306932:production-deploy-approval --protocol email --notification-endpoint foobar@gmail.com
```
