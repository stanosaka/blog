---
title: anatomy-of-containers
date: 2019-09-20 10:05:28
tags: [containers,docker]
---
# Anatomy of containers
Containers are specially encapsulated and secured processes running on the host system.

Containers leverage a lot of features and primitives available in the Linux OS. The most important ones are
- namespaces
- cgroups

All processes running in containers share the same Linux kernel of the underlying host operating system. This is fundamentally different compared with VMs, as each VM contains its own full-blown operating system.

Difference:

|             |Container   |VMs            |
|-------------|------------|---------------|
|startup times|milliseconds|several seconds|
|goal         |ephemeral   |long-living|

![high level architecture of Docker](https://i.imgur.com/3ReVPcO.png)
On the lower part of the the preceding figure, we have the Linux operating system with its cgroups, namespaces, and layer capabilities as well as other functionality that we do not need to explicitly mention here. Then, there is an intermediary layer composed of **containerd** and **runc**. On top of all that now sits the Docker engine. The Docker engine offers a RESTful interface to the outside world that can be accessed by any tool, such as the Docker CLI, Docker for Mac, and Docker for Windows or Kubernetes to just name a few.

## Namespaces
A namespace is an abstraction of global resources such as filesystems, network access, process tree (also named PID namespace) or the system group IDs, and user IDs.
![virtual FS](https://i.imgur.com/GVSkK6z.png)
The PID namespace is what keeps processes in one container from seeing or interacting with processes in another container. A process might have the apparent PID 1 inside a container, but if we examine it from the host system, it would have an ordinary PID, say 334:
![Process tree on a Docker host](https://i.imgur.com/zhefA3P.png)

## Control groups (cgroups)
Linux cgroups are used to limit, manage, and isolate resource usage of collections of processes running on a system.
Resources are CPU time, system memory, network bandwidth, or combinations of there resources, and so on.

Using cgroups, admins can limit the resources that containers can consume.With this, one can avoid, for example, the classical noisy neighbor problem, where a rogue process running in a container consumes all CPU time or reserves massive amounts of RAM and, as such, starves all the other processes running on the host, whether they're containerized or not.

## Union filesystem (UnionFS)
The UnionFS forms the backbone of what is known as container images.
UnionFS is mainly used on Linux and allows files and directories of distinct filesystems to be overlaid and with it form a single coherent file system.In this context, the individual filesystems are called branches. Contents of directories that have the same path within the merged branches will be seen together in a single merged directory, within the new, virtual filesystem. When merging branches, the priority between the branches is specified. In that way, when two branches contain the same file, the one with the higher priority is seen in the final FS.

## Container plumbing
The basement on top of which the Docker engine is built; we can also call it the container plumbing and is formed by the two component—runc and containerd.
## Runc
Runc is a lightweight, portable container runtime. It provides full support for Linux namespaces as well as native support for all security features available on Linux, such as SELinux, AppArmor, seccomp, and cgroups.

Runc is a tool for spawning and running containers according to the Open Container Initiative (OCI) specification.

## Containerd
Runc is a low-level implementation of a container runtime; containerd builds on top of it, and adds higher-level features, such as image transfer and storage, container execution, and supervision, as well as network and storage attachments.

# Creating and managing container images
## The layered filesystem
![stack of layers](https://i.imgur.com/Np4zxp1.jpg)
The layers of a container image are all immutable. Immutable means that once generated, the layer cannot ever be changed. The only possible operation affecting the layer is the physical deletion of it. 
![A sample custom image based on Alpine and Nginx](https://i.imgur.com/CkLp7N6.jpg)
Each layer only contains the delta of changes in regard to the previous set of layers. The content of each layer is mapped to a special folder on the host system, which is usually a subfolder of /var/lib/docker/.

Since layers are immutable, they can be cached without ever becoming stale. This is a big advantage.
## The writable container layer
![the writable container layer](https://i.imgur.com/Gio85pZ.jpg)
![Multiple containers sharing the same image layers](https://i.imgur.com/E80YSMK.jpg)
The container layer is marked as read/write. Another advantage of the immutability of image layers is that they can be shared among many containers created from this image. All that is needed is a thin, writable container layer for each container.

This technique, of course, results in a tremendous reduction of resources that are consumed. Furthermore, this helps to decrease the loading time of a container since only a thin container layer has to be created once the image layers have been loaded into memory, which only happens for the first container.

## Copy-on-write
Docker uses the copy-on-write technique when dealing with images. Copy-on-write is a strategy of sharing and copying files for maximum efficiency. If a layer uses a file or folder that is available in one of the low-lying layers, then it just uses it. If, on the other hand, a layer wants to modify, say, a file from a low-lying layer, then it first copies this file up to the target layer and then modifies it.
![copy-on-write](https://i.imgur.com/hTj3sG0.jpg)
The second layer wants to modify **File 2**, which is present in the base layer. Thus, it copied it up and then modified it. Now, let's say that we're sitting in the top layer of the preceding figure. This layer will use **File 1** from the base layer and **File 2** and **File 3** from the second layer.

## Creating images
Three ways to create a new container image on your system.
1. Interactive image creation
start with a base image that we want to use as a template and run a container of it interactively. 
```
$ docker container run -it --name sample alpine /bin/sh

#By default, the alpine container does not have the ping tool installed. Let's assume we want to create a new custom image that has ping installed. 

# apk update && apk add iputils
 # ping 127.0.0.1
PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.
64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.060 ms
64 bytes from 127.0.0.1: icmp_seq=2 ttl=64 time=0.054 ms
64 bytes from 127.0.0.1: icmp_seq=3 ttl=64 time=0.059 ms
^C
$ docker container ls -a | grep sample
#If we want to see what has changed in our container in relation to the base image
$ docker container diff sample
C /var
C /var/cache
C /var/cache/apk
A /var/cache/apk/APKINDEX.00740ba1.tar.gz
A /var/cache/apk/APKINDEX.d8b2a6f4.tar.gz
C /bin
C /bin/ping
C /bin/ping6
A /bin/traceroute6
C /lib
C /lib/apk
C /lib/apk/db
C /lib/apk/db/triggers
C /lib/apk/db/installed
C /lib/apk/db/scripts.tar
C /usr
C /usr/sbin
C /usr/sbin/arping
A /usr/sbin/clockdiff
A /usr/sbin/rarpd
A /usr/sbin/setcap
A /usr/sbin/tracepath
A /usr/sbin/tracepath6
A /usr/sbin/ninfod
A /usr/sbin/getcap
A /usr/sbin/tftpd
A /usr/sbin/getpcaps
A /usr/sbin/ipg
A /usr/sbin/rdisc
A /usr/sbin/capsh
C /usr/lib
A /usr/lib/libcap.so.2
A /usr/lib/libcap.so.2.27
C /etc
C /etc/apk
C /etc/apk/world
C /root
A /root/.ash_history

#In the preceding list, A stands for added, and C for changed. If we had any deleted files, then those would be prefixed with D.
#use the docker container commit command to persist our modifications and create a new image from them:
$ docker container commit sample my-alpine
sha256:31da02222ee7102dd755692bd498c6b170f11ffeb81ca700a4d12c0e5de9b913

## If we want to see how our custom image has been built
$ docker image history my-alpine
IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
31da02222ee7        38 seconds ago      /bin/sh                                         1.76MB
961769676411        4 weeks ago         /bin/sh -c #(nop)  CMD ["/bin/sh"]              0B
<missing>           4 weeks ago         /bin/sh -c #(nop) ADD file:fe64057fbb83dccb9…   5.58MB
#The first layer in the preceding list is the one that we just created by adding the iputils package.
```
2. Using Dockerfiles
The Dockerfile is a text file that is usually literally called Dockerfile. It contains instructions on how to build a custom container image. It is a declarative way of building images.
```
FROM python:2.7
RUN mkdir -p /app
WORKDIR /app
COPY ./requirements.txt /app/
RUN pip install -r requirements.txt
CMD ["python", "main.py"]
```
![the relation of Dockerfile and layers in an image](https://i.imgur.com/NMD2UG5.jpg)

**The FROM keyword**
Every DOckerfile starts with the FROM keyword. With it, we define which base image we want to start building our custom image from. If build starting with CentOS 7:
```
FROM centos:7
```

If we really want to start from scratch:
```
FROM scratch
```
FROM scratch is a no-op in the Dockerfile, and as such does not generate a layer in the resulting container image.

**The RUN keyword**
The argument for RUN is any valid Linux command, such as the following:
```
RUN yum install -y wget
```
If we use more than one line, we need to put a backslash (\) at the end of the lines to indicate to the shell that the command continues on the next line.
**The COPY and ADD keywords**
add some content to an existing base image to make it a custom image. Most of the time, these are a few source files of, say, a web application or a few binaries of a compiled application.

These two keywords are used to copy files and folders from the host into the image that we're building. The two keywords are very similar, with the exception that the ADD keyword also lets us copy and unpack TAR files, as well as provide a URL as a source for the files and folders to copy.
```
COPY . /app
COPY ./web /app/web
COPY sample.txt /data/my-sample.txt
ADD sample.tar /app/bin/
ADD http://example.com/sample.txt /data/
```
In the preceding lines of code:
- The first line copies all files and folders from the current directory recursively to the /app folder inside the container image
- The second line copies everything in the web subfolder to the target folder, /app/web
- The third line copies a single file, sample.txt, into the target folder, /data, and at the same time, renames it to my-sample.txt
- The fourth statement unpacks the sample.tar file into the target folder, /app/bin
- Finally, the last statement copies the remote file, sample.txt, into the target file, /data

Wildcards are allowed in the source path. For example, the following statement copies all files starting with sample to the mydir folder inside the image:
```
COPY ./sample* /mydir/
```

From a security perspective, it is important to know that by default, all files and folders inside the image will have a user ID (UID) and a group ID (GID) of 0. The good thing is that for both ADD and COPY, we can change the ownership that the files will have inside the image using the optional --chown flag, as follows:
```
ADD --chown=11:22 ./data/files* /app/data/
```
The preceding statement will copy all files starting with the name web and put them into the /app/data folder in the image, and at the same time assign user 11 and group 22 to these files.

Instead of numbers, one could also use names for the user and group, but then these entities would have to be already defined in the root filesystem of the image at /etc/passwd and /etc/group respectively, otherwise the build of the image would fail.

**The WORKDIR keyword**
The WORKDIR keyword defines the working directory or context that is used when a container is run from our custom image. So, if I want to set the context to the /app/bin folder inside the image, my expression in the Dockerfile would have to look as follows:
```
WORKDIR /app/bin
```
All activity that happens inside the image after the preceding line will use this directory as the working directory. It is very important to note that the following two snippets from a Dockerfile are not the same:
```
RUN cd /app/bin
RUN touch sample.txt
```
Compare the preceding code with the following code:
```
WORKDIR /app/bin
RUN touch sample.txt
```
The former will create the file in the root of the image filesystem, while the latter will create the file at the expected location in the /app/bin folder. Only the WORKDIR keyword sets the context across the layers of the image. The cd command alone is not persisted across layers.

**The CMD and ENTRYPOINT keywords**
While all other keywords defined for a Dockerfile are executed at the time the image is built by the Docker builder, these two are actually definitions of what will happen when a container is started from the image we define. When the container runtime starts a container, it needs to know what the process or application will be that has to run inside this container. That is exactly what CMD and ENTRYPOINT are used for—to tell Docker what the start process is and how to start that process.

```
$ ping 8.8.8.8 -c 3
```
ping is the command and 8.8.8.8 -c 3 are the parameters to this command.

ENTRYPOINT is used to define the command of the expression while CMD is used to define the parameters for the command. Thus, a Dockerfile using alpine as the base image and defining ping as the process to run in the container could look as follows:
```
FROM alpine:latest
ENTRYPOINT ["ping"]
CMD ["8.8.8.8", "-c", "3"]
```
For both ENTRYPOINT and CMD, the values are formatted as a JSON array of strings, where the individual items correspond to the tokens of the expression that are separated by whitespace. This the preferred way of defining CMD and ENTRYPOINT. It is also called the exec form.
```
$ docker image build -t pinger .
$ docker container run --rm -it pinger
PING 8.8.8.8 (8.8.8.8): 56 data bytes
64 bytes from 8.8.8.8: seq=0 ttl=56 time=5.612 ms
64 bytes from 8.8.8.8: seq=1 ttl=56 time=4.389 ms
64 bytes from 8.8.8.8: seq=2 ttl=56 time=6.592 ms
#The beauty of this is that I can now override the CMD part that I have defined in the Dockerfile
#This will now cause the container to ping the loopback for 5 seconds.

$ docker container run --rm -it pinger -w 5 127.0.0.1
PING 127.0.0.1 (127.0.0.1): 56 data bytes
64 bytes from 127.0.0.1: seq=0 ttl=64 time=0.077 ms
64 bytes from 127.0.0.1: seq=1 ttl=64 time=0.087 ms
64 bytes from 127.0.0.1: seq=2 ttl=64 time=0.090 ms
64 bytes from 127.0.0.1: seq=3 ttl=64 time=0.082 ms
64 bytes from 127.0.0.1: seq=4 ttl=64 time=0.081 ms

#If we want to override what's defined in the ENTRYPOINT in the Dockerfile, we need to use the --entrypoint parameter in the docker container run expression.
docker container run --rm -it --entrypoint /bin/sh pinger
/ # exit

FROM alpine:latest
CMD wget -O - http://www.google.com

## If you leave ENTRYPOINT undefined, then it will have the default value of /bin/sh -c, and whatever is the value of CMD will be passed as a string to the shell command. The preceding definition would thereby result in entering following process to run inside the container:
/bin/sh -c "wget -O - http://www.google.com"

Consequently, /bin/sh is the main process running inside the container, and it will start a new child process to run the wget utility.
```
![the image build process visualized](https://i.imgur.com/98sfbpv.jpg)

## Multistep builds

With a size of 176 MB, the resulting image is way too big. In the end, it is just a Hello World application. The reason for it being so big is that the image not only contains the Hello World binary, but also all the tools to compile and link the application from the source code. But this is really not desirable when running the application, say, in production. Ideally, we only want to have the resulting binary in the image and not a whole SDK.

It is precisely for this reason that we should define Dockerfiles as multistage. We have some stages that are used to build the final artifacts and then a final stage where we use the minimal necessary base image and copy the artifacts into it. This results in very small images. Have a look at this revised Dockerfile:

```
FROM alpine:3.7 AS build
RUN apk update && apk add --update alpine-sdk
RUN mkdir /app
WORKDIR /app
COPY . /app
RUN mkdir bin
RUN gcc hello.c -o bin/hello

FROM alpine:3.7
COPY --from=build /app/bin/hello /app/hello
CMD /app/hello
```
Here, we have a first stage with an alias build that is used to compile the application, and then the second stage uses the same base image alpine:3.7, but does not install the SDK, and only copies the binary from the build stage, using the --from parameter, into this final image.
```
docker image ls |grep hello-world
hello-world-small                                              latest                                     46bb8c275fda        About a minute ago   4.22MB
hello-world                                                    latest                                     75339d9c269f        7 minutes ago        178MB
```
We have been able to reduce the size from 178 MB down to 4 MB. This is reduction in size by a factor of 40. A smaller image size has many advantages, such as a smaller attack surface area for hackers, reduced memory and disk consumption, faster startup times of the corresponding containers, and a reduction of the bandwidth needed to download the image from a registry, such as Docker Hub.

## Dockerfile best practices
- First and foremost, we need to consider that containers are meant to be ephemeral. By ephemeral, we mean that a container can be stopped and destroyed and a new one built and put in place with an absolute minimum of setup and configuration. That means that we should try hard to keep the time that is needed to initialize the application running inside the container at a minimum, as well as the time needed to terminate or clean up the application.
- we should order the individual commands in the Dockerfile so that we leverage caching as much as possible. Building a layer of an image can take a considerable amount of time, sometimes many seconds or even minutes. While developing an application, we will have to build the container image for our application multiple times. We want to keep the build times at a minimum.

> When we're rebuilding a previously built image, the only layers that are rebuilt are the ones that have changed, but if one layer needs to be rebuilt, all subsequent layers also need to be rebuilt. This is very important to remember. Consider the following example:
```
FROM node:9.4
RUN mkdir -p /app
WORKDIR /app
COPY . /app
RUN npm install
CMD ["npm", "start"]
```
In this example, the npm install command on line five of the Dockerfile usually takes the longest. A classical Node.js application has many external dependencies, and those are all downloaded and installed in this step. This can take minutes until it is done. Therefore, we want to avoid running npm install each time we rebuild the image, but a developer changes their source code all the time during development of the application. That means that line four, the result of the COPY command, changes all the time and this layer has to be rebuilt each time. But as we discussed previously, that also means that all subsequent layers have to be rebuilt, which in this case includes the npm install command. To avoid this, we can slightly modify the Dockerfile and have the following:
```
FROM node:9.4
RUN mkdir -p /app
WORKDIR /app
COPY package.json /app/
RUN npm install
COPY . /app
CMD ["npm", "start"]
```
What we have done here is that, on line four, we only copy the single file that the npm install command needs as a source, which is the package.json file. This file rarely changes in a typical development process. As a consequence, the npm install command also has to be executed only when the package.json file changes. All the remaining, frequently changed content is added to the image after the npm install command.

- A further best practice is to keep the number of layers that make up your image relatively small. The more layers an image has, the more the graph driver needs to work to consolidate the layers into a single root filesystem for the corresponding container. Of course, this takes time, and thus the fewer layers an image has, the faster the startup time for the container can be.
> The easiest way to reduce the number of layers is to combine multiple individual RUN commands into a single one—for example, say that we had the following in a Dockerfile:

```
RUN apt-get update
RUN apt-get install -y ca-certificates
RUN rm -rf /var/lib/apt/lists/*
```
We could combine these into a single concatenated expression, as follows:
```
RUN apt-get update \
   && apt-get install -y ca-certificates \
   && rm -rf /var/lib/apt/lists/*
```
The former will generate three layers in the resulting image, while the latter only creates a single layer.

The next three best practices all result in smaller images. Why is this important? Smaller images reduce the time and bandwidth needed to download the image from a registry. They also reduce the amount of disk space needed to store a copy locally on the Docker host and the memory needed to load the image. Finally, smaller images also means a smaller attack surface for hackers. Here are the best practices mentioned:

1. The first best practice that helps to reduce the image size is to use a .dockerignore file. We want to avoid copying unnecessary files and folders into an image to keep it as lean as possible. A .dockerignore file works in exactly the same way as a .gitignore file, for those who are familiar with Git. In a .dockerignore file, we can configure patterns to exclude certain files or folders from being included in the context when building the image.

2. The next best practice is to avoid installing unnecessary packages into the filesystem of the image. Once again, this is to keep the image as lean as possible.

3. Last but not least, it is recommended that you use multistage builds so that the resulting image is as small as possible and only contains the absolute minimum needed to run your application or application service. 

# 3. Saving and loading images
The third way to create a new container image is by importing or loading it from a file. A container image is nothing more than a tarball. To demonstrate this, we can use the docker image save command to export an existing image to a tarball:
```
$ docker image save -o ./backup/my-alpine.tar my-alpine
$ docker image load -i ./backup/my-alpine.tar
```
# Running in production
**Blue-green deployments**
In blue-green deployments, the current version of the application service, called blue, handles all the application traffic. We then install the new version of the application service, called green, on the production system. The new service is not yet wired with the rest of the application.

Once green is installed, one can execute smoke tests against this new service and, if those succeed, the router can be configured to funnel all traffic that previously went to blue to the new service, green. The behavior of green is then observed closely and, if all success criteria are met, blue can be decommissioned. But if, for some reason, green shows some unexpected or unwanted behavior, the router can be reconfigured to return all traffic to blue. Green can then be removed and fixed, and a new blue-green deployment can be executed with the corrected version:
![blue-green deployment](https://i.imgur.com/bWC8uLg.png)

**Canary releases**
Canary releases are releases where we have the current version of the application service and the new version installed on the system in parallel. As such, they resemble blue-green deployments. At first, all traffic is still routed through the current version. We then configure a router so that it funnels a small percentage, say 1%, of the overall traffic to the new version of the application service. The behavior of the new service is then monitored closely to find out whether or not it works as expected. If all the criteria for success are met, then the router is configured to funnel more traffic, say 5% this time, through the new service. Again, the behavior of the new service is closely monitored and, if it is successful, more and more traffic is routed to it until we reach 100%. Once all traffic is routed to the new service and it has been stable for some time, the old version of the service can be decommissioned.
# System management
## Pruning unused resources
**Listing resource consumption**
$ docker system df
# in verbose mode 
$ docker system df -v
**Pruning containers**
# regain unused system resources
$ docker container prune # remove all containers from the system that are not in running status
$ docker container prune -f # skip confirmation step
# remove all containers from system, even the running ones
$ docker container rm -f $(docker container ls -aq)
**Pruning images**
$ docker image prune
# avoid Docker asking for a confirmation
$ docker image prune -f

#  Sometimes we do not just want to remove orphaned image layers but all images that are not currently in use on our system. For this, we can use the -a (or --all) flag:
$ docker image prune --force --all  
**Pruning volumes**
$ docker volume prune 
A useful flag when pruning volumes is the -f or --filter flag which allows us to specify the set of volumes which we're considering for pruning
$ docker volume prune --filter 'label=demo' 
**Pruning network**
will remove the networks on which currently no container or service is attached. 
$ docker network prune
**Pruning everything**
$ docker system prune
