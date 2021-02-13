---
title: devops-with-aws
tags: [Jenkins, Devops, CD/CI]
date: 2019-07-15 23:44:53
---
# Project setup
[code](https://github.com/espiderinc/hapi-rest-demo)
## Importance of automated test in CI,CD
- Automated tests
    - Unit tests
    - Integration tests
    - UAT tests
- Code coverage
- Notifications
## CI/CD with relational databases
- Managing the version of database schema
There is no easy way to control the version of relational database schema
- Database schema migrations
- DellStore2 sample database
   - Products table
- Sqitch change management system
## Project component setup
- PostgreSQL database on AWS RDS
- Node.JS HAPI RESTful API project
- Sqitch database mangement framework
## Setup PostreSQL database instance in AWS RDS
1. Create a rds in aws with postgresql engine version 9.4.7.
2. Connect to aws rds use pgAdmin 3.
3. Download sample schema dellstore2 from [link](http://pgfoundry.org/projects/dbsamples/)
4. In pgAdmin3 click Plugins-> PSQL console-> run command `\i /tmp/dellstore2.sql` to create a new schema.
## Setup Node.JS HAPI ReSTful API project
**HAPI** is a rich application framework for building applications and RESTful APIs with Node.JS

Official website for HAPI framework is HAPIJS.com
1. install node and npm
```
node -v
npm -v
mkdir myfirsthapiproject
cd myfirsthapiproject
npm init
npm install --save hapi
```
2.
```
git clone https://github.com/espiderinc/hapi-rest-demo.git
cd hapi-rest-demo
npm install
sudo npm install -g istanbul mocha
node index.js
```
3. ![access by](https://i.imgur.com/rCcEgzW.png))
![Imgur](https://i.imgur.com/VdGe8f3.png)

4. test
```
hapi-rest-demo git:(master) ✗ npm test

> hapi-rest-demo@1.0.0 test /home/stan/workspace/hapi-rest-demo
> istanbul cover _mocha test/**/*.js



(node:20777) [DEP0022] DeprecationWarning: os.tmpDir() is deprecated. Use os.tmpdir() instead.
  Task routes
    GET /products
      ✓ should return statusCode 200 (381ms)
      ✓ should return product [ACADEMY BROOKLYN] 


  2 passing (416ms)

=============================================================================
Writing coverage object [/home/stan/workspace/hapi-rest-demo/coverage/coverage.json]
Writing coverage reports at [/home/stan/workspace/hapi-rest-demo/coverage]
=============================================================================

=============================== Coverage summary ===============================
Statements   : 56.31% ( 58/103 )
Branches     : 39.29% ( 11/28 )
Functions    : 47.83% ( 11/23 )
Lines        : 57% ( 57/100 )
================================================================================
```
report generated workspace/hapi-rest-demo/coverage/lcov-report/index.html

## Setup swtich (database schema framework)
Managin database schema for relational databaes (with Sqitch)
**Sqitch** is a standalone system without any dependency on frameworks or ORMs.
- handels dependencies between scripts
- [project site](http://sqitch.org)
**install sqitch**
```
docker pull sqitch/sqitch
curl -L https://git.io/fAX6Z -o sqitch && chmod +x sqitch
mv sqitch /usr/bin/
sudo apt-get install -y libdbd-pg-perl postgresql-client
sqitch --version
```
**use sqitch**
```
mkdir stantutorial
cd stantutorial
sqitch init stantutorial --uri http://github.com/espiderinc/hapi-rest-demo.git
cat sqitch.conf
 [core]
         engine = pg
        # plan_file = sqitch.plan
        # top_dir = .
sqitch config --user user.name 'StanTutorial'
sqitch config --user user.email 'devops@cwzhou.win'
sqitch add schema -n 'Add schema for tutorial objects.'
Created deploy/schema.sql
Created revert/schema.sql
Created verify/schema.sql
Added "schema" to sqitch.plan

 cat deploy/schema.sql
-- Deploy stantutorial:schema to pg

BEGIN;

-- XXX Add DDLs here.
CREATE SCHEMA tutorial;

COMMIT;

cat revert/schema.sql
-- Revert stantutorial:schema from pg

BEGIN;

-- XXX Add DDLs here.
DROP SCHEMA tutorial;

COMMIT;

 cat verify/schema.sql
-- Verify stantutorial:schema on pg

BEGIN;

-- XXX Add verifications here.

select 1/count(*) from information_schema.schemata where schema_name='tutorial';

ROLLBACK;

 sqitch add video --requires schema -n 'add video table to schema tutorial'

 cat deploy/video.sql
-- Deploy stantutorial:video to pg
-- requires: schema

BEGIN;

-- XXX Add DDLs here.
SET client_min_messages = 'warning';
CREATE TABLE tutorial.video(
        subject TEXT    PRIMARY KEY,
        comment TEXT,
        timestamp TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

COMMIT;

cat revert/video.sql
-- Revert stantutorial:video from pg

BEGIN;

-- XXX Add DDLs here.
DROP TABLE tutorial.video;

COMMIT;

cat verify/video.sql
-- Verify stantutorial:video on pg

BEGIN;

-- XXX Add verifications here.
select subject, comment, timestamp
from tutorial.video
where false;

ROLLBACK;
sqitch deploy db:pg://foo:bar@stantest-postgresql.c3mzoji03zxf.ap-southeast-2.rds.amazonaws.com:5432/postgres
sqitch status db:pg://foo:bar@stantest-postgresql.c3mzoji03zxf.ap-southeast-2.rds.amazonaws.com:5432/postgres
sqitch verify db:pg://foo:bar@stantest-postgresql.c3mzoji03zxf.ap-southeast-2.rds.amazonaws.com:5432/postgres
sqitch revert db:pg://foo:bar@stantest-postgresql.c3mzoji03zxf.ap-southeast-2.rds.amazonaws.com:5432/postgres
sqitch status db:pg://foo:bar@stantest-postgresql.c3mzoji03zxf.ap-southeast-2.rds.amazonaws.com:5432/postgres
```
# CI and CD pipeline deep dive
## AWS prerequisites
- IAM instance profile
1. Create a policy, name: CodeDeploy-EC2-Permissions, json
```
{
    "Version": "2012-10-17",
    "Statement": [
     {
        "Action":[
           "s3:Get*",
           "s3:List*"  
        ],
        "Effect": "Allow",
        "Resource": "*"
     }    
        
        
    ]
}
```
2. Create a role, named CodeDeploy-EC2 -> Choose role type ec2->Attach permissions policies "CodeDeploy-EC2-Permissions"
- IAM service role
1. Create a role named stantutorialRole -> select role type CodeDeploy

## Jenkins installation
Ubuntu-> Configure Instance Details, IAM role, select CodeDeploy-EC2 (this will allow jenkins connect to s3 buckets)->
Tag instance: Key group Value hapi-demo
```
wget -q -O - http://pkg.jenkins-ci.org/debian/jenkins-ci.org.key| sudo apt-key add -
sudo vim /etc/apt/sources.list # add following line
deb http://pkg.jenkins-ci.org/debian-stable binary/
sudo apt-get install default-jdk
sudo apt-get install jenkins
sudo service jenkins start
```
Install plugin AWS CodePipeline
Install node.js:
```
curl -sL https://deb.nodesource.com/setup_10.x | sudo -E bash -
apt-get install -y nodejs
sudo npm install -g npm
node -v
```
Install sqitch
```
sudo apt-get install build-essential cpanminus perl perl-doc
cpanm --quiet --notest App::Sqitch
sqitch --version
apt-get install -y postgresql libdbd-pg-perl
```
Create a new instance hapi-demo install node.js and
```
sudo apt install python3-pip
sudo apt install ruby-full
sudo apt install wget
wget https://aws-codedeploy-ap-southeast-2.s3.amazonaws.com/latest/install
chmod +x install
sudo ./install auto
```
## CodeDeploy application
Create a new Codedeploy application, choose compute type ec2/on-premises, Service role-> statutorialRole; Environment configuration tick Amazon EC2 instances, Key-> Name, Value-> hapi-demo; Deployment setting-> CodeDeployDefault.OneAtATime

## Review appSpec.yml file
appspec.yml file is an application specification file for aws codedeploy

```
cat appspec.yml
version: 0.0
os: linux
files:
  - source: /
    destination: /myapp
permissions:
  - object: /myapp/startApp.sh
    mode: 777
hooks:
  ApplicationStart:
    - location: startApp.sh
      timeout: 10
```
## Setup Jenkins job
Create a freestyle jenkins job, configurat as following screenshots:
![screenshot01](https://i.imgur.com/yb9AZch.png)
![screenshot02](https://i.imgur.com/jxkg9xA.png)
![screenshot03](https://i.imgur.com/ghjvXwd.png)

## Build AWS CodePieline
1. Source provider [GitHub](https://github.com/stanosaka/hapi-rest-demo)
2. Build provider: Add Jenkins; Prvider name must match the name in jenkin's job
3. Deployment provider: aws codedeploy
![aws codedeploy](https://i.imgur.com/Zcps1fo.png)
![deployed pages](https://i.imgur.com/e2nGdbj.png)
![api](https://i.imgur.com/eJAhUJH.png)
![lcov-report](https://i.imgur.com/MJs6nKN.png)

# Next Steps
## Notifications
AWS SNS notifications for build and deployment status
1. Create a policy named: notification-policy
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "sns:Publish",
            "Resource": "*"
        }
    ]
}
```
Attach notification-policy to Role CodeDeploy-EC2
2. in CodeDeploy edit deployment group;
![create deployment trigger](https://i.imgur.com/Lx8Kxdw.png)
![notification](https://i.imgur.com/Xl4OvF8.png)

## Code changes
Automatically and continuously deploy code without any downtime

## Database schema changes
Consistently and automatically deploy relational database schema changes
```
sqitch add product-add-comments
cat /db/deploy/product-add-comments.sql
-- Deploy spidertutorial:product-add-comments to pg

BEGIN;

-- XXX Add DDLs here.
alter table products add comments varchar(100) default 'default comments';

COMMIT;
git commit -a -m "add new column to products"
git push origin master:master
```
