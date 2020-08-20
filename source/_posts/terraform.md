---
title: terraform
date: 2019-07-13 13:29:54
tags: 
- terraform 
- aws 
- automation
---
[Supplemental content](https://github.com/PacktPublishing/Hands-on-Infrastructure-Automation-with-Terraform-on-AWS)
# Install Terraform and Tools on linux
## Terraform development environment
- Terraform
- aws account
- aws cli with credentials configured
- git
- shell (bash, pwoershell, cmd, git-bash)
- Text editor (visual studio code with Extensions terraform)
```
wget https://releases.hashicorp.com/terraform/0.12.4/terraform_0.12.4_linux_amd64.zip
unzip terraform_0.12.4_linux_amd64.zip
sudo mv terraform /usr/local/bin
terraform version
```
# First deployment with Terraform
## Configuration language basics
Terraform uses HCL (Hashicorp configuration language)
- human friendliness
- JSON is quite verbose and doesn't support comments, is machine readable
- YAML is quite easy to mess up indentation and it's not always clear whether you should use a colon or hyphen(specially when you using nested maps and lists)- machine-friendly, which is designed to be written and modified by humans
- can sue JSON as input to Terraform

**main features of HCL**
- can have single-line comments, start with a double slash or a number sign
- multi-line comments are wrapped in /*
- Values assign syntax key = vaule
- Strings are double bolded
- Numbers, booleans, arrays, lists, objects, named maps or dictonaries
- Interpolations, conditionals, and various build-in functions

## Set up aws provider
```
mkdir s3_backbone && cd s3_backbone
git init
terraform init
```
Provide: Terraform object that is responsible for managing the lifecycle of a resouce:
- Create, REad, Update, and Delete operations (CRUD)
```
cat providers.tf
provider "aws"{
  region = "ap-southeast-2"
}
git add -A
git commit -m "Add aws provider"
terraform init
git status
echo ".terraform" >> .gitignore
git status
git add .gitignore&& git commit -m "Add .gitignore"
terraform plan # creates an execution plan
```
## Deploy an s3 bucket into aws
google terraform s3 bucket
```
cat s3.tf
resource "aws_s3_bucket" "main" {
  bucket = "packt-terraform-section2-bucket-stan"
  acl    = "private"
}
git add . && git commit -m "Add S3 bucket"
terraform plan
# The output of this command is similar to what we get when we run the diff command on Linux: resources with a plus sign are going to be created, resources with the minus sign are going to be deleted, and resources with a tilde sign are going to be modified
terraform apply
git status
# notice that Terraform also created a new file, terraform.tfstate
# it's a JSON file, which contains some information about the bucket we just created. Terraform uses the state file to map real-world resources to your configuration, and keep track of metadata
```
**What is state?**
- Desired state
- Actual state
- Known state
When we write our configuration files, we describe the desired state. This is how we want our infrastructure to be. Then there's the actual state: this is how our infrastructure looks like, right now. You can get this actual state by exploring your infrastructure in the web console, or running some describe commands against the API. And to bridge these two states, there is the known state, which is stored in the state file. Terraform uses it to keep track of all resources it already created for this set of templates. In general, this known state should be the same as the actual state. When we run the plan command, Terraform performs a refresh, and then determines what actions are necessary to achieve the desired state specified in the configuration files. When you run the apply command, Terraform executes the planned actions, and then stores the updated actual state in the state file.

for example, you went to the web console and manually changed something - Terraform will detect such changes, and unless they also exist in the desired state, it will revert them. So, if we are going to treat our **infrastructure as code**, we should get into the mindset of not changing our sources manually.

Make sure the state file is ignored by Git
```
 cat .gitignore
.terraform

*.tfstate*
git add -A && git commit -m "Ignore TF state"
```
## Structuring the project
Typical project structure
```
|-main.tf
|-outputs.tf
|-variables.tf
```
group related resources together, and keep the configuration files to a manageable size - ideally, not longer than 200 lines of code.

# Modifying resoureces
## Variables
Keep the code DRY- Don't Repeat Yourself-SOFTWARE ENGINEERING PRICIPLE aimed at reducing interpretation of the same data

Terraform performs automatic conversion from string values to numeric and boolean values, based on context.

Maps are useful for selecting a value based on some other provided value.

A list value is an ordered sequence of strings, indexed by integers starting with 0.

Several ways to set variables:
1. 
```
cat variables.tf
variable "s3_bucket_name" {
        #default = "packt-terraform-section2-bucket-stan"
        description = "Name of the S3 bucket"
        type = "string"
}

variable "s3_tags" {
        type = "map"

        default = {
                created_by = "terraform"
                environment = "test"
        }
}

variable "s3_regions" {
        type = "list"
        default = ["ap-southeast-2", "us-west-2"]
}

terraform plan
var.s3_bucket_name
  Name of the S3 bucket

  Enter a value: packt-terraform-section2-bucket-stan

Refreshing Terraform state in-memory prior to plan...
The refreshed state will be used to calculate this plan, but will not be
persisted to local or remote state storage.

aws_s3_bucket.main: Refreshing state... [id=packt-terraform-section2-bucket-stan]

------------------------------------------------------------------------
```
2. 
```
terraform plan -var 's3_bucket_name="packt-terraform-section2-bucket-stan"'
Refreshing Terraform state in-memory prior to plan...
The refreshed state will be used to calculate this plan, but will not be
persisted to local or remote state storage.

aws_s3_bucket.main: Refreshing state... [id=packt-terraform-section2-bucket-stan]

------------------------------------------------------------------------
```
3.
```
TF_VAR_s3_bucket_name="packt-terraform-section2-bucket-stan" terraform plan
Refreshing Terraform state in-memory prior to plan...
The refreshed state will be used to calculate this plan, but will not be
persisted to local or remote state storage.

aws_s3_bucket.main: Refreshing state... [id=packt-terraform-section2-bucket-stan]

------------------------------------------------------------------------
```
**Variable definition files**
-  We can also have multiple tfvars files, and pass them explicitly to Terraform using the var file flag. If we pass several files, Terraform will merge their values - and if a particular variable is defined in more than one variable file, the last value that is filed wins. 
- Terraform automatically loads all files which match terraform.tfvars or *.auto.tfvars from the current directory
- Other files can be passed explicitly using -var-file flag
```
cat terraform.tfvars
s3_bucket_name = "packt-terraform-section2-bucket-stan"
terraform plan
Refreshing Terraform state in-memory prior to plan...
The refreshed state will be used to calculate this plan, but will not be
persisted to local or remote state storage.

aws_s3_bucket.main: Refreshing state... [id=packt-terraform-section2-bucket-stan]

------------------------------------------------------------------------
```
**String Interpolation**
- The process of evaluating a string expression and replacing all paceholders with their values
`"${var.s3_bucket_name}"`
**First-Class Expressions**
`var.s3_bucket_name`
```
 cat s3.tf
resource "aws_s3_bucket" "main" {
  bucket = "${var.s3_bucket_name}"
  acl    = "private"

  tags   = {
    env = "${lookup(var.s3_tags, "environment")}"
  }

  region ="${var.s3_regions[0]}"
}
#Terraform console is a useful tool for automation scripts, as it allows you to access arbitrary attributes from a Terraform configuration.
terraform console
> var.s3_tags
{
  "created_by" = "terraform"
  "environment" = "test"
}
> var.s3_tags["environment"]
test
> exit
```
## Local development workflow
**Using git to store state is a bad idea**
- maintenance overhead
- secrets in plain text

**State**
- Local state
- Version Control
- Remote state
- Backends
   - Terraform enterprise
   - S3
   - Consul: a service networking solution to connect and secure services across any runtime platform and public or private cloud.
   - Etcd
   - HTTP

Recommend using S3
**Local Values**
- **Input variables** are similar to arguments of a function
- **Local values** are analogous to local variables within the function's scope

edit variables.tf file:
```
locals {
        s3_tags= {
                created_by = "terraform"
                environment = "${var.environment}"
        }
}
```

we can skip refreshing it again to save a couple of seconds, here. I will pass a flag, -refresh=false.
The state filei(terraform.tfstate) is also used to store some metadata, such as resource dependencies, or a pointer to the provider configuration in situations where multiple AWS providers are present. Another use is to store a cache of the attribute values for all the resources in the state. When we run terraform plan, Terraform must know the current state of resources, and by default it will query the providers and sync the latest attributes from all our resources. For large infrastructure, this can be too slow, and we may get throttled at the API level - so, as a performance improvement, there is an option to disable this behavior by passing refresh=false flag.
```
terraform plan -refresh=false

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
-/+ destroy and then create replacement

Terraform will perform the following actions:

terraform apply -auto-approve  ##skip answer yes

export TF_CLI_ARGS_apply="-auto-approve"
```
**Common Workflow**
- validate
- plan
- apply
- fmt
[Pre-Commit Hooks](https://github.com/antonbabenko/pre-commit-terraform)
## Deleting Resources
1.
```
terraform plan -destroy
terraform destroy
```
2. remove a resource from configuration and then run terraform plan && terraform apply

the 2nd one is more suted to CI/CD systems. You will most likely have some pipeline for provisioning your resources, but you may not necessarily have any automated way of destroying them, because this doesn't happen that often. Another point is that, this way, you can destroy specific resources without having to pass special flags, which would require changes to automation scripts.

**Protect a resoure from deletion**
Use the life cycle meta-parameter.
```
lifecycle {
  prevent_destory = "true"
}
```
Only protects against terraform destory.
## Managing state ********************************************
[code](https://github.com/PacktPublishing/Hands-on-Infrastructure-Automation-with-Terraform-on-AWS/tree/master/Section_3)

**set up the remote state backend**
1. Create the Terraform configuration section

This block is special, as it configures the behavior of Terraform itself, such as setting up a backend or requiring a minimum Terraform version to execute a configuration.
```
terraform {
  required_version = "> 0.11.7"
}
```
if we have some configuration which we can't migrate to the latest version, for some reason, we can pin it to an older version in this block. Let's set the current version.

**Manage terraform versions for each project by Terraform switcher**
```
# MacOS with brew
brew install warrensbox/tap/tfswitch
# Linux
curl -L https://raw.githubusercontent.com/warrensbox/terraform-switcher/release/install.sh | bash
tfswitch
```
2. Add a backend
```
backend "s3" {
    bucket = "packt-terraform-bucket-stan-test-ap-southeast-2"
    key = "test/backbone"
    region = "ap-southeast-2"
    encrypt = "true"
  }
```
3. Apply
```
git add .
git commit -m "Configure remote state"
git clean -fdx #clean the repository of all unchecked files
Removing .terraform/
Removing terraform.tfstate
Removing terraform.tfstate.backup
terraform init
terraform plan #there is no more local state file anymore

```
4. configuration todo
- enforce encryption by default (here use the default one, you can use KMS instead)
by edit s3.tf:
```
  server_side_encryption_configuration {
    rule {
      apply_server_side_encryption_by_default {
         sse_algorithm = "AES256"
      }
    }
  }
```
- versioning: alwasy have a way back
```
  versioning {
     enabled = true
  }
```
- lifecycle policy: Versioning will store all previous versions of the file, which will eventually bloat the bucket size. To reverse this, we can set a lifecycle policy. Remove all the version after 90 days. In your particular case, you might want to use a different value, or maybe move them to **glacier storage** instead of deleting the old files.
```
  lifecycle_rule {
     id      = "state"
     prefix  = "state/"
     enabled = true

     noncurrent_version_expiration {
        days = 90
     }
  }
```

**Best Practices**
- Do not store state locally
- Do not commit state to version control
- If using S3:
    - Enable versioning
    - Enforce encryption
    - Store state close to the infrastructure
    - Limit access to the bucket, consider enabling log in

# Building a multi-tier environment
**What we will build?**
- Network layer-Virtual Private Cloud (VPC), internet gateway, public and private subnets, NAT gateway, and a bastion host)
- Relational Database Service (RDS) instance running PostgreSQL
- Elastic Container Service (ECS) cluster to host a dockerised app
**Provider Caching**
- **terraform init** downloads providers separately for each project
- We can cache them by setting an environment variable
    export TF_PLUGIN_VACHE_DIR="$HOME/.terraform.d/plugin-cache"
[code](https://github.com/PacktPublishing/Hands-on-Infrastructure-Automation-with-Terraform-on-AWS/tree/master/Section_4/vpc)
![AWS VPC](https://i.imgur.com/Rzhlc11.png)
**Count Meta-Parameter**
- Available to all resources
- Allows creating multiple copies of a resouce without repeating its configuration
- Helps keep your infrastructure code **DRY**
**Splat Expression**
```
resource "aws_nat_gateway" "main" {
  count         = "${length(var.availability_zones)}"
  subnet_id     = "${element(aws_subnet.public.*.id, count.index)}"
  allocation_id = "${element(aws_eip.nat.*.id, count.index)}"
}
```
The challenge, here, is that the gateways need to reference the subnets and the IPS that we created, but because we used count, we don't have a direct reference to each resource. We can resolve this by using a splat expression, which you can see in action where we reference the subnet ID and the allocation ID. A splat expression allows to obtain a list of attribute values from a set of resources created using the count argument. Notice this asterisk, which represents all values in the generated list.
## Organizing data with output variables
**Resources and data sources**
- Resources provide **Create, Read, Update**,and **Delete** functionality (CRUD)
- Data srouces support **read** operations only
```
data "aws_ami" "amazon_linux" {
  most_recent = true
  filter {
    name   = "name"
    values = ["amzn-ami-*-x86_64-gp2"]
  }
}
```
**Output Variables**
- Expose important resource attributes and make tme easier to query
- Outputs are exposed to the user and stored in the state file during terraform apply
- A single output block configures a single variable
**Example Output**
```
output "vpc_id" {
   value = "${aws_vpc.main.id}"
   description = "VPC id"
   sensitive = false
}
```
After we run the apply once, the outputs are stored in the state file - so, the next time we need to get them, we can use `terraform output`.
or
```
terraform output public_subnets
```
**provider version**
```
 terraform providers
.
├── provider.aws 1.31
└── provider.template
ls -l ~/.terraform.d/plugin-cache/linux_amd64
```
## Integrating components in a complex environment
![Adding a database server](https://i.imgur.com/LcsUUkG.png)
**Steps to add a DB server**
1. Create a VPC (done)
2. Add Subnets to the VPC (done)
3. Create a DB Subnet Group
4. Create a VPC Secruity Group
5. Create a DB Instance in the VPC

Integrate separate Terraform configurations while keeping them in different projects

Remote state serves as a centralized source of truth:
```
 cat data_sources.tf
# Remote state
data "terraform_remote_state" "vpc" {
  backend = "s3"

  config {
    bucket = "packt-terraform-bucket-stan-test-${var.region}"
    key    = "test/vpc"
    region = "${var.region}"
  }
}
```
Remote state allows us to share information between current projects, and to build our infrastructures in small atomic configurations focused on one thing.
```
db_subnet_group.tf
subnet_ids = ["${data.terraform_remote_state.vpc.private_subnets}"]
```
Notice the interpolation syntax that we use.first, specify that it's a data source - then its type, terraform remote state. Then comes the name of particular remote state, VPC, and lastly the attribute that we are interested in. 

## Using templates
![Application tier](https://i.imgur.com/IjCKZfO.png)
deploy a small but realistic web application into our VPC. Our app runs in Docker, so we will provision an LST container service cluster - **ECS** - to host it in our private subnets. We will use **Fargate**, which is a managed compute and orchestration engine, so we won't need to maintain EC2 instances that run our containers. 

**Use Terraform templates to compose complex string inputs**
App: a REST API for a todo applicaton, written in Go. It uses Postgres scale database as its backend.

[public image on Docker hub](https://hub.docker.com/r/endofcake/go-todo-rest-api-example/)

1. provision an ECS cluster
```
cat ecs_cluster.tf
resource "aws_ecs_cluster" "main" {
  name = "${var.ecs_cluster_name}"
}
```

**Partitioning Infrastructure**
- If the resources are likely to be created and destryoed together, they belong together
- If some resource can be used by multiple other resources, it's better to keep it sparate (VPC,RDS, and ECS cluster)

template eg.
```
bastion.tf
# User data
data "template_file" "user_data" {
  template = "${file("${path.module}/templates/user_data.sh")}"
}

/tmplates/user_data.sh
#!/bin/env bash

set -euo pipefail
exec > >(tee /var/log/user-data.log|logger -t user-data -s 2>/dev/console) 2>&1

echo "Starting user data..."
yum update -y
yum install -y postgresql96.x86_64
touch /home/ec2-user/success
echo "All done"
```
Next, I create template_cloudinit_config data source, and pull in the rendered template file. cloudinit_config allows us to compose multi-part user data scripts.
```
data "template_cloudinit_config" "user_data" {
  gzip          = true
  base64_encode = true

  part {
    content_type = "text/x-shellscript"
    content      = "${data.template_file.user_data.rendered}"
  }
}
```
If we run terraform apply with this configuration on Windows, it most likely wouldn't work - the problem is that Windows use a different line break tag, which is not valid on Linux. Windows use both carriage return and line feed, while Linux only uses line feed. The easiest way to check the line break tag is to look at the bottom right corner of the editor. Anyway, long story short, we want to make sure that this script is valid, even if we deploy our configuration from a Windows machine. This is probably the only case where there is any difference between running Terraform on Linux and on Windows. There are two changes that you should make to resolve this.

1. add a gitattributes file
```
cat .gitattributes
# Always LF
*.sh text eol=lf
```
2. use EditorConfig plugin to define how our text editor displays and saves our code
![editorconfig plugin](https://i.imgur.com/55JD2B3.png) 
```
cat .editorconfig
[*.sh]
end_of_line = lf
indent_size = 2
```
[ecs app todo code](https://github.com/stanosaka/Hands-on-Infrastructure-Automation-with-Terraform-on-AWS/tree/master/Section_4/ecs_app_todo)
```
~/terraform/ecs_app_todo|
⇒  tree
.
├── cloudwatch.tf
├── data_sources.tf
├── ecs_task.tf
├── graph.png
├── iam.tf
├── lb.tf
├── outputs.tf
├── providers.tf
├── templates
│   └── ecs_task.tpl
├── terraform.tfvars
└── variables.tf
```
templates/ecs_task.tpl file: ECS task definition. It describes which Docker images to use, the required resources, and other configurations necessary to launch the containers. As you can see, it's a JSON block, which I've extracted into a template. It requires a few parameters, mostly to set up the connection to the database. 

ecs_task.tf file: This is the Terraform configuration of our ECS task. I'm using the familiar template file data source, and I'm passing the required variables using the vars parameter, which accepts a map of variables. Some of the variables come from the tfvars file, but many are imported from the remote state. 

data_sources.tf file: If we check out our data sources, we see that we're pulling in remote state from all three projects that we created earlier. 

lb.tf file: another important resource that we are creating is the load balancer, which distributes income and application traffic across multiple targets, for high availability. It will also allow us to connect to our service from the public internet, and here is the security group which allows public access

**confirm it's working**
```
curl -d '{"title":"sample project"}' -H "Content-Type: application/json" -X POST todoapp-alb-91218331.ap-southeast-2.elb.amazonaws.com/projects
{"ID":1,"CreatedAt":"2019-07-21T12:20:38.881103057Z","UpdatedAt":"2019-07-21T12:20:38.881103057Z","DeletedAt":null,"title":"sample project","archived":false,"tasks":null}

curl todoapp-alb-91218331.ap-southeast-2.elb.amazonaws.com/projects
[{"ID":1,"CreatedAt":"2019-07-21T12:20:38.881103Z","UpdatedAt":"2019-07-21T12:20:38.881103Z","DeletedAt":null,"title":"sample project","archived":false,"tasks":null}]

ssh bastion
export PGPASSWORD=foobar
export PGHOST=todoapp.foobar.ap-southeast-2.rds.amazonaws.com
psql -U terraform -d todoapp
todoapp=> \d+
                             List of relations
 Schema |      Name       |   Type   |   Owner   |    Size    | Description
--------+-----------------+----------+-----------+------------+-------------
 public | projects        | table    | terraform | 16 kB      |
 public | projects_id_seq | sequence | terraform | 8192 bytes |
 public | tasks           | table    | terraform | 8192 bytes |
 public | tasks_id_seq    | sequence | terraform | 8192 bytes |
(4 rows)
todoapp=> select * from projects;
 id |          created_at           |          updated_at           | deleted_at |     title      | archived
----+-------------------------------+-------------------------------+------------+----------------+----------
  1 | 2019-07-21 12:20:38.881103+00 | 2019-07-21 12:20:38.881103+00 |            | sample project | f
(1 row)
```


## Working with dependency graph
**Dependency graph**
- All resources in the configuration are organized into a graph
- The graph is used to determine the order in which the resources are created
![Directed acyclic graph](https://i.imgur.com/9swFk8u.png)
Terraform organizes all resources in a configuration in a directed acyclic graph. What this means in plain English is that the dependencies between the resources go in one direction, and there can be no cycles. So, no circular dependencies. When we run the plan command, Terraform builds this graph, checks whether there are any cycles, and then determines which operations can be run in parallel. By default, up to ten nodes in the graph can be processed concurrently. We can control this setting using the parallelism flag on the plan, apply, and destroy commands, but in most cases this is not required.
**Parallelism**
- Up to 10 nodes can be processed concurrently by default
- Conifgurable with **-parallemism** flag for plan, apply, and destroy commands (advanced setting)
**Dependencies**
- Implicit-one resource references another resource using the interpolation syntax
```
resource "aws_lb" "todo_app" {
    security_groups = ["${aws_security_group.lb.id}"]
}
```
- Explicit-using depends_on metaparameter
   `depends_on = ["aws_security_group.lb"]`

Use explicit dependencies to resolve race conditions
```
# Have to set an explicit dependency here to avoid
  # race condition with LB creation
depends_on = ["aws_lb_listener.todo_app"]
```
**tools**
1. [Graphviz](http://graphviz.gitlab.io/download/)
```
terraform graph|dot -Tpng > graph.png
terraform plan
terraform graph -draw-cycles|dot -Tpng > graph.png
```
2. [blast radius](https://github.com/28mm/blast-radius)

## Main takeaways
- Use outputs and remote state data source to integrate stacks of resources in a complex environment
- Keep your code DRY by using *count* parameter and splat expressions
- Use **templates** to generate complex string inputs, such as user data scripts or ECS task definitions

# Creating reusable components with moduels
## Modules
- Self-contained packages of Terraform configurations that are managed as a group
When we want to avoid writing duplicate code in a general-purpose programming language, we usually write a library. In, Terraform, we can put our code in a **module**.
- Improve code resue
- Provide an abstration layer
  for example, you may need to add a vault cluster to your environment, and the vault is another great Hashicorp tool which is used for managing secrets, which requires dozens of components - but instead of thinking about individual security groups or EC2 instances, you can treat all these resources as a single group which requires some parameters, and gives you a ready-to-use vault cluster. 
- Can be teated as blackbox
- Share best practices within an organization
- Versioned artifacts

## Creating the first module
- Root module:
   - The current working dirctory holding Terraform files
- Child modules:
   - All modules sourced by the root (parent) module
**Delaring Modules**
```
module "child" {
source = "./child"
}
```

1. rename ecs app project to module.ecs_app_web
```
cat main.tf
module "child" {
    source = "../module.ecs_app_web"
}
```
**terraform get**
- terraform get: Download modules referenced in the root module
- terraform get -update: Check the downloaded modules for updates and download the new versions. if present
```
 terraform get -update
- module.child
  Updating source "../module.ecs_app_web"
```
When you run terraform get, the local modules will be simlinked into .terraform directory, so you can inspect what's there if you notice some unexpected behavior.

2. copy terraform.tfvars and variables.tf, modify main.tf file, add
```
region = "${var.region}" 
app_image_version = "${var.app_image_version}" 
app_image_repository = "${var.app_image_repository}" 
app_name = "${var.app_name}" 
container_port = "${var.container_port}" 
desired_count = "${var.desired_count}" 
pgsslmode = "${var.pgsslmode}" 
network_mode = "${var.network_mode}" 
requires_compatibilities = "${var.requires_compatibilities}" 
launch_type = "${var.launch_type}" 
health_check_path = "${var.health_check_path}" 
```
**terraform providers**
- terraform providers:
  - Print information about the providers used in the currrent configuration
- terraform providers [config-path]:
  - Pass an explicit path to the configuration instead of using the current working directory by default
```
 terraform providers
.
└── module.child
    ├── provider.aws 1.31
    ├── provider.template 1.0.0
    └── provider.terraform
```
There is only one module, and it uses two providers - AWS and Template, plus a special provider called Terraform, which is responsible for working with the remote state backend.

**best practices**: Keep explicit provider configurations only in the root module, and pass them down to descendant modules.

**Two wasy pass providers**
- the most common approach: let the descendant modules inherit the providers implicitly - that is, automatically
- have several providers of the same type, and then pass them explicitly by alias; this can be useful if we need to create resources in different AWS regions, for example

```
root@stan-OptiPlex-380:~stan/Doc/Terraform/ecs_app_todo|master⚡
⇒  mv ../module.ecs_app_web/providers.tf ./
root@stan-OptiPlex-380:~stan/Doc/Terraform/ecs_app_todo|master⚡
⇒  terraform init
Initializing modules...
- module.child

Initializing the backend...

Successfully configured the backend "s3"! Terraform will automatically
use this backend unless the backend configuration changes.

Initializing provider plugins...
- Checking for available provider plugins on https://releases.hashicorp.com...
- Downloading plugin for provider "aws" (1.31.0)...
- Downloading plugin for provider "template" (1.0.0)...

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
root@stan-OptiPlex-380:~stan/Doc/Terraform/ecs_app_todo|master⚡
⇒  terraform providers
.
├── provider.aws 1.31
├── provider.template 1.0.0
├── provider.terraform (from state)
└── module.child
    ├── provider.aws (inherited)
    ├── provider.template (inherited)
    └── provider.terraform
```
**Use module-relative path for embedded files (${path.module})**

**Encapsulation**
- A language mechanism for restricting direct access to some of the object's components

we can choose to make the module as transparent as possible - and in this case, it would inject all dependencies from the root module, and keep no data sources in the child module






# Error and debug
1. Error 1:
```
terraform init

Initializing the backend...
Backend configuration changed!

Terraform has detected that the configuration specified for the backend
has changed. Terraform will now check for existing state in the backends.


Error inspecting states in the "s3" backend:
    NoSuchBucket: The specified bucket does not exist
        status code: 404, request id: E51C641611FF2763, host id: 9AN52en4R7RaZueavAicV5/N01SahL+Y1TZBT8TGnYBYYD5ywWxPKgkiiqDx8+FsgkwNNyadfSU=

Prior to changing backends, Terraform inspects the source and destination
states to determine what kind of migration steps need to be taken, if any.
Terraform failed to load the states. The data in both the source and the
destination remain unmodified. Please resolve the above error and try again.
```
Debug mode:
```
 TF_LOG=trace terraform init
2019/07/21 11:50:39 [INFO] Terraform version: 0.11.7  41e50bd32a8825a84535e353c3674af8ce799161
2019/07/21 11:50:39 [INFO] Go runtime version: go1.10.1
2019/07/21 11:50:39 [INFO] CLI args: []string{"/root/.terraform.versions/terraform_0.11.7", "init"}
2019/07/21 11:50:39 [DEBUG] Attempting to open CLI config file: /root/.terraformrc
2019/07/21 11:50:39 [DEBUG] File doesn't exist, but doesn't need to. Ignoring.
2019/07/21 11:50:39 [INFO] CLI command args: []string{"init"}
2019/07/21 11:50:39 [DEBUG] command: loading backend config file: /root/terraform/vpc

Initializing the backend...
2019/07/21 11:50:39 [TRACE] Preserving existing state linea
...
```
2. Error2:
![error2](https://i.imgur.com/NvWneaV.png)
if ecs try to connect to localhost as rds, and couldnot connect sucessfully

Debug:
ecs_task.tpl pg environment vars are all missing


<!-- theme: spkane -->
<!-- _class: lead -->

<!-- $size: 16:9 -->

# Terraform
## Getting Started

<br></br>
<br></br>

###### Release %BUILD_RELEASE%

---

<!-- _class: lead -->

# Instructor
# Sean P. Kane

### @spkane

![bg right](images/skane-2018-side-black-cropped.jpg)

---

<!-- footer: Copyright © 2015-2020, Sean P. Kane (@spkane) -->
<!-- paginate: true -->
<!-- _class: lead -->

# [Follow Along Guide](https://gist.github.com/spkane/df02f7858f6bef9f045fa1c296f8146f)
## Textual Slides

---

# Prerequisites
* A recent computer and OS
  * Recent Linux, OS X, or Windows 10
  * Reliable and fast internet connectivity
* Hashicorp Terraform

---

# Prerequisites
* A graphical web browser
* A text editor
* A software package manager
* Git client
* General comfort with the command line will be helpful.
* [optional] tar, wget, curl, jq, SSH client

---

# A Note for Windows Users
This class was written from a largely Unix based perspective, but everything can be made to work in Windows with very little effort.
* Unix Variables
  * `export MY_VAR=test`
  * `echo ${MY_VAR}`
* Windows 10 Variables (powershell)
  * `$env:my_var = "test"`
  * `Get-ChildItem Env:my_var`

---

# A Note About Proxies
Proxies can interfere with some activities if they are not configured correctly.

---

# Instructor Environment
* **Operating System**: Mac OS X (v10.15.X+)
* **Terminal**: iTerm2 (Build 3.X.X+) - https://www.iterm2.com/
* **Shell Customization**: Bash-it - https://github.com/Bash-it/bash-it
* **Shell Prompt Theme**: Starship - https://starship.rs/
* **Shell Prompt Font**: Fira Code - https://github.com/tonsky/FiraCode
* **Text Editor**: Visual Studio Code (v1.X.X+) - https://code.visualstudio.com/

---

# Definition

* **ter·ra·form**
* */ˈterəˌfôrm/*
  * (verb) to alter a planet for the purpose of sustaining life

---

# Hashicorp Terraform

* Terraform is a tool that makes it possible to document and automate the creation, modification, and destruction of almost anything that can be managed by an API.
* This means that it is finally conceivable to automate the management of everything that your software stacks needs to actually run in any environment, including cloud resources, DNS entries, CDN configuration, and much more.

---

# Installing Terraform

* Download:
  * https://www.terraform.io/downloads.html
* Unarchive and copy into your executable $PATH

---

# Version 0.12

* Version 0.12 of terraform was a major release that included many significant improvements, but also included some breaking changes.
* Be aware that code written for terraform 0.12 is not compatible with earlier releases and that in general, you should not use not use older terraform binaries with existing terraform managed infrastructure.

---

# Code Setup

```shell
$ cd ${HOME}
$ mkdir class-terraform-starting
$ cd ${HOME}/class-terraform-starting
$ git clone https://github.com/spkane/todo-for-terraform \
    --config core.autocrlf=input
$ cd todo-for-terraform
```

---

# Exploring Terraform

```shell
$ cd terraform-infrastructure
```

---

# HCL & JSON

* Hashicorp Configuration Language v2
  * https://github.com/hashicorp/hcl/tree/hcl2
* HCL is a JSON-compatible configuration language written by Hashicorp to be machine and human friendly.

* HCL is intended to provide a less-verbose JSON style configuration language that supports comments, while also providing humans with a language that is easier to approach than YAML.


---

# Terraform & Backends

* Open `main.tf`
  * Terraform block
    * Define high-level requirements for this associated HCL. Terraform and provider version, etc.
  * Backend block
    * Define where remote state is stored and any information required to read and write it.

---

# Providers (1 of 2)

* Providers
  * Individual plugins that enable terraform to properly interact with an API.
  * These can range between Hashicorp's officially supported providers to custom providers written by a single developer.

---

# Providers (2 of 2)

* In this example we are using the `aws` and `ns1` providers.
  * https://github.com/terraform-providers/terraform-provider-aws
  * https://github.com/terraform-providers/terraform-provider-ns1

---

# Variables

* Open `variables.tf`
  * Defines all the variables that you will be using and their default values.
* You will get errors if you use variables that are not defined in this file.

---

# Data Sources

* Open `data.tf`
* Using output as input
  * Remote Terraform State
  * APIs
  * Scripts
    * Open `bin/local-ip.sh`
  * etc

---

# Building Infrastructure

* `key-pairs.tf`
* `backend.tf`
* `frontend.tf`
* `security-groups.tf`

---

# Backend Service

* Open `key-pairs.tf`
  * SSH public key for system access
* Open `backend.tf`
  * Server Instance w/ basic provisioning
  * Setup of `todo` backend service
* The files in `./files` support the system provisioning.

---

# Frontend Infrastructure

* Open `frontend.tf`
  * S3 bucket (file share) for Load Balancer Logs
    * Security Policy for access to S3 bucket
  * Load Balancer for backend `todo` service
    * Listener
    * Target Group
    * Target Group Attachment
  * DNS record for load balancer

---

# Firewall Security

* Open `security-groups.tf`
  * SSH to the backend server
  * Traffic between load balancer and `todo` service

---

# Outputs

* Open `outputs.tf`
  * Human and computer-readable data

---

# The Graph

![bg contain](images/graph.png)

---

# The Setup

* `terraform init`

---

# The Plan

* `terraform plan`

---

# The Apply

* `terraform apply`
  * If all looks good, answer: `yes`

---

# The Outputs

* `terraform output`

---

# The State File

```
$ terraform state list
$ terraform state show aws_instance.todo[0]
$ terraform state pull > \
    $HOME/class-terraform-starting/state.json
$ less $HOME/class-terraform-starting/state.json
$ rm $HOME/class-terraform-starting/state.json
```

---

# Examine the Server

* `ssh -i $HOME/.ssh/oreilly_aws ubuntu@${todo_ip}`
* `sudo systemctl status todo-list`
* `exit`
* `cd ..`

---

# Test the Todo API

```
$ source ./bin/ip_vars.sh
$ curl -i http://todo-api.spkane.org:8080/
$ curl -i http://todo-api.spkane.org:8080/ -X POST \
    -H 'Content-Type: application/spkane.todo-list.v1+json' \
    -d '{"description":"go shopping","completed":false}'
$ curl -i http://todo-api.spkane.org:8080/
$ curl -i http://todo-api.spkane.org:8080/1 -X DELETE \
    -H 'Content-Type: application/spkane.todo-list.v1+json'
$ curl -i http://todo-api.spkane.org:8080/
```

---

# Download the Todo Provider

* Open in your web browser:
  * https://github.com/spkane/todo-for-terraform/releases/tag/v1.0.0
* Download the `terraform-provider-todo` archive for your platform.

---

# Install the Todo Provider

* `cd $HOME/Downloads`
  * or where ever you downloaded the archive to.

```
$ unzip terraform-provider-todo-*.zip
$ mv terraform-provider-todo \
    $HOME/class-terraform-starting/todo-for-terraform/terraform-tests/
```

---

# Copy the Code

* `cd $HOME/class-terraform-starting/todo-for-terraform`
* `mkdir -p tf-code`
* `cp -a terraform-tests tf-code`
* `cd ./tf-code/terraform-tests`

---

# Defining Variables

* Open `variables.tf`
  * This is where we define variables we will use in the terraform code.

---

# Using the Todo Provider

* Open `main.tf`
  * Configure the todo provider
    * Change `host = "127.0.0.1"` to `host = "todo-api.spkane.org"`
  * Create 5 new todos
  * Read 1 existing todo as a data source
  * Create 5 more new todos based on the data source

---

# Defining Outputs

* Open `outputs.tf`
  * Prints the IDs for all of the new todos

---

# Prepare the Data

* We need a todo with ID 1 to read in as an example data source:

```
curl -i http://todo-api.spkane.org:8080/
curl -i http://todo-api.spkane.org:8080/ -X POST \
    -H 'Content-Type: application/spkane.todo-list.v1+json' \
    -d '{"description":"go shopping","completed":false}'
```

---

# macOS Catalina+ Notice

* You may need to whitelist the provider binary, since it is not signed.
  * Run `./terraform-provider-todo`
    * Click `Cancel`
  * Go to `System Preferences` → `Security & Privacy` → `General`
    * Click `Allow Anyway`
  * Run `./terraform-provider-todo`
    * Click `Open`

---

# Apply Terraform Code

* `terraform init`
* `terrraform apply`
  * **Plan**: 10 to add, 0 to change, 0 to destroy.
    * If all looks good, answer: `yes`

---

# Examine the Outputs

* `terraform output`

* You may notice that your IDs are likely not in order. This is because, by default terraform creates many of the resources in parallel and we have many students using the server at the same time.

---

# Examine The State File

* Examine the state from one of the resulting todos
  * `state show todo.test1[0]`

  ```terraform
  # todo.test1[0]:
  resource "todo" "test1" {
      completed   = false
      description = "0-1 test todo"
      id          = "6"
  }
  ```

---

# The Real Object

* From the output of the last command, grab the ID and use it at the end of this command.
* `curl -i http://todo-api.spkane.org:8080/6`

```json
HTTP/1.1 200 OK
Date: Wed, 01 Jan 2020 20:13:45 GMT
Content-Type: application/spkane.todo-list.v1+json
Content-Length: 59
Connection: keep-alive

[{"completed":false,"description":"0-1 test todo","id":6}]
```

---

# Updating Objects

* Change the 2 `count = 5` lines to read `count = 4`
* Add `(updated)` to the end of the first description string.

---

# Code With Edits

```terraform
resource "todo" "test1" {
  count = 4
  description = "${count.index}-1 test todo (updated)"
  completed = false
}

resource "todo" "test2" {
  count = 4
  description = "${count.index}-2 test todo (linked to ${data.todo.foreign.id})"
  completed = false
}
```

---

# Examine The First & Last Todo

* `terraform state show todo.test1[0]`
* `terraform state show todo.test1[4]`

---

# Apply The Updates

* `terrraform apply`
  * **Plan**: 0 to add, 4 to change, 2 to destroy.
    * If all looks good, answer: `yes`

---

# Re-examine The First & Last Todo

* `terraform state show todo.test1[0]`
  * The description should now be updated.
* `terraform state show todo.test1[4]`
  * This should give you an error since it has now been deleted.
* `terraform state show todo.test1[3]` will work however, since we only have 4 todos now.

---

# Prepare to Import

* Create a new todo by hand:

```
$ curl -i http://todo-api.spkane.org:8080/ -X POST \
    -H 'Content-Type: application/spkane.todo-list.v1+json' \
    -d '{"description":"Imported Todo","completed":false}'
```

  * Note the ID in your output (*13* in this example): `{"completed":false,"description":"Imported Todo","id":13}`

---

# Modify The Code

* In `main.tf` add:

```terraform
resource "todo" "imported" {
  description = "Imported Todo"
  completed = false
}
```

---

# Run a Plan

* `terraform plan`
  * You should see: **Plan**: 1 to add, 0 to change, 0 to destroy.

```terraform
  # todo.imported will be created
  + resource "todo" "imported" {
      + completed   = false
      + description = "Imported Todo"
      + id          = (known after apply)
    }
```

---

# Import a Pre-Existing Todo

* Import the ID of the Todo that you just created.
  * `terraform import todo.imported[0] 13`

---

# Re-run the Plan

* `terraform plan`
  * You should see
    * **No changes. Infrastructure is up-to-date.**

---

# Rename a Resource

* In `main.tf`:
  * change the line `resource "todo" "imported" {` to read `resource "todo" "primary" {`
* Run `terraform plan`
  * You should see
    * **Plan**: 1 to add, 0 to change, 1 to destroy.
* This would delete one todo and create a new one.
  * This is not what we want.

---

# Manipulating State

* `terraform state mv todo.imported todo.primary`
* Run `terraform plan`
  * You should see
    * **No changes. Infrastructure is up-to-date.**
* By moving the state of the existing resource to the new name, everything lines back up properly.

---

# Terraform Modules

* Blocks of re-useable Terraform code w/ inputs and outputs
* `cd ..`
* `cp -a ../__modules .`
* `cd __modules/todo-test-data`

---

# Module Variables

* Open `variables.tf`
  * This files defines all the variables that the module uses and any default values.

---

# The Main Module Code

* Open `main.tf`
  * This files will allow us to easily create two set of todos matching our specific requirements.

---

# Module Outputs

* Open `outputs.tf`
  * If you think of modules like functions then outputs are return values.

---

# Prepare to Use the Module

* `cd ../../code/`
* Open `main.tf`

---

# Utilize the Module (1 of 2)

* Add:

```terraform
module "series-data" {
    source       = "../__modules/todo-test-data"
    number       = 5
    purpose      = "testing"
    team_name    = "oreilly"
    descriptions = ["my first completed todo", "my second completed todo",
                    "my third completed todo", "my fourth completed todo",
                    "my fifth completed todo"
                   ]
}
```

---

# Utilize the Module (2 of 2)

* Open `outputs.tf`
* Add:

```terraform
output "first_series_ids" {
  value = "${module.series-data.first_series_ids}"
}

output "second_series_ids" {
  value = "${module.series-data.second_series_ids}"
}
```

---

# Apply the Module

* `terraform init`
  * Initialize/download the module.
* `terraform apply`
  * If all looks good, answer: `yes`

---

# Destroy the Todos

* `terraform destroy`
  * **Plan**: 0 to add, 0 to change, 9 to destroy.
    * If all looks good, answer: `yes`

---

# Destroy the Infrastructure

* `cd ../terraform-infrastructure/`
* `terraform destroy`
  * If all looks good, answer: `yes`

---

# What We Have Learned

* How to install Terraform
* The primary use case for Terraform
* How to install a provider and what they are for
* Creating, reading, updating, and deleting objects
* Reading data sources & importing existing objects
* Making & using modules
* What the Terraform state is
* and more...

---

<!-- _class: lead -->

# Additional Reading

[Terraform: Up & Running](https://www.terraformupandrunning.com/)
[Terraform Documentation](https://www.terraform.io/docs/index.html)

---

<!-- _class: lead -->

# Additional Learning Resources


## https://learning.oreilly.com/

---

<!-- _class: lead -->

# Student Survey

**Please take a moment to fill out the class survey linked to in the chat channel.**

O’Reilly and I value your comments about the class.

Thank you!
