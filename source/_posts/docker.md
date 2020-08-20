---
title: docker
tags: docker
date: 2019-05-05 16:47:47
---
**what is docker**
- Open platform
- Build, ship, and run applications
- Popular because:
  - Flexible
  - Lightweight
  - Interchangeable
  - Portable
  - Scalable
## List Docker CLI commands

``` sh
docker
docker container --help

## Display Docker version and info
docker --version
docker version
docker info

## Execute Docker image
docker run hello-world

## List Docker images
docker image ls

## List Docker containers (running, all, all in quiet mode)
docker container ls
docker container ls --all
docker container ls -aq


docker build -t friendlyhello .  # Create image using this directory's Dockerfile
docker run -p 4000:80 friendlyhello  # Run "friendlyhello" mapping port 4000 to 80
docker run -d -p 4000:80 friendlyhello         # Same thing, but in detached mode
docker container ls                                # List all running containers
docker container ls -a             # List all containers, even those not running
docker container stop <hash>           # Gracefully stop the specified container
docker container kill <hash>         # Force shutdown of the specified container
docker container rm <hash>        # Remove specified container from this machine
docker container rm $(docker container ls -a -q)         # Remove all containers
docker image ls -a                             # List all images on this machine
docker image rm <image id>            # Remove specified image from this machine
docker image rm $(docker image ls -a -q)   # Remove all images from this machine
docker login             # Log in this CLI session using your Docker credentials
docker tag <image> username/repository:tag  # Tag <image> for upload to registry
docker push username/repository:tag            # Upload tagged image to registry
docker run username/repository:tag                   # Run image from a registry
```

# Beginning Devops with Docker
## Chapter 1. Images and Containers
**Virtualization versus Containerization**
![virtual machine block diagram](https://i.imgur.com/KKfsLxn.png)
In virtual machines, the physical hardware is abstracted, therefore we have may server running on one server. Virtual machines do sometimes take time to start up and are expensive in capacity (they can be GBs in size), although the greatest advantage they have over containers is the ability to run different Linux distributions such as CentOS instead of just Ubuntu.

![containerization](https://i.imgur.com/rWfUi8x.png)
In containerization, it is only the app layer (where code and dependencies are packaged) that is abstracted, making it possible for many containers to run on the same OS kernel but on separate user space.

Containers use less space and boot fast.

- Describe how Docker improves a DevOps workflow
Normally, we have two pieces to a working application: the project code base and the provisioning script. The code base is the application code. It is managed by version control and hosted in GitHub, among other platforms.
![Docker implements](https://i.imgur.com/7bpoPrj.png)
The **Dockerfile** takes the place of the provisioning script. The two combined (project code and Dockerfile) make a **Docker image**. A Docker image can be run as an application. This running application sourced from a Docker image is called a **Docker container**.

**Basic Docker Terminal Commands**
```
docker info ## displays system-wide information
docker search <term> (foir example. docker search ubuntu) ##searches Docker Hub for Images
docker pull ## pulls an image from the registry to your local machine eg docker pull ubuntu
docker images ## lists the Docker images we have locally
docker run <image> #will check whether the image by the name hello-world exists locally,if the image is
not local, it will be pulled form the default registry, Docker Hub, and run as a container, by default

docker ps ## lists the running containers
docker stop <container ID>  ## stop the container 
docker rm <container ID> ## Delete the container and the image
```

- Interpret Dockerfile syntax
- Build images
- Set up containers and images
- Set up a local dynamic environment
- Run applications in Docker containers
- Obtain a basic overview of how Docker manages images via Docker Hub
- Deploy a Docker image to Docker Hub

**The Dockerfile**
```
FROM openjdk:8-jre-alpine

ENV SPRING_OUTPUT_ANSI_ENABLED=ALWAYS \
    JHIPSTER_SLEEP=0 \
    JAVA_OPTS=""

CMD echo "The application will start in ${JHIPSTER_SLEEP}s..." && \
    sleep ${JHIPSTER_SLEEP} && \
    java ${JAVA_OPTS} -Djava.security.egd=file:/dev/./urandom -jar /app.war

EXPOSE 8080 5701/udp

ADD *.war /app.war
```
The **FROM** instruction specifies the base image to use while initializing the build. Here, we specify open JDK 8 as our Java runtime.

The **ENV** instruction is used to set environment variables, and the **CMD** instruction is used to specify commands to be executed.

The **EXPOSE** instruction is used to specify the port that the container listens to during runtime.

**useful commands for Docker and Docker compose**
![commands1](https://i.imgur.com/FAExCVU.png)
![commands2](https://i.imgur.com/LtTRrfD.png)

## Docker-compose
**Sample docker-compose file that will start a Rdis database on port 5000**
```
version: '3'
services:
  web:
    build: .
    ports:
     - "5000:5000"
  redis:
    image: "redis:alpine"
```
The first line of the *docker-compose* file should be the version of the *docker-compose* tool. 

Then we need to specify all the necessary services that we need for our application to run. They should be defined in the *services:* section.

We can also define multiple services inside here, giving a name to each (*web* and *redis*).

This is followed by how to build the *service* (either via a command to build or referring a Dockerfile). 

If the application needs any port access, we can configure it using *5000:5000* (that is internal port: external port). 

Then, we have to specify the volume information. This basically tells *docker-compose* to serve the files from the location specified.   

Once we have specified the services required for our application, then we can start the application via *docker-compose*. This will start your entire application along with the services, and expose the *services* on the port specified. 

With *docker-compose*, we can perform the following operations:

- **Start**: docker-compose -f <docker_file> up
- **Stop**: docker-compose -f <docker_file> down  
We can also perform the following operations:

- **List the running services and their status**: *docker ps*
- **Logs**: *docker log <container_id>*

## Kubernetes
A single deployable component is called a **pod** in Kubernetes. This can be as simple as a running process in the container. A group of pods can be combined together to form a **deployment**. 

The following code is a sample Kubernetes file that will start a Nginx server:
```
apiVersion: v1
kind: Service
metadata:
  name: nginxsvc
  labels:
    app: nginx
spec:
  type: NodePort
  ports:
  - port: 80
    protocol: TCP
    name: http
  - port: 443
    protocol: TCP
    name: https
  selector:
    app: nginx
```
1. Start with an *apiVersion* in a Kubernetes deployment file.

2. Followed by the type, which takes either a pod, deployment, namespace, ingress (load balancing the pods), role, and many more.

Ingress forms a layer between the services and the internet so that all the inbound connections are controlled or configured with the ingress controller before sending them to Kubernetes services on the cluster. On the other hand, the egress controller controls or configures services going out of the Kubernetes cluster.

3. This is followed by the metadata information, such as the type of environments, the application name (nginxsvc), and labels (Kubernetes uses this information to identify and segregate the pods). Kubernetes uses this metadata information to identify the particular pods or a group of pods, and we can manage the instances with this metadata. This is one of the key differences with docker-compose, where docker-compose doesn't have the flexibility of defining the metadata about the containers.

4. This is followed by the spec, where we define the specification of the images or our application. We can also define the pull strategy for our images as well as define the environment variables along with the exposed ports. We can define the resource limitations on the machine (or VM) for a particular service. They provide health checks, that is, each service is monitored for the health and when some services fail, they are immediately replaced by newer ones. They also provide service discovery out of the box, by assigning each pod an IP, which makes it easier for the services to identify and interact with them. They also provide a better dashboard, to visualize your architecture and the status of the application. You can do most of the management via this dashboard, such as checking the status, logs, scale up, or down the services, and so on.

## Docker ps command
```bash
docker ps --format "table {{.ID}} \t {{.Image}} \t {{.Ports}} \t {{.Names}}"
CONTAINER ID         IMAGE                   PORTS                                                NAMES
7d3a6d518e8e         jenkinsci/blueocean     0.0.0.0:8080->8080/tcp, 0.0.0.0:50000->50000/tcp     jenkins
9655eb0e836e         alpine                                                                       quotes
root@stan-OptiPlex-380:~|⇒  docker ps --format "table {{.ID}} \t {{.Names }} \t {{.Status}}"
CONTAINER ID         NAMES               STATUS
7d3a6d518e8e         jenkins             Up 2 minutes
9655eb0e836e         quotes              Up 4 hours
```

## Docker networking
-Mostly used types
  - Bridge networks
    - The default network driver
    - Applications run in standalone containers that need to communicate
  - Overlay networks:
    - Connect multiple Docker daemons together
    - Enable swarm services to communicate with each other

**Docker basic networking**
- Step 1: create a network and volume

Listing and creating a network:
```
docker network ls
docker network create net1
```
Listing and creating a volume:
```
docker volume ls
docker volume create --driver=local --name=volume1
```
- Step 2: start docker containers
  - Starting docker containers:
```
docker run -it --name=container1 -v volume1:/opt --network=net1 busybox sh
docker run -it --name=container2 -v volume1:/opt --network=net1 busybox sh
#-t is used for interactive session and t will give you the terminal
```
- Step 3: test connectivity between these containers
```
# ping container2
PING container2 (172.19.0.3): 56 data bytes
64 bytes from 172.19.0.3: seq=0 ttl=64 time=0.170 ms
64 bytes from 172.19.0.3: seq=1 ttl=64 time=0.134 ms
64 bytes from 172.19.0.3: seq=2 ttl=64 time=0.133 ms
```
**Overlay networking**

**What does an overlay network do?**
- Containers only connect to other containers on the same network 
- Connecting to containers on other hosts requires that the container publish the needed port 
- Overlay network allows containers running on different hosts to connect on a private network 
![overlay networking](https://i.imgur.com/KjgJtA0.png)
- Step 1: Initiate docker swarm
> Host1: Check networks and initiate docker swarm:
   docker network ls
   docker swarm init
  Host2: Initiate swarm worker and check its networks:
   docker swarm join --toaken foobar 192.168.10.91:2377 
   docker network ls
- Step 2: Create overlay network and spin a docker container on host1
> Create an overlay network:
   docker network create -d overlay --atachable=true swarmnet
   - List networks:
   docker network ls
   - Spin a Docker container:
   docker run -itd --name container1 --network swarmnet busybox

- Step 3: Spin a container on host2 and check its networks
> List networks:
   docker network ls
   - Spin a Docker container:
   docker run -itd --name container2 --network swarmnet busybox
   - List networks and see new network:
   docker network ls

- Step 4: Login to the containers check its networks and sing each other
> Host1: Log in to the container, check IPs, and ping container on Host2:
     docker exec -it container1 sh
     ifconfig
     ping container2
  Host2:  Log in to the container, check IPs, and ping container on Host1:
     docker exec -it container2 sh
     ifconfig
     ping container1

# upload that new image to dockerhub
```
docker tag helloworld stanosaka/helloworld
docker push stanosaka/helloworld
```

Cleaning up containers:We can clean up everything by stopping and removing all containers with these two handy one-line commands:
```
$ docker stop $(docker ps -a -q)
$ docker system prune
```
# Amazon web services and docker development
```
apt-get update
apt-get install     apt-transport-https     ca-certificates     curl
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key
add-apt-repository    "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
apt update
apt-get install docker-ce docker-ce-cli containerd.io
systemctl start docker
systemctl status docker

root@ip-172-31-6-219:~# cat dockerfile
FROM ubuntu:16.04


RUN apt update
RUN apt install -y apache2
RUN echo "Welcome to my web site" > /var/www/html/index.html
CMD /usr/sbin/apache2ctl -D FOREGROUND
EXPOSE 80


docker info
docker run -it --detach=true ubuntu bash
docker ps
docker renmae nervous_visvesvaraya newname
docker network inspect newnet|less
docker network connect newnet newname
docker inspect newname|less
ping 172.18.0.2
ping 172.17.0.2
docker build -t "webserver" .
docker images
docker run -d -p 80:80 webserver /usr/sbin/apache2ctl -D FOREGROUND
docker ps
curl localhost
 aws iam create-service-linked-role --aws-service-name ecs.amazonaws.com
{
    "Role": {
        "AssumeRolePolicyDocument": {
            "Version": "2012-10-17",
            "Statement": [
                {
                    "Action": [
                        "sts:AssumeRole"
                    ],
                    "Effect": "Allow",
                    "Principal": {
                        "Service": [
                            "ecs.amazonaws.com"
                        ]
                    }
                }
            ]
        },
        "RoleId": "AROAYYRUPCKTK7Z2M2PLO",
        "CreateDate": "2019-09-02T02:28:51Z",
        "RoleName": "AWSServiceRoleForECS",
        "Path": "/aws-service-role/ecs.amazonaws.com/",
        "Arn": "arn:aws:iam::602479202982:role/aws-service-role/ecs.amazonaws.com/AWSServiceRoleForECS"
    }
}
```
### Working with ecs throught the aws cli
```
aws ecs list-clusters
aws ecs describe-clusters
aws ecs create-cluster --cluster-name newcluster
```
**Configure ECS to authenticate with Docker Hub**
```
#1.ssh to ecs server
cd /etc/ecs
[root@ip-172-31-13-18 ecs]# vim ecs.config
ECS_ENGINE_AUTH_TYPE=docker
ECS_ENGINE_AUTH_DATA={"https://index.docker.io/v1/":{"username":"foobar","password":"badpassword","email":"foo.bar@gmail.com"}}
[root@ip-172-31-13-18 ecs]# stop ecs
ecs stop/waiting
[root@ip-172-31-13-18 ecs]# start ecs
ecs start/running, process 4996
[root@ip-172-31-13-18 ecs]# docker inspect ecs-agent|grep ECS_DATADIR
                "ECS_DATADIR=/data",
#2. make repo private in docker hub
#3. configure in aws console about ecs
```

**working with ec2 container registry (ECR)**
```
aws ecr get-login help
docker login -u AWS -p badpassword https://602479202982.dkr.ecr.ap-southeast-2.amazonaws.com
docker images
docker tag c0a44e0f3831 602479202982.dkr.ecr.ap-southeast-2.amazonaws.com/myrepo
docker push 602479202982.dkr.ecr.ap-southeast-2.amazonaws.com/myrepo
```

## Docker registry
```
docker run -d -p 5000:5000 --restart always --name registry registry:2
docker pull hello-world
docker tag hello-world localhost:5000/hello-world
docker push localhost:5000/hello-world
docker exec -it registry /bin/sh
docker images
docker rmi hello-world
docker rmi localhost:5000/hello-world
docker images
docker pull localhost:5000/hello-world
docker run \
  -d \
  -e ENV_DOCKER_REGISTRY_HOST=registry \
  -e ENV_DOCKER_REGISTRY_PORT=5000 \
  -p 8080:80 \
  --link registry:registry \
  konradkleine/docker-registry-frontend:v2
```


## Error
1. 
```
 docker login 127.0.0.1:5000
Username: myuser
Password:
** Message: 17:25:24.126: Remote error from secret service: org.freedesktop.Secret.Error.IsLocked: Cannot create an item in a locked collection
Error saving credentials: error storing credentials - err: exit status 1, out: `Cannot create an item in a locked collection`
```

Fix by:

```
docker logout
Removing login credentials for https://index.docker.io/v1/
➜  outyet git:(master) mv ~/.docker/config.json ~/.docker/config_old.json
➜  outyet git:(master) docke login
zsh: command not found: docke
➜  outyet git:(master) docker login 127.0.0.1:5000
Username: myuser
Password:
WARNING! Your password will be stored unencrypted in /home/stan/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
```
