---
title: docker-orchestration-and-microservices
tags:
   - docker
   - orchestration
   - microservices
---
# Microservices
## Understand microservices
**Traditional "Monolithic" Web Apps**
- All functionality is within this single app, regardless of the role
- A back end server (Rails, PHP, etc..) that dynamically generates HTML to send to the client
- One giant deployment

**Microservice-Oriented Architecture**
- Applications are compositions of small and independent processes
- These processes generally communicate throught a REST API that responds to HTTP requests, with JSON encoded repsonses
- Often a front end application (Angular, React, etc...) will use these JSON representations to build the actual HTML consumed by the client
- Other times an iOS or Andriod app will build their views using these JSON reponses
- More involved procedures that can be run independently are located in separate codebases/servers (i.e., payment processing)
- The overall application will be composed of there microservices. They are often "piped" together much like Unix piping small command line grograms together
- A database must be accessed through its RESTful API. No directly accessing it
- Each microservice should be standalone, loosely coupled, and able to be deployed separately
- A good test: an outside provider should be able to build something using your APIs (e.g., create a new front end using your back end app)

**Why Microservices?**
- Monolithic apps are usually harder to maintain over time
- Doesn't ite you to one "stack". Language agnostic (different languages/frameworks for different services)
- Horizontal scaling: increased performance of app by add more intance
- Fits nicely with the Docker paradigm

**Difficulties introduced by Microservices**
- Each microservice will have associated overhead, and you must deal with this complexity
- Deployments are more complex
- Sometimes hard to deine up front
- Monolith first
- Docker can help ease these issues and is a great companion to microservices

**An Online Store**
- An online store is a very common form of web app that fits very well in the microservices model
- Amazon.com is the most commonly cited storefront example of the benefits of switching to a service-ortented architecture, with over 100 services called on a page visit

**Example Services**
- Billing microservice
- Inventory microservice
- Shipping microservice
- Storefront microservice

**API Gateway Pattern**
- While this application is composed of multiple services, users should still be able to interface with it as if it was one application
- To solve this problem, we will have an API gateway that sits in front of all the back end microservices, and makes the proper calls to each microservice. It acts sort of like a load balancer does
- Our front end service, however, will sit on the other side of the API gateway, and use the API back end to talk to the other services

## Dockerizing Microservices
**The Docker connection**
- As alluded to, Docker can be a great companion for a microservice or service-oritented architecuture (SOA)
- This allows us to not only easily get the right environment for each service, but it's a greate form of documenting system level dependencies
- We will later define a container "topology" that declares microservice dependencies for the local development, testing, and deployment

**Dockerizing?**
- Dockerizing can be defined as modifying an app to run insdie Docker containers
- Commonly includes also making a custom Docker image for the application
- Each microservice must be Dockerized

**Common hurdles to dockerizing**
- Configuration is typically done via environment variables. If your app is configured via files, some adjustments might be required
- Properly capturing the logging of your application
- Authoring a robust Dockerfile can be tricky

**Avoid reinventing the wheel**
- There are many high quality official Docker images on Docker Hub to start from
- A great way to start out is to use one of these as a base in your FROM statement
- After specifying the base image, you can add your own customizations
- Forking the original is also an option, but it's usually not preferred
- Be very careful with base images, and be sure to review the code for malicious content!(Anyone can push to Docker Hub!)

**Dockerizing a Basic Express Restful API**
- This process applies to any back end web server
- The Express framework for Node.js is lightweight and easy to understand. This makes it a decent choice for an example back end
- Feel free to- ant I would advise this- repeat the process in your flavor of back end!

**Super basic express app**
- Application code (server.js)
- Application dependencies (package.json)
- System dependencies (MongoDB)
**Create a container for each app**
```
// inside directory 
// microservice-externaldb/
// officalNodeDockerfile
// node-microservice/
// two-microservices/

// terminal commands
// vi officialNodeDockerfile
/*
FROM buildpack-deps:jessie

#verify gpg and sha256: http://nodejs.org/dist/v0.10.30/SHASUMS256.txt.asc
#gpg: aka "Timothy J Fontaine (Work) <tj.fontaine@joyent.com>"
#gpg: aka "Julien Gilli <jgilli@fastmail.fm>"
RUN gpg --keyserver pool.sks-keyservers.net --recv-keys 7937DFD2AB06298B229

EVN NODE_VERSION 0.10.40
ENV NPM_VERSION 2.11.3

RUN curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/node-v$NODE_VERSION-l"
		&& curl -SLO "https://nodesjs.ord/dist/v$NODE_VERSION/SHASUM256.txt.asc"
		&& gpg --verify SHASUMS256.txt.asc \
		&& grep " node-v$NODE_VERSION-linux-x64.tar.gz\$" SHASUMS256.txt.asc |
		&& tar -xzf "node-v$NODE_VERSION-linux-x64.tar.gz" -C /usr/local --stri
		&& rm "node-v$NODE_VERSION-linux-x64.tar.gz" SHASUMS256.txt.asc \
		&& npm install -g npm@"$NPM_VERSION" \
		&& npm cache clear

CMD [ "node" ]		
*/

// next terminal command 
// docker run -it node
// docker pull node
// docker run -it node   "drops into node terminal"
// console.log('hello');

// terminal commands
//cd node-microservice/
//inside node-microservice/
//Dockerfile 
//node_modules/ 
//package.json
//server.js

//terminal  
//vi server.js

/* inside server.js 
var express = require('express');
var app = express();

app.get('/', function(req, res) {
	res.json({'msg': 'helloworld'})
});

var port = process.env.port || 3000;
app.listen(port, function() {
	console.log('server started');
});
*/

//terminal
// node server.js &
// curl http://localhost:3000
// vi Dockerfile

/* inside DockerFile
FROM node:0.12.7

RUN mkdir -p /user/src/app
WORKDIR /usr/src/app

COPY . /usr/src/app
RUN npm install

EXPOSE 3000

CMD [ "npm", "start" ]
*/

//terminal
//npm install
//docker build -t myservice .
//docker run -P -d myservice
//docker ps
//curl http://localhost:37269
//git init
//git add Dockerfile package.json server.js
//git commit -m 'initial commit dockerized app'
```
## Configure your apps for both production and development
**A general case deployable image**
- Dockerfiles and images are meant to be general, and adaptable. They should not in any way be coupled to your machine

- However, you will often need to point your Docker containers at your host system, such as for mounting a volume, or in some way need to customize your container

- The proper place to bo these sorts of customization is in the floags for the Docker command, or as we'll see later, in a docker-compose.yml file

# Using registries to store and distribute docker images
**Docker Hub**
- A "GitHub for Docker files"
- DOcker's hosted offering of a Docker registry
- Provided by the creators of Docker
- Unlimited free public repositories
- One free private repository
- Additional private repositories for a fee

**Git-like workflow**
- Has an analogous pubh and pull operation
- Docker pub is to Docker what Github is to Git (roughly)

**Automated Image Builds**
- Docker Hub can automatically create up-to-date image builds for us
- To do this, you create a Github or Bitbucket repo with your Dockerfile and related code
- This will be VERY useful to us later, as a way to lock to specific versions of our image, for both development and pushing to production

**Other hosted Docker registries**
- Google Container Registry
- Quay.io
- You can also host a docker registry on your own hardware

**Tips on labeling images**
- Don't use latest tag in production!
- You can either manually create version number so fro production, or trigger your automated build to specific tags

**Configure your docker image to be pushed to docker hub**
```
//terminal
// docker build -t stanosaka/myservice:latest .  "specifying a specific githube location"
// docker login
// docker push stanosaka/myservice
// docker pull stanosaka/myservice
```
**Configure your docker image to be rebuit upon code pushes to Github**
```
/* just github setup and docker setup
create a repository in github and create automated build on dockerhub
find the github repository

*/

//terminal 
// vi server.js
/*
change console.log in server.js file to "Hello world again"
*/
//terminal
// git add server.js
// git commit - m "update app"
// git push origin master
```
## Docker Machine
**What Docker machine is**
- A way to remotely create, deploy, and manage Docker containers
- Has drivers to interface with various cloud providers (AWS,DigitalOcean, Azure, and many more!)
- Also has drivers for local virtual machines
- Interfaces with Docker Compose and Docker Swarm (more on that later!)

**How docker machine works**
- Create the underlying host, be it an AWS instance, a VirtualBox VM, or whichever driver you specify
- Installs a Docker Daemon on the host, that you can interact with remotely via the Docker CLI on your host, as if it was your local Docker Daemon
- By default, this remote operations will be done over a secure TLS connection
- A ~/.docker/machine will also be created on your local machine
- This directory contains your cert files
- This directory also maintains a list of your Docker machines, the necessary information to find them , and various meta-data about your Docker Machines

**Use docker machine to simulate deployment with a vm**
```
//terminal
// which docker-machine
$ base=https://github.com/docker/machine/releases/download/v0.16.0 && curl -L $base/docker-machine-$(uname -s)-$(uname -m) >/tmp/docker-machine && sudo install /tmp/docker-machine /usr/local/bin/docker-machine
// docker-machine ls
// docker-machine create --driver virtualbox dev
// docker-machine ls
// docker-machine env dev
// docker info
// eval $(docker-machine env dev)
// docker info
// docker-machine env
// docker-machine env dev
// set SHELL /bin/bash
// docker-machine env dev
// cd ~/.docker/machine/machines/dev
// pwd
// cd dev/
// docker-machine ssh dev
// ls


//more terminal comands
// docker info
// docker run -d -p 80:3000 stanosaka/myservice-image
// docker ps
// docker-machine ls
// curl http://192.168.99.100
// docker-machine ip dev
// curl $(docker-machine ip dev)
```
**Use docker machine to deploy to a cloud providedr**
```
//terminal 
// sub=$(az account show --query "id" -o tsv)
// docker-machine create -d azure --azure-subscription-id $sub --azure-ssh-user azureuser --azure-open-port 80 --azure-size "Standard_DS2_v2" --azure-location "australiaeast" staging
// docker-machine ls
// eval (docker-machine env staging)
// docker status
// docker info
// docker-machine ssh staging
// ls
// docker ps
// ctrl c     (exiting)
// docker run -d -p 80:3000 cadbot/myservice
// docker-machine ip staging
// docker-machine ls
// curl $(docker-machine ip staging)
```
**Most suitable for single-instance single-container**
- Basic multi-instance configuration can be done through container env vars
- Only sufficient for simple configuration options, such as remote database URIs
- Multiple containers can run on the same host, if they are mapped to different ports
- Configuring communication between these containers can be complex

**More complex app toplogies with Docker Mahine (sneak peek)**
- Docker Compose gives us the tools to easily configure multiple containers on the same host
- Limited abilities with cross host multi-container apps
- On the other side of the conin, Docker Swarm allows us to easily create multiple Docker hosts!
- Bringing it all together, we'll be able to create fairly robust multi-container multi-host applications!

**Runing a web service locally**
- In a monolithic web app,we typically only have to run one process to get our entire app running local
- In a microservice architecutre, we have to run a set of services in order to run our application locally
- Sometimes you'll want to run the entire "service" locally, other times just a subset

**Defining topologies with Docker Compose**
- Born from the open source Fig project
- A simple docker-compose.yml file enables you to specify a topology of containers
- You will typically place one of these yaml files in each mircoservice
- when invoked, it will start a "mini" local version of your service

**Defining our first container's config**
- Instead of building/running our image directly via docker run, we can declare if the image should be built or pulled in docker-compose.yml, and run it that way
- Another benefit of Docker Compose is it allows us to save our service development specific configs such as volumes and ports. This is a huge win, as Dockerfiles do not allow you to specify a container to host port mapping. As Docker Compose files are meant to be more specific and less general purpose, they give us a greater degree of control

**Docker Volumes**
- All data from Docker containers are lost except the volumes from the container
- You can use a volume by specifying a location inside of the Docker container
> when specified this way, there will be a Docker volume on your host that persists after the Docker container is stopped

- you can also specify a directory on your host as a volume
> This allows you to update the source code in your container without having to create a new image or container each time

**Define application topologies with docker compose**
```
//terminal
// vi server.js

/* server.js 
var express = require('express');
var app = express();

app.get('/', functin(req, res) {
	res.json({'msg': 'some message'})
});

var port = process.env.port || 300;
app.listen(port, function() {
	console.log('server started');
});
*/

//terminal
// vi docker-compose.yml

/* docker-compose.yml
webservice:
	build: . 
	ports:
		- "3000:80"
	environment: 
		port: "80"
*/

//terminal
// docker-compose up
// docker-compose run webservice
// docker ps
// curl http://localhost:3000
// docker-compose ps
// docker ps
// docker stop service_webservice_run_1
// curl http://localhost:3000
// vi server.js

/* changes to server.js

change res.json msg to 'hello world!'

*/

//terminal
// docker-compose up -d
// curl http://localhost:3000
// docker-compose build webservice
// curl http://localhost:3000


//more terminal 
// vi docker-compose.yml

/*
webservice:
	build: . 
	ports:
		- "3000:80"
	environment: 
		port: "80"
	volumes:
		- .:/user/src/app

*/

//terminal
// vi Dockerfile
// vi server.js
// docker-compose up -d
// curl http://localhost:3000
// docker stop 19868a4cae3b
// vi server.js
// docker-compose up -d
// curl http://localhost:3000
// docker ps
// docker exec -it 44262af7e8eb /bin/bash
// touch tempfile
```
