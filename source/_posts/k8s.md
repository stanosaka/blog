---
title: k8s
date: 2019-06-04 11:37:00
tags: kubernetes
---
# K8S

Big example is google

orchestration (computing)
automcatic management of them

manifests system robust 

common question why use k8s instead of docker swarm

number 1 ad: docker swarm is built in
K8S: far richer, deploy do different cloud platforms
K8S is just far more popular 
is a orchestration of choice.
K8S is the most in demand container orchestration system out there.


A Java-based with angular on the front-end.
# Terminology
- Cluster: A group of physical or virtual machines
- Node: Ap hysical or virtual machine that is part of a cluster
- Control Plane: The set of processes that run the core k8s services (e.g. APi, scheduler, etcd ...)
- Pod: The lowest compute unit in Kubernetes, a group of containers that run on a Node.

# Architecture
**Head Node**
That's the brain of Kubernetes
- API server
- Scheduler: to place the containers where they need to go
- Controller manager: makes sure that the state of the system is what it should be
- Etcd: data store, used to store the state of the system
- Sometimes:
  - kubelet: a process manage all of this
  - docker: container engine
**Worker node**
- kubelet
That's the Kubernetes agent that runs on all the Kubernetes cluster nodes. Kubelet talks with the Kubernetes API server and then talks to the local Docker daemon to be able to manage the Docker containers running on the node.
- kube-proxy: a system that allos you to manage the IP tables in that node so that the traffic between the pods and the nodes is what it should be
- docker
# Installing k8s in 3 easy ways
- minikube
- gcloud container clusters
- kubeadm
# Installing Minikube
cut-down version-minikube
https://kubernetes.io/docs/tasks/tools/install-minikube/

You might have an incompatibility between your distribution and the one that Minikube is expecting.
two command line tools: kubectl is the controller pogramme for k8s. and Minikube

Enable the Hyper-V role throught settings
when enable don't use oracle virtual box

goo.gl/4yEFbF    for win 10 Professional

minikube start hangs forever on mac #2765

If you're completely stuck then do ask me

# Docker Overview

Difference between Docker Images and Container:
A container is an instance of docker images.
Docker container is the run time instance of images.
```
docker image pull richardchesterwood/k8s-fleetman-webapp-angular:release0-5
docker image ls
docker container run -p 8080:80 -d richardchesterwood/k8s-fleetman-webapp-angular:release0-5
docker container ls
minikube ip
docker container stop 2c5
docker container rm 2c5
```
# Pods

A pod is a group of one or more containers, with shared storage/network, and a specification for how to run the containers.

basic concept is Pod. A pod and a container are in a one to one relationship.

## writing a Pod

`kubectl get all` show everything we have defined in our Kubernetes cluster.

`kubectl apply -f first-pod.yaml`

`kubectl describe pod webapp`

`kubectl exec webapp ls`

`kubectl -it exec webapp sh` it means get in interactively with teletype emulation. interactive

## Services

Pods are not visible outside the cluster
Pods are designed to be very throw away things. Pods have short lifetimes. Pods regularly die.

Service has stable port, with a service we can connect to kubernetes cluster.

key vaule pairs app with some value.

![Services](https://i.imgur.com/Vv0KFxA.png)

## NodePort and ClusterIP
ClusterIP

NodePort

## Pod selection with Labels

![Pod secltion with Labels](https://i.imgur.com/nEb2kOH.png)

```
cat first-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp
  labels:
    app: webapp
    release: "0"
spec:
  containers:
  - name: webapp
    image: richardchesterwood/k8s-fleetman-webapp-angular:release0


---
apiVersion: v1
kind: Pod
metadata:
  name: webapp-release-0-5
  labels:
    app: webapp
    release: "0-5"
spec:
  containers:
  - name: webapp
    image: richardchesterwood/k8s-fleetman-webapp-angular:release0-5
```
```
cat webapp-service.yaml
apiVersion: v1
kind: Service
metadata:
 name: fleetman-webapp

spec:
 # This defines which pods are going to be represented by this Service
 # The service becomes a network endpoint for either other services
 #  or maybe external users to connect to (eg browser)
 selector:
   app: webapp
   release: "0-5"

 ports:
   - name: http
     port: 80
     nodePort: 30080

 type: NodePort
```

```
kubectl apply -f webapp-service.yaml

kubectl describe svc fleetman-webapp
Name:                     fleetman-webapp
Namespace:                default
Labels:                   <none>
Annotations:              kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"name":"fleetman-webapp","namespace":"default"},"spec":{"ports":[{"name":"http","nodeP...
Selector:                 app=webapp,release=0
Type:                     NodePort
IP:                       10.108.217.186
Port:                     http  80/TCP
TargetPort:               80/TCP
NodePort:                 http  30080/TCP
Endpoints:                172.17.0.5:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>

kubectl get po
NAME                 READY     STATUS    RESTARTS   AGE
webapp               1/1       Running   2          2h
webapp-release-0-5   1/1       Running   0          8m

kubectl get po --show-labels
NAME                 READY     STATUS    RESTARTS   AGE       LABELS
webapp               1/1       Running   2          2h        app=webapp,release=0
webapp-release-0-5   1/1       Running   0          8m        app=webapp,release=0-5

kubectl get po --show-labels -l release=0
NAME      READY     STATUS    RESTARTS   AGE       LABELS
webapp    1/1       Running   2          2h        app=webapp,release=0

kubectl get po --show-labels -l release=1
No resources found.
```
# REPLICASETS

## ReplicaSets

![ReplicaSets](https://i.imgur.com/iUSOtXN.png)
When Pod die, it will never come back.

```
kubectl get all

kubectl describe svc fleetman-webapp

kubectl delete po webapp-release-0-5
```

ReplicaSets specify how many instances of this pod do we want k8s running on time

## Writing a ReplicaSet
[ReplicaSet v1 apps](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.14/#replicaset-v1-apps)
```
cat pods.yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: webapp
spec:
  selector:
    matchLabels:
      app: webapp
  replicas: 1
  template: # template for the pods
    metadata:
      labels:
        app: webapp
    spec:
      containers:
      - name: webapp
        image: richardchesterwood/k8s-fleetman-webapp-angular:release0-5

---
apiVersion: v1
kind: Pod
metadata:
  name: queue
  labels:
    app: queue
spec:
  containers:
  - name: queue
    image: richardchesterwood/k8s-fleetman-queue:release1
```

### Applying a ReplicaSet

Delete all the Pods:

```
kubectl delete po --all
```

```
kubectl apply -f pods.yaml
replicaset.apps "webapp" created
pod "queue" created

kubectl get all
NAME               READY     STATUS    RESTARTS   AGE
pod/queue          1/1       Running   0          58s
pod/webapp-hzpcp   1/1       Running   0          58s

NAME                      TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
service/fleetman-queue    NodePort    10.106.99.143    <none>        8161:30010/TCP   3h
service/fleetman-webapp   NodePort    10.108.217.186   <none>        80:30080/TCP     23h
service/kubernetes        ClusterIP   10.96.0.1        <none>        443/TCP          1d

NAME                     DESIRED   CURRENT   READY     AGE
replicaset.apps/webapp   1         1         1         59s
```
The difference between current and ready is, current is the number of containers that are running, and ready is the number of containers that are responding to requests.

```
kubectl describe rs webapp
Name:         webapp
Namespace:    default
Selector:     app=webapp
Labels:       app=webapp
Annotations:  kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"apps/v1","kind":"ReplicaSet","metadata":{"annotations":{},"name":"webapp","namespace":"default"},"spec":{"replicas":1,"selector":{"match...
Replicas:     1 current / 1 desired
Pods Status:  1 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  app=webapp
  Containers:
   webapp:
    Image:        richardchesterwood/k8s-fleetman-webapp-angular:release0-5
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type    Reason            Age   From                   Message
  ----    ------            ----  ----                   -------
  Normal  SuccessfulCreate  4m    replicaset-controller  Created pod: webapp-hzpcp
```

```
kubectl get all
NAME               READY     STATUS    RESTARTS   AGE
pod/queue          1/1       Running   0          5m
pod/webapp-hzpcp   1/1       Running   0          5m

NAME                      TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
service/fleetman-queue    NodePort    10.106.99.143    <none>        8161:30010/TCP   3h
service/fleetman-webapp   NodePort    10.108.217.186   <none>        80:30080/TCP     23h
service/kubernetes        ClusterIP   10.96.0.1        <none>        443/TCP          1d

NAME                     DESIRED   CURRENT   READY     AGE
replicaset.apps/webapp   1         1         1         5m

kubectl delete po webapp-hzpcp
pod "webapp-hzpcp" deleted

kubectl get all
NAME               READY     STATUS        RESTARTS   AGE
pod/queue          1/1       Running       0          7m
pod/webapp-hff5l   1/1       Running       0          3s
pod/webapp-hzpcp   0/1       Terminating   0          7m

NAME                      TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
service/fleetman-queue    NodePort    10.106.99.143    <none>        8161:30010/TCP   3h
service/fleetman-webapp   NodePort    10.108.217.186   <none>        80:30080/TCP     23h
service/kubernetes        ClusterIP   10.96.0.1        <none>        443/TCP          1d

NAME                     DESIRED   CURRENT   READY     AGE
replicaset.apps/webapp   1         1         1         7m
```

# Deployments

It's a replica set with one additional feature. With a deployment, we get automatic rolling updates with zero downtime.

[Deployment API reference guide](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.14/#deployment-v1-apps)

A deployment as being an entity in Kubernetes that manages the replica set for you.

```
cat pods.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
spec:
  selector:
    matchLabels:
      app: webapp
  replicas: 2
  template: # template for the pods
    metadata:
      labels:
        app: webapp
    spec:
      containers:
      - name: webapp
        image: richardchesterwood/k8s-fleetman-webapp-angular:release0

---
apiVersion: v1
kind: Pod
metadata:
  name: queue
  labels:
    app: queue
spec:
  containers:
  - name: queue
    image: richardchesterwood/k8s-fleetman-queue:release1

kubectl apply -f pods.yaml
deployment.apps "webapp" created
pod "queue" unchanged

kubectl get all
NAME                          READY     STATUS    RESTARTS   AGE
pod/queue                     1/1       Running   0          24m
pod/webapp-7469fb7fd6-4mcth   1/1       Running   0          12s
pod/webapp-7469fb7fd6-sv4rw   1/1       Running   0          12s

NAME                      TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
service/fleetman-queue    NodePort    10.106.99.143    <none>        8161:30010/TCP   3h
service/fleetman-webapp   NodePort    10.108.217.186   <none>        80:30080/TCP     23h
service/kubernetes        ClusterIP   10.96.0.1        <none>        443/TCP          1d

NAME                     DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/webapp   2         2         2            2           12s

NAME                                DESIRED   CURRENT   READY     AGE
replicaset.apps/webapp-7469fb7fd6   2         2         2         12s
```

## Managing Rollouts

```
kubectl rollout status deploy webapp
deployment "webapp" successfully rolled out


kubectl rollout status deploy webapp
Waiting for rollout to finish: 1 out of 2 new replicas have been updated...
Waiting for rollout to finish: 1 out of 2 new replicas have been updated...
Waiting for rollout to finish: 2 old replicas are pending termination...
Waiting for rollout to finish: 1 old replicas are pending termination...
Waiting for rollout to finish: 1 old replicas are pending termination...
Waiting for rollout to finish: 1 old replicas are pending termination...
deployment "webapp" successfully rolled out

kubectl rollout history deploy webapp
deployments "webapp"
REVISION  CHANGE-CAUSE
2         <none>
3         <none>

# Default command automatically go back to one version

kubectl rollout undo deploy

```
It's really only for use in an emergency.

# Networking and service discovery

## Networking Overview

![Networking Overview](https://i.imgur.com/G9GkqlS.png)

## Namespace-kube-system

A namespace is a way of partitioning your resources in Kubernetes into separate areas.

![namespace](https://i.imgur.com/9FRKlUT.png) 

![namespace](https://i.imgur.com/9QF8vkE.png)

```
kubectl get ns
NAME          STATUS    AGE
default       Active    1d
kube-public   Active    1d
kube-system   Active    1d

kubectl get po
NAME                      READY     STATUS    RESTARTS   AGE
queue                     1/1       Running   0          3h
webapp-7469fb7fd6-sg87f   1/1       Running   0          1h
webapp-7469fb7fd6-znbxx   1/1       Running   0          1h

kubectl get po -n kube-system
NAME                                    READY     STATUS    RESTARTS   AGE
coredns-576cbf47c7-kwq7n                1/1       Running   0          1d
coredns-576cbf47c7-vt56b                1/1       Running   0          1d
etcd-minikube                           1/1       Running   0          1d
kube-addon-manager-minikube             1/1       Running   0          1d
kube-apiserver-minikube                 1/1       Running   0          1d
kube-controller-manager-minikube        1/1       Running   0          1d
kube-proxy-cn982                        1/1       Running   0          1d
kube-scheduler-minikube                 1/1       Running   0          1d
kubernetes-dashboard-5bff5f8fb8-mqz9z   1/1       Running   0          1d
storage-provisioner                     1/1       Running   0          1d

kubectl get all -n kube-system
NAME                                        READY     STATUS    RESTARTS   AGE
pod/coredns-576cbf47c7-kwq7n                1/1       Running   0          1d
pod/coredns-576cbf47c7-vt56b                1/1       Running   0          1d
pod/etcd-minikube                           1/1       Running   0          1d
pod/kube-addon-manager-minikube             1/1       Running   0          1d
pod/kube-apiserver-minikube                 1/1       Running   0          1d
pod/kube-controller-manager-minikube        1/1       Running   0          1d
pod/kube-proxy-cn982                        1/1       Running   0          1d
pod/kube-scheduler-minikube                 1/1       Running   0          1d
pod/kubernetes-dashboard-5bff5f8fb8-mqz9z   1/1       Running   0          1d
pod/storage-provisioner                     1/1       Running   0          1d

NAME                           TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)         AGE
service/kube-dns               ClusterIP   10.96.0.10     <none>        53/UDP,53/TCP   1d
service/kubernetes-dashboard   ClusterIP   10.102.124.0   <none>        80/TCP          1d

NAME                        DESIRED   CURRENT   READY     UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
daemonset.apps/kube-proxy   1         1         1         1            1           <none>          1d

NAME                                   DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/coredns                2         2         2            2           1d
deployment.apps/kubernetes-dashboard   1         1         1            1           1d

NAME                                              DESIRED   CURRENT   READY     AGE
replicaset.apps/coredns-576cbf47c7                2         2         2         1d
replicaset.apps/kubernetes-dashboard-5bff5f8fb8   1         1         1         1d

kubectl describe svc kube-dns -n kube-system
Name:              kube-dns
Namespace:         kube-system
Labels:            k8s-app=kube-dns
                   kubernetes.io/cluster-service=true
                   kubernetes.io/name=KubeDNS
Annotations:       prometheus.io/port=9153
                   prometheus.io/scrape=true
Selector:          k8s-app=kube-dns
Type:              ClusterIP
IP:                10.96.0.10
Port:              dns  53/UDP
TargetPort:        53/UDP
Endpoints:         172.17.0.2:53,172.17.0.3:53
Port:              dns-tcp  53/TCP
TargetPort:        53/TCP
Endpoints:         172.17.0.2:53,172.17.0.3:53
Session Affinity:  None
Events:            <none>
```
## Accessing MySQL from a Pod

```
cat networking-tests.yaml
apiVersion: v1
kind: Pod
metadata:
  name: mysql
  labels:
     app: mysql
spec:
  containers:
   - name: mysql
     image: mysql:5
     env:
      # Use secret in real life
      - name: MYSQL_ROOT_PASSWORD
        value: password
      - name: MYSQL_DATABASE
        value: fleetman

---
kind: Service
apiVersion: v1
metadata:
  name: database
spec:
  selector:
     app: mysql
  ports:
  - port: 3306
  type: ClusterIP

kga
NAME                          READY     STATUS    RESTARTS   AGE
pod/mysql                     1/1       Running   0          3m
pod/queue                     1/1       Running   0          18h
pod/webapp-7469fb7fd6-sg87f   1/1       Running   0          17h
pod/webapp-7469fb7fd6-znbxx   1/1       Running   0          17h

NAME                      TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
service/database          ClusterIP   10.101.3.159     <none>        3306/TCP         3m
service/fleetman-queue    NodePort    10.106.99.143    <none>        8161:30010/TCP   21h
service/fleetman-webapp   NodePort    10.108.217.186   <none>        80:30080/TCP     1d
service/kubernetes        ClusterIP   10.96.0.1        <none>        443/TCP          1d

NAME                     DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/webapp   2         2         2            2           17h

NAME                                DESIRED   CURRENT   READY     AGE
replicaset.apps/webapp-7469fb7fd6   2         2         2         17h
replicaset.apps/webapp-74bd9697b4   0         0         0         17h
replicaset.apps/webapp-8f948b66c    0         0         0         17h

kubectl exec -it webapp-7469fb7fd6-sg87f sh
/ # ls
bin    etc    lib    mnt    root   sbin   sys    usr
dev    home   media  proc   run    srv    tmp    var
```

## Service Discovery

```
/ # cat /etc/resolv.conf
nameserver 10.96.0.10
search default.svc.cluster.local svc.cluster.local cluster.local
options ndots:5

/ # nslookup database
nslookup: can't resolve '(null)': Name does not resolve

Name:      database
Address 1: 10.101.3.159 database.default.svc.cluster.local

kubectl exec -it webapp-7469fb7fd6-sg87f sh
/ # mysql
sh: mysql: not found
/ # apk update
fetch http://dl-cdn.alpinelinux.org/alpine/v3.7/main/x86_64/APKINDEX.tar.gz
fetch http://dl-cdn.alpinelinux.org/alpine/v3.7/community/x86_64/APKINDEX.tar.gz
v3.7.3-61-gd3d301001c [http://dl-cdn.alpinelinux.org/alpine/v3.7/main]
v3.7.3-51-gf95d10fed7 [http://dl-cdn.alpinelinux.org/alpine/v3.7/community]
OK: 9067 distinct packages available
/ # apk add mysql-client
(1/6) Installing mariadb-common (10.1.38-r1)
(2/6) Installing ncurses-terminfo-base (6.0_p20171125-r1)
(3/6) Installing ncurses-terminfo (6.0_p20171125-r1)
(4/6) Installing ncurses-libs (6.0_p20171125-r1)
(5/6) Installing mariadb-client (10.1.38-r1)
(6/6) Installing mysql-client (10.1.38-r1)
Executing busybox-1.27.2-r7.trigger
OK: 53 MiB in 34 packages

# mysql -h database -uroot -ppassword fleetman
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MySQL connection id is 3
Server version: 5.7.26 MySQL Community Server (GPL)

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MySQL [fleetman]> create table testable (test varchar(255));
Query OK, 0 rows affected (0.05 sec)

MySQL [fleetman]> show tables;
+--------------------+
| Tables_in_fleetman |
+--------------------+
| testable           |
+--------------------+
1 row in set (0.01 sec)
```

Can find the ip address of any service that we like just by its name. And that's called service discovery.

## Fully Qualified Domain Names (FQDN)

```
# nslookup database
nslookup: can't resolve '(null)': Name does not resolve

Name:      database
Address 1: 10.101.3.159 database.default.svc.cluster.local
```

## An introduction to Microservices

Each microservice should be  Highly Cohesive and Lossely Coupled.

highly cohesive: each microservice should handlng one business requirement. Cohesive means that a microservice should have a single set of reponsibilities.

Each microservice will maintain its own data store. And that microservice will be really the only poart of the system that can read or write that data.

## Fleetman Microservices- setting the scene

![scene](https://i.imgur.com/fkwUIA0.png)
The logic in the API gateway is typically some kind of a mapping. So it would be something like if the incoming request ends with /vehicles, then delegate the call to,
in this case, the position tracker.

[API gateway pattern](https://microservices.io/patterns/apigateway.html)

## Deploying the Queue

![scene](https://i.imgur.com/fkwUIA0.png)

API gateway: a web front end which is implemented in Java script

Position tracker: back end, calcuating the speeds of vehicles and storing the positions of all the vehicles.

Queue: which is going to store the messages that are received from the vehicles as they move around the country.

Positon simulator: a testing microservice which is going to generate some positions of vehicles.

Delete all the Pods: 
```
kubectl delete -f .
pod "mysql" deleted
service "database" deleted
deployment.apps "webapp" deleted
pod "queue" deleted
service "fleetman-webapp" deleted
service "fleetman-queue" deleted
```

```
kubectl apply -f workloads.yaml
kubectl describe pod queue-6bf9fd876-4xrqx
Name:               queue-6bf9fd876-4xrqx
Namespace:          default
Priority:           0
PriorityClassName:  <none>
Node:               minikube/10.0.2.15
Start Time:         Wed, 05 Jun 2019 15:14:40 +1000
Labels:             app=queue
                    pod-template-hash=6bf9fd876
Annotations:        <none>
Status:             Running
IP:                 172.17.0.5
Controlled By:      ReplicaSet/queue-6bf9fd876
Containers:
  webapp:
    Container ID:   docker://f306dda43b8bf2eaab70449ecac022bb0b98e37b8b2be0d1b2e1b25eea9db1ac
    Image:          richardchesterwood/k8s-fleetman-queue:release1
    Image ID:       docker-pullable://richardchesterwood/k8s-fleetman-queue@sha256:bc2cb90a09aecdd8bce5d5f3a8dac17281ec7883077ddcfb8b7acfe2ab3b6afa
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Wed, 05 Jun 2019 15:14:41 +1000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-nmqkz (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  default-token-nmqkz:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-nmqkz
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  2m    default-scheduler  Successfully assigned default/queue-6bf9fd876-4xrqx to minikube
  Normal  Pulled     2m    kubelet, minikube  Container image "richardchesterwood/k8s-fleetman-queue:release1" already present on machine
  Normal  Created    2m    kubelet, minikube  Created container
  Normal  Started    2m    kubelet, minikube  Started container

kubectl apply -f services.yaml
service "fleetman-webapp" created
service "fleetman-queue" created

kubectl apply -f services.yaml
service "fleetman-webapp" created
service "fleetman-queue" created

minikube ip
There is a newer version of minikube available (v1.1.0).  Download it here:
https://github.com/kubernetes/minikube/releases/tag/v1.1.0

To disable this notification, run the following:
minikube config set WantUpdateNotification false
192.168.99.118
```

Open a web browser http://192.168.99.118:30010, click **Manage ActiveMQ broker**, enter username and password both admin.

## Deploying the Position Simulator

```
cat workloads.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: queue
spec:
  selector:
    matchLabels:
      app: queue
  replicas: 1
  template: # template for the pods
    metadata:
      labels:
        app: queue
    spec:
      containers:
      - name: webapp
        image: richardchesterwood/k8s-fleetman-queue:release1

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: position-simulator
spec:
  selector:
    matchLabels:
      app: position-simulator
  replicas: 1
  template: # template for the pods
    metadata:
      labels:
        app: position-simulator
    spec:
      containers:
      - name: webapp
        image: richardchesterwood/k8s-fleetman-position-simulator:release1
        env:
        - name: SPRING_PROFILES_ACTIVE
          value: producadskfjsjfsislfslsj

kubectl apply -f workloads.yaml
deployment.apps "queue" unchanged
deployment.apps "position-simulator" configured

kga
NAME                                      READY     STATUS              RESTARTS   AGE
pod/position-simulator-6f97fd485f-gplr8   0/1       ContainerCreating   0          7s
pod/position-simulator-dff7b7599-2vbdd    0/1       ImagePullBackOff    0          2m
pod/queue-6bf9fd876-4xrqx                 1/1       Running             0          50m

NAME                      TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                          AGE
service/fleetman-queue    NodePort    10.110.95.121    <none>        8161:30010/TCP,61616:30536/TCP   47m
service/fleetman-webapp   NodePort    10.103.224.156   <none>        80:30080/TCP                     47m
service/kubernetes        ClusterIP   10.96.0.1        <none>        443/TCP                          2d

NAME                                 DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/position-simulator   1         2         1            0           2m
deployment.apps/queue                1         1         1            1           50m

NAME                                            DESIRED   CURRENT   READY     AGE
replicaset.apps/position-simulator-6f97fd485f   1         1         0         7s
replicaset.apps/position-simulator-dff7b7599    1         1         0         2m
replicaset.apps/queue-6bf9fd876                 1         1         1         50m

kubectl describe pod position-simulator-6f97fd485f-gplr8
Name:               position-simulator-6f97fd485f-gplr8
Namespace:          default
Priority:           0
PriorityClassName:  <none>
Node:               minikube/10.0.2.15
Start Time:         Wed, 05 Jun 2019 16:05:27 +1000
Labels:             app=position-simulator
                    pod-template-hash=6f97fd485f
Annotations:        <none>
Status:             Running
IP:                 172.17.0.7
Controlled By:      ReplicaSet/position-simulator-6f97fd485f
Containers:
  webapp:
    Container ID:   docker://2a4f677af3a8bd72f674543e13a4b84c3c86148791106a19270d86878091b09d
    Image:          richardchesterwood/k8s-fleetman-position-simulator:release1
    Image ID:       docker-pullable://richardchesterwood/k8s-fleetman-position-simulator@sha256:58ce4aa156469115a58be0bcfcec84bb7e42dd4552f83dcf5b6381234001108b
    Port:           <none>
    Host Port:      <none>
    State:          Terminated
      Reason:       Error
      Exit Code:    1
      Started:      Wed, 05 Jun 2019 16:05:56 +1000
      Finished:     Wed, 05 Jun 2019 16:05:59 +1000
    Ready:          False
    Restart Count:  0
    Environment:
      SPRING_PROFILES_ACTIVE:  producadskfjsjfsislfslsj
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-nmqkz (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             False
  ContainersReady   False
  PodScheduled      True
Volumes:
  default-token-nmqkz:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-nmqkz
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age              From               Message
  ----    ------     ----             ----               -------
  Normal  Scheduled  33s              default-scheduler  Successfully assigned default/position-simulator-6f97fd485f-gplr8 to minikube
  Normal  Pulling    32s              kubelet, minikube  pulling image "richardchesterwood/k8s-fleetman-position-simulator:release1"
  Normal  Pulled     4s               kubelet, minikube  Successfully pulled image "richardchesterwood/k8s-fleetman-position-simulator:release1"
  Normal  Created    0s (x2 over 4s)  kubelet, minikube  Created container
  Normal  Started    0s (x2 over 4s)  kubelet, minikube  Started container
  Normal  Pulled     0s               kubelet, minikube  Container image "richardchesterwood/k8s-fleetman-position-simulator:release1" already present on machine
```

### inspecting Pod Logs
```
kubectl logs position-simulator-6f97fd485f-gplr8

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v1.4.0.RELEASE)

2019-06-05 06:09:06.044  INFO 1 --- [           main] c.v.s.PositionsimulatorApplication       : Starting PositionsimulatorApplication v0.0.1-SNAPSHOT on position-simulator-6f97fd485f-gplr8 with PID 1 (/webapp.jar started by root in /)
2019-06-05 06:09:06.056  INFO 1 --- [           main] c.v.s.PositionsimulatorApplication       : The following profiles are active: producadskfjsjfsislfslsj
2019-06-05 06:09:06.151  INFO 1 --- [           main] s.c.a.AnnotationConfigApplicationContext : Refreshing org.springframework.context.annotation.AnnotationConfigApplicationContext@5f4da5c3: startup date [Wed Jun 05 06:09:06 UTC 2019]; root of context hierarchy
2019-06-05 06:09:07.265  WARN 1 --- [           main] s.c.a.AnnotationConfigApplicationContext : Exception encountered during context initialization - cancelling refresh attempt: org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'journeySimulator': Injection of autowired dependencies failed; nested exception is java.lang.IllegalArgumentException: Could not resolve placeholder 'fleetman.position.queue' in string value "${fleetman.position.queue}"
2019-06-05 06:09:07.273  INFO 1 --- [           main] utoConfigurationReportLoggingInitializer :

Error starting ApplicationContext. To display the auto-configuration report enable debug logging (start with --debug)


2019-06-05 06:09:07.286 ERROR 1 --- [           main] o.s.boot.SpringApplication               : Application startup failed

org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'journeySimulator': Injection of autowired dependencies failed; nested exception is java.lang.IllegalArgumentException: Could not resolve placeholder 'fleetman.position.queue' in string value "${fleetman.position.queue}"
	at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor.postProcessPropertyValues(AutowiredAnnotationBeanPostProcessor.java:355) ~[spring-beans-4.3.2.RELEASE.jar!/:4.3.2.RELEASE]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.populateBean(AbstractAutowireCapableBeanFactory.java:1214) ~[spring-beans-4.3.2.RELEASE.jar!/:4.3.2.RELEASE]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:543) ~[spring-beans-4.3.2.RELEASE.jar!/:4.3.2.RELEASE]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:482) ~[spring-beans-4.3.2.RELEASE.jar!/:4.3.2.RELEASE]
	at org.springframework.beans.factory.support.AbstractBeanFactory$1.getObject(AbstractBeanFactory.java:306) ~[spring-beans-4.3.2.RELEASE.jar!/:4.3.2.RELEASE]
	at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:230) ~[spring-beans-4.3.2.RELEASE.jar!/:4.3.2.RELEASE]
	at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:302) ~[spring-beans-4.3.2.RELEASE.jar!/:4.3.2.RELEASE]
	at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:197) ~[spring-beans-4.3.2.RELEASE.jar!/:4.3.2.RELEASE]
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.preInstantiateSingletons(DefaultListableBeanFactory.java:776) ~[spring-beans-4.3.2.RELEASE.jar!/:4.3.2.RELEASE]
	at org.springframework.context.support.AbstractApplicationContext.finishBeanFactoryInitialization(AbstractApplicationContext.java:861) ~[spring-context-4.3.2.RELEASE.jar!/:4.3.2.RELEASE]
	at org.springframework.context.support.AbstractApplicationContext.refresh(AbstractApplicationContext.java:541) ~[spring-context-4.3.2.RELEASE.jar!/:4.3.2.RELEASE]
	at org.springframework.boot.SpringApplication.refresh(SpringApplication.java:759) [spring-boot-1.4.0.RELEASE.jar!/:1.4.0.RELEASE]
	at org.springframework.boot.SpringApplication.refreshContext(SpringApplication.java:369) [spring-boot-1.4.0.RELEASE.jar!/:1.4.0.RELEASE]
	at org.springframework.boot.SpringApplication.run(SpringApplication.java:313) [spring-boot-1.4.0.RELEASE.jar!/:1.4.0.RELEASE]
	at org.springframework.boot.SpringApplication.run(SpringApplication.java:1185) [spring-boot-1.4.0.RELEASE.jar!/:1.4.0.RELEASE]
	at org.springframework.boot.SpringApplication.run(SpringApplication.java:1174) [spring-boot-1.4.0.RELEASE.jar!/:1.4.0.RELEASE]
	at com.virtualpairprogrammers.simulator.PositionsimulatorApplication.main(PositionsimulatorApplication.java:28) [classes!/:0.0.1-SNAPSHOT]
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method) ~[na:1.8.0_131]
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62) ~[na:1.8.0_131]
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43) ~[na:1.8.0_131]
	at java.lang.reflect.Method.invoke(Method.java:498) ~[na:1.8.0_131]
	at org.springframework.boot.loader.MainMethodRunner.run(MainMethodRunner.java:48) [webapp.jar:0.0.1-SNAPSHOT]
	at org.springframework.boot.loader.Launcher.launch(Launcher.java:87) [webapp.jar:0.0.1-SNAPSHOT]
	at org.springframework.boot.loader.Launcher.launch(Launcher.java:50) [webapp.jar:0.0.1-SNAPSHOT]
	at org.springframework.boot.loader.JarLauncher.main(JarLauncher.java:58) [webapp.jar:0.0.1-SNAPSHOT]
Caused by: java.lang.IllegalArgumentException: Could not resolve placeholder 'fleetman.position.queue' in string value "${fleetman.position.queue}"
	at org.springframework.util.PropertyPlaceholderHelper.parseStringValue(PropertyPlaceholderHelper.java:174) ~[spring-core-4.3.2.RELEASE.jar!/:4.3.2.RELEASE]
	at org.springframework.util.PropertyPlaceholderHelper.replacePlaceholders(PropertyPlaceholderHelper.java:126) ~[spring-core-4.3.2.RELEASE.jar!/:4.3.2.RELEASE]
	at org.springframework.core.env.AbstractPropertyResolver.doResolvePlaceholders(AbstractPropertyResolver.java:219) ~[spring-core-4.3.2.RELEASE.jar!/:4.3.2.RELEASE]
	at org.springframework.core.env.AbstractPropertyResolver.resolveRequiredPlaceholders(AbstractPropertyResolver.java:193) ~[spring-core-4.3.2.RELEASE.jar!/:4.3.2.RELEASE]
	at org.springframework.context.support.PropertySourcesPlaceholderConfigurer$2.resolveStringValue(PropertySourcesPlaceholderConfigurer.java:172) ~[spring-context-4.3.2.RELEASE.jar!/:4.3.2.RELEASE]
	at org.springframework.beans.factory.support.AbstractBeanFactory.resolveEmbeddedValue(AbstractBeanFactory.java:813) ~[spring-beans-4.3.2.RELEASE.jar!/:4.3.2.RELEASE]
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.doResolveDependency(DefaultListableBeanFactory.java:1039) ~[spring-beans-4.3.2.RELEASE.jar!/:4.3.2.RELEASE]
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.resolveDependency(DefaultListableBeanFactory.java:1019) ~[spring-beans-4.3.2.RELEASE.jar!/:4.3.2.RELEASE]
	at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor$AutowiredFieldElement.inject(AutowiredAnnotationBeanPostProcessor.java:566) ~[spring-beans-4.3.2.RELEASE.jar!/:4.3.2.RELEASE]
	at org.springframework.beans.factory.annotation.InjectionMetadata.inject(InjectionMetadata.java:88) ~[spring-beans-4.3.2.RELEASE.jar!/:4.3.2.RELEASE]
	at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor.postProcessPropertyValues(AutowiredAnnotationBeanPostProcessor.java:349) ~[spring-beans-4.3.2.RELEASE.jar!/:4.3.2.RELEASE]
	... 24 common frames omitted
```

`kubectl logs -f position-simulator-6f97fd485f-gplr8` follow the log

```
cat workloads.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: queue
spec:
  selector:
    matchLabels:
      app: queue
  replicas: 1
  template: # template for the pods
    metadata:
      labels:
        app: queue
    spec:
      containers:
      - name: webapp
        image: richardchesterwood/k8s-fleetman-queue:release1

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: position-simulator
spec:
  selector:
    matchLabels:
      app: position-simulator
  replicas: 1
  template: # template for the pods
    metadata:
      labels:
        app: position-simulator
    spec:
      containers:
      - name: webapp
        image: richardchesterwood/k8s-fleetman-position-simulator:release1
        env:
        - name: SPRING_PROFILES_ACTIVE
          value: production-microservice

kubectl apply -f workloads.yaml
deployment.apps "queue" unchanged
deployment.apps "position-simulator" configured

kga
NAME                                      READY     STATUS        RESTARTS   AGE
pod/position-simulator-6d8769d8-ghtmw     1/1       Running       0          9s
pod/position-simulator-6f97fd485f-gplr8   0/1       Terminating   6          8m
pod/queue-6bf9fd876-4xrqx                 1/1       Running       0          59m

NAME                      TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                          AGE
service/fleetman-queue    NodePort    10.110.95.121    <none>        8161:30010/TCP,61616:30536/TCP   55m
service/fleetman-webapp   NodePort    10.103.224.156   <none>        80:30080/TCP                     55m
service/kubernetes        ClusterIP   10.96.0.1        <none>        443/TCP                          2d

NAME                                 DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/position-simulator   1         1         1            1           10m
deployment.apps/queue                1         1         1            1           59m

NAME                                            DESIRED   CURRENT   READY     AGE
replicaset.apps/position-simulator-6d8769d8     1         1         1         9s
replicaset.apps/position-simulator-6f97fd485f   0         0         0         8m
replicaset.apps/position-simulator-dff7b7599    0         0         0         10m
replicaset.apps/queue-6bf9fd876                 1         1         1         59m

kubectl logs -f position-simulator-6d8769d8-ghtmw

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v1.4.0.RELEASE)

2019-06-05 06:13:36.205  INFO 1 --- [           main] c.v.s.PositionsimulatorApplication       : Starting PositionsimulatorApplication v0.0.1-SNAPSHOT on position-simulator-6d8769d8-ghtmw with PID 1 (/webapp.jar started by root in /)
2019-06-05 06:13:36.213  INFO 1 --- [           main] c.v.s.PositionsimulatorApplication       : The following profiles are active: production-microservice
2019-06-05 06:13:36.361  INFO 1 --- [           main] s.c.a.AnnotationConfigApplicationContext : Refreshing org.springframework.context.annotation.AnnotationConfigApplicationContext@443b7951: startup date [Wed Jun 05 06:13:36 UTC 2019]; root of context hierarchy
2019-06-05 06:13:38.011  INFO 1 --- [           main] o.s.j.e.a.AnnotationMBeanExporter        : Registering beans for JMX exposure on startup
2019-06-05 06:13:38.016  INFO 1 --- [           main] o.s.c.support.DefaultLifecycleProcessor  : Starting beans in phase 2147483647
2019-06-05 06:13:38.041  INFO 1 --- [           main] c.v.s.PositionsimulatorApplication       : Started PositionsimulatorApplication in 2.487 seconds (JVM running for 3.201)
2019-06-05 06:13:38.046  INFO 1 --- [           main] s.c.a.AnnotationConfigApplicationContext : Closing org.springframework.context.annotation.AnnotationConfigApplicationContext@443b7951: startup date [Wed Jun 05 06:13:36 UTC 2019]; root of context hierarchy
2019-06-05 06:13:38.048  INFO 1 --- [           main] o.s.c.support.DefaultLifecycleProcessor  : Stopping beans in phase 2147483647
2019-06-05 06:13:38.049  INFO 1 --- [           main] o.s.j.e.a.AnnotationMBeanExporter        : Unregistering JMX-exposed beans on shutdown
ca^H^H^C
```

![Digram](https://i.imgur.com/QhM7LEN.png)

## Deploying the Position Tracker

```
cat workloads.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: queue
spec:
  selector:
    matchLabels:
      app: queue
  replicas: 1
  template: # template for the pods
    metadata:
      labels:
        app: queue
    spec:
      containers:
      - name: queue
        image: richardchesterwood/k8s-fleetman-queue:release1

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: position-simulator
spec:
  selector:
    matchLabels:
      app: position-simulator
  replicas: 1
  template: # template for the pods
    metadata:
      labels:
        app: position-simulator
    spec:
      containers:
      - name: position-simulator
        image: richardchesterwood/k8s-fleetman-position-simulator:release1
        env:
        - name: SPRING_PROFILES_ACTIVE
          value: production-microservice
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: position-tracker
spec:
  selector:
    matchLabels:
      app: position-tracker
  replicas: 1
  template: # template for the pods
    metadata:
      labels:
        app: position-tracker
    spec:
      containers:
      - name: position-tracker
        image: richardchesterwood/k8s-fleetman-position-tracker:release1
        env:
        - name: SPRING_PROFILES_ACTIVE
          value: production-microservice

kubectl apply -f workloads.yaml
deployment.apps "queue" configured
deployment.apps "position-simulator" configured
deployment.apps "position-tracker" created

kga
NAME                                      READY     STATUS        RESTARTS   AGE
pod/position-simulator-6d8769d8-ghtmw     0/1       Terminating   0          13m
pod/position-simulator-7889db8b94-49qz2   1/1       Running       0          10s
pod/position-tracker-86d694f997-5j6fm     1/1       Running       0          10s
pod/queue-6bf9fd876-4xrqx                 1/1       Terminating   0          1h
pod/queue-9668b9bb4-4pqxr                 1/1       Running       0          10s

NAME                      TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                          AGE
service/fleetman-queue    NodePort    10.110.95.121    <none>        8161:30010/TCP,61616:30536/TCP   1h
service/fleetman-webapp   NodePort    10.103.224.156   <none>        80:30080/TCP                     1h
service/kubernetes        ClusterIP   10.96.0.1        <none>        443/TCP                          2d

NAME                                 DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/position-simulator   1         1         1            1           24m
deployment.apps/position-tracker     1         1         1            1           10s
deployment.apps/queue                1         1         1            1           1h

NAME                                            DESIRED   CURRENT   READY     AGE
replicaset.apps/position-simulator-6d8769d8     0         0         0         13m
replicaset.apps/position-simulator-6f97fd485f   0         0         0         21m
replicaset.apps/position-simulator-7889db8b94   1         1         1         10s
replicaset.apps/position-simulator-dff7b7599    0         0         0         24m
replicaset.apps/position-tracker-86d694f997     1         1         1         10s
replicaset.apps/queue-6bf9fd876                 0         0         0         1h
replicaset.apps/queue-9668b9bb4                 1         1         1         10s

kubectl describe pod position-tracker-86d694f997-5j6fm
Name:               position-tracker-86d694f997-5j6fm
Namespace:          default
Priority:           0
PriorityClassName:  <none>
Node:               minikube/10.0.2.15
Start Time:         Wed, 05 Jun 2019 16:26:58 +1000
Labels:             app=position-tracker
                    pod-template-hash=86d694f997
Annotations:        <none>
Status:             Running
IP:                 172.17.0.7
Controlled By:      ReplicaSet/position-tracker-86d694f997
Containers:
  position-tracker:
    Container ID:   docker://c0203a7654c944a92f1b18b48d5c84917b9ecc791536429e6b02ca6248edb78f
    Image:          richardchesterwood/k8s-fleetman-position-tracker:release1
    Image ID:       docker-pullable://richardchesterwood/k8s-fleetman-position-tracker@sha256:5da78e936eb77677fbf30e528253a543badc76a5dd3a356b6d13e716da187298
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Wed, 05 Jun 2019 16:27:08 +1000
    Ready:          True
    Restart Count:  0
    Environment:
      SPRING_PROFILES_ACTIVE:  production-microservice
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-nmqkz (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  default-token-nmqkz:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-nmqkz
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  1m    default-scheduler  Successfully assigned default/position-tracker-86d694f997-5j6fm to minikube
  Normal  Pulling    1m    kubelet, minikube  pulling image "richardchesterwood/k8s-fleetman-position-tracker:release1"
  Normal  Pulled     1m    kubelet, minikube  Successfully pulled image "richardchesterwood/k8s-fleetman-position-tracker:release1"
  Normal  Created    1m    kubelet, minikube  Created container
  Normal  Started    1m    kubelet, minikube  Started container

kubectl logs -f position-tracker-86d694f997-5j6fm

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v1.4.0.RELEASE)

2019-06-05 06:27:14.242  INFO 1 --- [           main] c.v.tracker.PositionTrackerApplication   : Starting PositionTrackerApplication v0.0.1-SNAPSHOT on position-tracker-86d694f997-5j6fm with PID 1 (/webapp.jar started by root in /)
2019-06-05 06:27:14.297  INFO 1 --- [           main] c.v.tracker.PositionTrackerApplication   : The following profiles are active: production-microservice
2019-06-05 06:27:15.147  INFO 1 --- [           main] ationConfigEmbeddedWebApplicationContext : Refreshing org.springframework.boot.context.embedded.AnnotationConfigEmbeddedWebApplicationContext@108c4c35: startup date [Wed Jun 05 06:27:15 UTC 2019]; root of context hierarchy
2019-06-05 06:27:25.787  INFO 1 --- [           main] s.b.c.e.t.TomcatEmbeddedServletContainer : Tomcat initialized with port(s): 8080 (http)
2019-06-05 06:27:25.852  INFO 1 --- [           main] o.apache.catalina.core.StandardService   : Starting service Tomcat
2019-06-05 06:27:25.854  INFO 1 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet Engine: Apache Tomcat/8.5.4
2019-06-05 06:27:25.985  INFO 1 --- [ost-startStop-1] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2019-06-05 06:27:25.986  INFO 1 --- [ost-startStop-1] o.s.web.context.ContextLoader            : Root WebApplicationContext: initialization completed in 10877 ms
2019-06-05 06:27:26.212  INFO 1 --- [ost-startStop-1] o.s.b.w.servlet.ServletRegistrationBean  : Mapping servlet: 'dispatcherServlet' to [/]
2019-06-05 06:27:26.221  INFO 1 --- [ost-startStop-1] o.s.b.w.servlet.FilterRegistrationBean   : Mapping filter: 'characterEncodingFilter' to: [/*]
2019-06-05 06:27:26.222  INFO 1 --- [ost-startStop-1] o.s.b.w.servlet.FilterRegistrationBean   : Mapping filter: 'hiddenHttpMethodFilter' to: [/*]
2019-06-05 06:27:26.223  INFO 1 --- [ost-startStop-1] o.s.b.w.servlet.FilterRegistrationBean   : Mapping filter: 'httpPutFormContentFilter' to: [/*]
2019-06-05 06:27:26.223  INFO 1 --- [ost-startStop-1] o.s.b.w.servlet.FilterRegistrationBean   : Mapping filter: 'requestContextFilter' to: [/*]
2019-06-05 06:27:26.789  INFO 1 --- [           main] s.w.s.m.m.a.RequestMappingHandlerAdapter : Looking for @ControllerAdvice: org.springframework.boot.context.embedded.AnnotationConfigEmbeddedWebApplicationContext@108c4c35: startup date [Wed Jun 05 06:27:15 UTC 2019]; root of context hierarchy
2019-06-05 06:27:26.900  INFO 1 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/vehicles/{vehicleName}],methods=[GET]}" onto public org.springframework.http.ResponseEntity<com.virtualpairprogrammers.tracker.domain.VehiclePosition> com.virtualpairprogrammers.tracker.rest.PositionReportsController.getLatestReportForVehicle(java.lang.String)
2019-06-05 06:27:26.901  INFO 1 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/vehicles/],methods=[GET]}" onto public java.util.Collection<com.virtualpairprogrammers.tracker.domain.VehiclePosition> com.virtualpairprogrammers.tracker.rest.PositionReportsController.getUpdatedPositions(java.util.Date)
2019-06-05 06:27:26.903  INFO 1 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/error],produces=[text/html]}" onto public org.springframework.web.servlet.ModelAndView org.springframework.boot.autoconfigure.web.BasicErrorController.errorHtml(javax.servlet.http.HttpServletRequest,javax.servlet.http.HttpServletResponse)
2019-06-05 06:27:26.904  INFO 1 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/error]}" onto public org.springframework.http.ResponseEntity<java.util.Map<java.lang.String, java.lang.Object>> org.springframework.boot.autoconfigure.web.BasicErrorController.error(javax.servlet.http.HttpServletRequest)
2019-06-05 06:27:26.948  INFO 1 --- [           main] o.s.w.s.handler.SimpleUrlHandlerMapping  : Mapped URL path [/webjars/**] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
2019-06-05 06:27:26.948  INFO 1 --- [           main] o.s.w.s.handler.SimpleUrlHandlerMapping  : Mapped URL path [/**] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
2019-06-05 06:27:26.991  INFO 1 --- [           main] o.s.w.s.handler.SimpleUrlHandlerMapping  : Mapped URL path [/**/favicon.ico] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
2019-06-05 06:27:27.364  INFO 1 --- [           main] o.s.j.e.a.AnnotationMBeanExporter        : Registering beans for JMX exposure on startup
2019-06-05 06:27:27.414  INFO 1 --- [           main] o.s.c.support.DefaultLifecycleProcessor  : Starting beans in phase 2147483647
2019-06-05 06:27:28.408  INFO 1 --- [           main] s.b.c.e.t.TomcatEmbeddedServletContainer : Tomcat started on port(s): 8080 (http)
2019-06-05 06:27:28.436  INFO 1 --- [           main] c.v.tracker.PositionTrackerApplication   : Started PositionTrackerApplication in 17.136 seconds (JVM running for 20.295)
```

![ActiveMQ screenshot](https://i.imgur.com/MrwaPyf.png)

```
cat services.yaml
apiVersion: v1
kind: Service
metadata:
 name: fleetman-webapp

spec:
 # This defines which pods are going to be represented by this Service
 # The service becomes a network endpoint for either other services
 #  or maybe external users to connect to (eg browser)
 selector:
   app: webapp

 ports:
   - name: http
     port: 80
     nodePort: 30080

 type: NodePort

---
apiVersion: v1
kind: Service
metadata:
 name: fleetman-queue

spec:
 # This defines which pods are going to be represented by this Service
 # The service becomes a network endpoint for either other services
 #  or maybe external users to connect to (eg browser)
 selector:
   app: queue

 ports:
   - name: http
     port: 8161
     nodePort: 30010

   - name: endpoint
     port: 61616

 type: NodePort

---
apiVersion: v1
kind: Service
metadata:
 name: fleetman-position-tracker

spec:
 # This defines which pods are going to be represented by this Service
 # The service becomes a network endpoint for either other services
 #  or maybe external users to connect to (eg browser)
 selector:
   app: position-tracker

 ports:
   - name: http
     port: 8080

 type: ClusterIP

kubectl apply -f services.yaml
service "fleetman-webapp" unchanged
service "fleetman-queue" unchanged
service "fleetman-position-tracker" configured

kga
NAME                                      READY     STATUS    RESTARTS   AGE
pod/position-simulator-589c64887f-lhl8g   1/1       Running   0          38m
pod/position-tracker-86d694f997-5j6fm     1/1       Running   0          45m
pod/queue-9668b9bb4-4pqxr                 1/1       Running   0          45m

NAME                                TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                          AGE
service/fleetman-position-tracker   ClusterIP   10.104.177.133   <none>        8080/TCP                         3m
service/fleetman-queue              NodePort    10.110.95.121    <none>        8161:30010/TCP,61616:30536/TCP   1h
service/fleetman-webapp             NodePort    10.103.224.156   <none>        80:30080/TCP                     1h
service/kubernetes                  ClusterIP   10.96.0.1        <none>        443/TCP                          2d

NAME                                 DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/position-simulator   1         1         1            1           1h
deployment.apps/position-tracker     1         1         1            1           45m
deployment.apps/queue                1         1         1            1           1h

NAME                                            DESIRED   CURRENT   READY     AGE
replicaset.apps/position-simulator-589c64887f   1         1         1         38m
replicaset.apps/position-simulator-6d8769d8     0         0         0         58m
replicaset.apps/position-simulator-6f97fd485f   0         0         0         1h
replicaset.apps/position-simulator-7889db8b94   0         0         0         45m
replicaset.apps/position-simulator-dff7b7599    0         0         0         1h
replicaset.apps/position-tracker-86d694f997     1         1         1         45m
replicaset.apps/queue-6bf9fd876                 0         0         0         1h
replicaset.apps/queue-9668b9bb4                 1         1         1         45m
```
![Check](https://i.imgur.com/3tgbP7M.png)

## Deploying the API Gateway

Add code in services.yaml file:
```
---
apiVersion: v1
kind: Service
metadata:
 name: fleetman-api-gateway

spec:
 # This defines which pods are going to be represented by this Service
 # The service becomes a network endpoint for either other services
 #  or maybe external users to connect to (eg browser)
 selector:
   app: api-gateway

 ports:
   - name: http
     port: 8080

 type: ClusterIP
```

Add code in workloads.yaml file:
```
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-gateway
spec:
  selector:
    matchLabels:
      app: api-gateway
  replicas: 1
  template: # template for the pods
    metadata:
      labels:
        app: api-gateway
    spec:
      containers:
      - name: api-gateway
        image: richardchesterwood/k8s-fleetman-api-gateway:release1
        env:
        - name: SPRING_PROFILES_ACTIVE
          value: production-microservice
```

`kubectl apply -f .`

## Deploying the Webapp

Add for workloads.yaml file:
```
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
spec:
  selector:
    matchLabels:
      app: webapp
  replicas: 1
  template: # template for the pods
    metadata:
      labels:
        app: webapp
    spec:
      containers:
      - name: webapp
        image: richardchesterwood/k8s-fleetman-webapp-angular:release1
        env:
        - name: SPRING_PROFILES_ACTIVE
          value: production-microservice
```
For serivce.yaml no change.

![final](https://i.imgur.com/dadMM2m.jpg)

# Persistence

**New Release!!**
---
Release 2 is now available!

Please update all images to :release2 tag

New feature: vehicle tracking shows the history of each vehicle

---

Change image in workloads.yaml from release1 to release2 by `:%s/release1/release2/g`
![release2](https://i.imgur.com/oOLbvYT.jpg)

## Upgrading to a Mongo Pod
![release3](https://i.imgur.com/zT2lk5f.jpg)

## Volume Mounts
[Volume v1 core](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.14/#volume-v1-core)

## PersistentVolumeClaims
```
cat storage.yaml
# What do want?
apiVersion: v1
kind: PersistentVolumeClain
metadata:
  name: mongo-pvc
spec:
  accessMode:
    - ReadWriteOnce
  resouces:
    requests:
      storage: 20Gi

---
# How do we want it implemented
apiVersion: v1
kind: PersistentVolume
metadata:
  name: local-storage
spec:
  capacity:
    storage: 20Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/somenew/dirctory/structure/"
    type: DirectoryOrCreate
```
## StorageClasses and Binding
```
 cat mongo-stack.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongodb
spec:
  selector:
    matchLabels:
      app: mongodb
  replicas: 1
  template: # template for the pods
    metadata:
      labels:
        app: mongodb
    spec:
      containers:
      - name: mongodb
        image: mongo:3.6.12-xenial
        volumeMounts:
          - name: mongo-persistent-storage
            mountPath: /data/db
      volumes:
        - name: mongo-persistent-storage
          # pointer to the configuration of HOW we want the mount to be implemented
          persistentVolumeClaim:
             claimName: mongo-pvc
---
kind: Service
apiVersion: v1
metadata:
  name: fleetman-mongodb
spec:
  selector:
    app: mongodb
  ports:
    - name: mongoport
      port: 27017
  type: ClusterIP

 cat storage.yaml
# What do want?
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mongo-pvc
spec:
  storageClassName: mylocalstorage
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi

---
# How do we want it implemented
apiVersion: v1
kind: PersistentVolume
metadata:
  name: local-storage
spec:
  storageClassName: mylocalstorage
  capacity:
    storage: 20Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/somenew/dirctory/structure/"
    type: DirectoryOrCreate
```
# DEPLOYING TO THE AWS CLOUD

## Getting started with AWS
![image1](https://i.imgur.com/FvK1lg2.png)

![image2](https://i.imgur.com/1pQD2XE.png)

![image3](https://i.imgur.com/KD5pigA.png)

![ebs](https://i.imgur.com/yZoC4kE.png)

## Introducing Kops

[Kops](https://github.com/kubernetes/kops)

This apparently is the easiest way to get a production-grade Kubernetes cluster up and running.

## Installing the Kops Environment
[Deploy to aws](https://github.com/kubernetes/kops/blob/master/docs/aws.md)

## Configuring your first cluster

[Cluster State storage](https://github.com/kubernetes/kops/blob/master/docs/aws.md#cluster-state-storage)

```
[ec2-user@foobar ~]$ export AWS_ACCESS_KEY_ID=$(aws configure get aws_access_key_id)
[ec2-user@foobar ~]$ export AWS_SECRET_ACCESS_KEY=$(aws configure get aws_secret_access_key)
[ec2-user@foobar ~]$ export NAME=fleetman.k8s.local
[ec2-user@foobar ~]$ export KOPS_STATE_STORE=s3://stanzhou-state-storage
[ec2-user@foobar ~]$ aws ec2 describe-availability-zones --region ap-southeast-2
{
    "AvailabilityZones": [
        {
            "State": "available",
            "ZoneName": "ap-southeast-2a",
            "Messages": [],
            "ZoneId": "apse2-az1",
            "RegionName": "ap-southeast-2"
        },
        {
            "State": "available",
            "ZoneName": "ap-southeast-2b",
            "Messages": [],
            "ZoneId": "apse2-az3",
            "RegionName": "ap-southeast-2"
        },
        {
            "State": "available",
            "ZoneName": "ap-southeast-2c",
            "Messages": [],
            "ZoneId": "apse2-az2",
            "RegionName": "ap-southeast-2"
        }
    ]
}

$ kops create cluster --zones ap-southeast-2a,ap-southeast-2b,ap-southeast-2c ${NAME}
I0607 06:10:02.189636    3468 create_cluster.go:519] Inferred --cloud=aws from zone "ap-southeast-2a"
I0607 06:10:02.243690    3468 subnets.go:184] Assigned CIDR 172.20.32.0/19 to subnet ap-southeast-2a
I0607 06:10:02.243802    3468 subnets.go:184] Assigned CIDR 172.20.64.0/19 to subnet ap-southeast-2b
I0607 06:10:02.243857    3468 subnets.go:184] Assigned CIDR 172.20.96.0/19 to subnet ap-southeast-2c
Previewing changes that will be made:


SSH public key must be specified when running with AWS (create with `kops create secret --name fleetman.k8s.local sshpublickey admin -i ~/.ssh/id_rsa.pub`)

[ec2-user@foobar ~]$ ssh-keygen -b 2048 -t rsa -f ~/.ssh/id_rsa
[ec2-user@foobar ~]$ kops create secret --name ${NAME} sshpublickey admin -i ~/.ssh/id_rsa.pub
[ec2-user@foobar ~]$ kops edit ig nodes --name ${NAME}
[ec2-user@foobar ~]$ kops get ig --name ${NAME}
NAME                    ROLE    MACHINETYPE     MIN     MAX     ZONES
master-ap-southeast-2a  Master  m3.medium       1       1       ap-southeast-2a
nodes                   Node    t2.medium       3       5       ap-southeast-2a,ap-southeast-2b,ap-southeast-2c
[ec2-user@foobar ~]$ kops edit ig master-ap-southeast-2a --name ${NAME}
Edit cancelled, no changes made.
```
## Running the Cluster
```
[ec2-user@i ~]$ kops update cluster ${NAME} --yes
I0607 06:28:38.011102   32239 apply_cluster.go:559] Gossip DNS: skipping DNS validation
I0607 06:28:38.263244   32239 executor.go:103] Tasks: 0 done / 94 total; 42 can run
I0607 06:28:39.702035   32239 vfs_castore.go:729] Issuing new certificate: "apiserver-aggregator-ca"
I0607 06:28:40.216189   32239 vfs_castore.go:729] Issuing new certificate: "etcd-clients-ca"
I0607 06:28:40.356654   32239 vfs_castore.go:729] Issuing new certificate: "etcd-peers-ca-main"
I0607 06:28:40.743191   32239 vfs_castore.go:729] Issuing new certificate: "etcd-peers-ca-events"
I0607 06:28:40.824760   32239 vfs_castore.go:729] Issuing new certificate: "etcd-manager-ca-events"
I0607 06:28:41.265388   32239 vfs_castore.go:729] Issuing new certificate: "etcd-manager-ca-main"
I0607 06:28:41.373174   32239 vfs_castore.go:729] Issuing new certificate: "ca"
I0607 06:28:41.551597   32239 executor.go:103] Tasks: 42 done / 94 total; 26 can run
I0607 06:28:42.539134   32239 vfs_castore.go:729] Issuing new certificate: "kube-scheduler"
I0607 06:28:42.891972   32239 vfs_castore.go:729] Issuing new certificate: "kubecfg"
I0607 06:28:43.157916   32239 vfs_castore.go:729] Issuing new certificate: "apiserver-proxy-client"
I0607 06:28:43.556052   32239 vfs_castore.go:729] Issuing new certificate: "kubelet"
I0607 06:28:43.677894   32239 vfs_castore.go:729] Issuing new certificate: "apiserver-aggregator"
I0607 06:28:43.748079   32239 vfs_castore.go:729] Issuing new certificate: "kube-proxy"
I0607 06:28:44.025132   32239 vfs_castore.go:729] Issuing new certificate: "kubelet-api"
I0607 06:28:44.589696   32239 vfs_castore.go:729] Issuing new certificate: "kube-controller-manager"
I0607 06:28:44.730038   32239 vfs_castore.go:729] Issuing new certificate: "kops"
I0607 06:28:44.864527   32239 executor.go:103] Tasks: 68 done / 94 total; 22 can run
I0607 06:28:45.089177   32239 launchconfiguration.go:364] waiting for IAM instance profile "masters.fleetman.k8s.local" to be ready
I0607 06:28:45.101954   32239 launchconfiguration.go:364] waiting for IAM instance profile "nodes.fleetman.k8s.local" to be ready
I0607 06:28:55.483430   32239 executor.go:103] Tasks: 90 done / 94 total; 3 can run
I0607 06:28:55.974524   32239 vfs_castore.go:729] Issuing new certificate: "master"
I0607 06:28:56.119668   32239 executor.go:103] Tasks: 93 done / 94 total; 1 can run
I0607 06:28:56.336766   32239 executor.go:103] Tasks: 94 done / 94 total; 0 can run
I0607 06:28:56.407976   32239 update_cluster.go:291] Exporting kubecfg for cluster
kops has set your kubectl context to fleetman.k8s.local

Cluster is starting.  It should be ready in a few minutes.

Suggestions:
 * validate cluster: kops validate cluster
 * list nodes: kubectl get nodes --show-labels
 * ssh to the master: ssh -i ~/.ssh/id_rsa admin@api.fleetman.k8s.local
 * the admin user is specific to Debian. If not using Debian please use the appropriate user based on your OS.
 * read about installing addons at: https://github.com/kubernetes/kops/blob/master/docs/addons.md.

[ec2-user@i ~]$ kops validate cluster
Using cluster from kubectl context: fleetman.k8s.local

Validating cluster fleetman.k8s.local

INSTANCE GROUPS
NAME                    ROLE    MACHINETYPE     MIN     MAX     SUBNETS
master-ap-southeast-2a  Master  m3.medium       1       1       ap-southeast-2a
nodes                   Node    t2.medium       3       5       ap-southeast-2a,ap-southeast-2b,ap-southeast-2c

NODE STATUS
NAME                                                    ROLE    READY
ip-172-20-115-253.ap-southeast-2.compute.internal       node    True
ip-172-20-39-212.ap-southeast-2.compute.internal        node    True
ip-172-20-45-219.ap-southeast-2.compute.internal        master  True
ip-172-20-89-8.ap-southeast-2.compute.internal          node    True

Your cluster fleetman.k8s.local is ready

[ec2-user@i ~]$ kubectl get all
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   100.64.0.1   <none>        443/TCP   4m50s
```
## Provisioning SSD drives with a StorageClass

We have a workloads yaml file, where we'd be finding the pods that we want to deploy to our cluster. We have mongostack which is a specialist file, just for the mongo database. We have storage.yaml, which is currently defining that we want to store the mongo data in a local directory on the host machine. And we have a yaml file for the services.

[aws ebs])https://kubernetes.io/docs/concepts/storage/storage-classes/#aws-ebs)
```
[ec2-user@i ~]$ cat storage-aws.yaml
# What do want?
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mongo-pvc
spec:
  storageClassName: cloud-ssd
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 7Gi

---
# How do we want it implemented
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: cloud-ssd
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2

[ec2-user@ip-1 ~]$ kubectl get pvc
NAME        STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
mongo-pvc   Bound    pvc-b2ed286e-88f3-11e9-b509-02985f983814   7Gi        RWO            cloud-ssd      3m45s

[ec2-user@ip-1 ~]$ kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM               STORAGECLASS   REASON   AGE
pvc-b2ed286e-88f3-11e9-b509-02985f983814   7Gi        RWO            Delete           Bound    default/mongo-pvc   cloud-ssd               18s
```

![ebs volume created](https://i.imgur.com/91K0spE.png)

## Deploying the Fleetman Workload
```
[ec2-user@ip-1-9 ~]$ cat services.yaml
apiVersion: v1
kind: Service
metadata:
 name: fleetman-webapp

spec:
 # This defines which pods are going to be represented by this Service
 # The service becomes a network endpoint for either other services
 #  or maybe external users to connect to (eg browser)
 selector:
   app: webapp

 ports:
   - name: http
     port: 80
 type: LoadBalancer

---
apiVersion: v1
kind: Service
metadata:
 name: fleetman-queue

spec:
 # This defines which pods are going to be represented by this Service
 # The service becomes a network endpoint for either other services
 #  or maybe external users to connect to (eg browser)
 selector:
   app: queue

 ports:
   - name: http
     port: 8161

   - name: endpoint
     port: 61616

 type: ClusterIP

---
apiVersion: v1
kind: Service
metadata:
 name: fleetman-position-tracker

spec:
 # This defines which pods are going to be represented by this Service
 # The service becomes a network endpoint for either other services
 #  or maybe external users to connect to (eg browser)
 selector:
   app: position-tracker

 ports:
   - name: http
     port: 8080

 type: ClusterIP

---
apiVersion: v1
kind: Service
metadata:
 name: fleetman-api-gateway

spec:
 # This defines which pods are going to be represented by this Service
 # The service becomes a network endpoint for either other services
 #  or maybe external users to connect to (eg browser)
 selector:
   app: api-gateway

 ports:
   - name: http
     port: 8080

 type: ClusterIP

[ec2-user@ip-1-4-9 ~]$ cat workloads.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: queue
spec:
  selector:
    matchLabels:
      app: queue
  replicas: 1
  template: # template for the pods
    metadata:
      labels:
        app: queue
    spec:
      containers:
      - name: queue
        image: richardchesterwood/k8s-fleetman-queue:release2

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: position-simulator
spec:
  selector:
    matchLabels:
      app: position-simulator
  replicas: 1
  template: # template for the pods
    metadata:
      labels:
        app: position-simulator
    spec:
      containers:
      - name: position-simulator
        image: richardchesterwood/k8s-fleetman-position-simulator:release2
        env:
        - name: SPRING_PROFILES_ACTIVE
          value: production-microservice
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: position-tracker
spec:
  selector:
    matchLabels:
      app: position-tracker
  replicas: 1
  template: # template for the pods
    metadata:
      labels:
        app: position-tracker
    spec:
      containers:
      - name: position-tracker
        image: richardchesterwood/k8s-fleetman-position-tracker:release3
        env:
        - name: SPRING_PROFILES_ACTIVE
          value: production-microservice
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-gateway
spec:
  selector:
    matchLabels:
      app: api-gateway
  replicas: 1
  template: # template for the pods
    metadata:
      labels:
        app: api-gateway
    spec:
      containers:
      - name: api-gateway
        image: richardchesterwood/k8s-fleetman-api-gateway:release2
        env:
        - name: SPRING_PROFILES_ACTIVE
          value: production-microservice
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
spec:
  selector:
    matchLabels:
      app: webapp
  replicas: 1
  template: # template for the pods
    metadata:
      labels:
        app: webapp
    spec:
      containers:
      - name: webapp
        image: richardchesterwood/k8s-fleetman-webapp-angular:release2
        env:
        - name: SPRING_PROFILES_ACTIVE
          value: production-microservice
[ec2-user@-4-9 ~]$ kubectl apply -f .
deployment.apps/mongodb unchanged
service/fleetman-mongodb unchanged
service/fleetman-webapp unchanged
service/fleetman-queue unchanged
service/fleetman-position-tracker unchanged
service/fleetman-api-gateway unchanged
persistentvolumeclaim/mongo-pvc unchanged
storageclass.storage.k8s.io/cloud-ssd unchanged
deployment.apps/queue configured
deployment.apps/position-simulator configured
deployment.apps/position-tracker unchanged
deployment.apps/api-gateway configured
deployment.apps/webapp configured
[ec2-user@ip-172-31-4-9 ~]$ kubectl get all
NAME                                      READY   STATUS    RESTARTS   AGE
pod/api-gateway-5d445d6f69-5gmm4          1/1     Running   0          82s
pod/mongodb-5559556bf-jjkpw               1/1     Running   0          20m
pod/position-simulator-7ffd4f8f68-2gnjr   1/1     Running   0          82s
pod/position-tracker-5ff4fb7479-jjj9f     1/1     Running   0          11m
pod/queue-75f4ddd795-6vn9d                1/1     Running   0          82s
pod/webapp-689dd9b4f4-ntdpz               1/1     Running   0          82s

NAME                                TYPE           CLUSTER-IP       EXTERNAL-IP                                                                    PORT(S)              AGE
service/fleetman-api-gateway        ClusterIP      100.65.245.184   <none>                                                                         8080/TCP             11m
service/fleetman-mongodb            ClusterIP      100.70.242.251   <none>                                                                         27017/TCP            20m
service/fleetman-position-tracker   ClusterIP      100.70.48.51     <none>                                                                         8080/TCP             11m
service/fleetman-queue              ClusterIP      100.65.176.119   <none>                                                                         8161/TCP,61616/TCP   11m
service/fleetman-webapp             LoadBalancer   100.70.56.219    a660e0c5e88f611e9b50902985f98381-1044209190.ap-southeast-2.elb.amazonaws.com   80:30149/TCP         11m
service/kubernetes                  ClusterIP      100.64.0.1       <none>                                                                         443/TCP              68m

NAME                                 DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/api-gateway          1         1         1            1           11m
deployment.apps/mongodb              1         1         1            1           20m
deployment.apps/position-simulator   1         1         1            1           11m
deployment.apps/position-tracker     1         1         1            1           11m
deployment.apps/queue                1         1         1            1           11m
deployment.apps/webapp               1         1         1            1           11m

NAME                                            DESIRED   CURRENT   READY   AGE
replicaset.apps/api-gateway-5d445d6f69          1         1         1       82s
replicaset.apps/api-gateway-6d7dccc464          0         0         0       11m
replicaset.apps/mongodb-5559556bf               1         1         1       20m
replicaset.apps/position-simulator-549554f4d9   0         0         0       11m
replicaset.apps/position-simulator-7ffd4f8f68   1         1         1       82s
replicaset.apps/position-tracker-5ff4fb7479     1         1         1       11m
replicaset.apps/queue-75f4ddd795                1         1         1       82s
replicaset.apps/queue-b46577b46                 0         0         0       11m
replicaset.apps/webapp-689dd9b4f4               1         1         1       82s
replicaset.apps/webapp-6cdd565c5                0         0         0       11m
[ec2-user@ip-172-31-4-9 ~]$ kubectl log -f pod/position-tracker-5ff4fb7479-jjj9f
log is DEPRECATED and will be removed in a future version. Use logs instead.
2019-06-07 07:42:40.878 ERROR 1 --- [enerContainer-1] o.s.j.l.DefaultMessageListenerContainer  : Could not refresh JMS Connection for destination 'positionQueue' - retrying using FixedBackOff{interval=5000, currentAttempts=15, maxAttempts=unlimited}. Cause: Could not connect to broker URL: tcp://fleetman-queue.default.svc.cluster.local:61616. Reason: java.net.SocketException: Socket closed
2019-06-07 07:42:46.002  INFO 1 --- [enerContainer-1] o.s.j.l.DefaultMessageListenerContainer  : Successfully refreshed JMS Connection
2019-06-07 07:42:47.440  INFO 1 --- [nio-8080-exec-8] org.mongodb.driver.connection            : Opened connection [connectionId{localValue:3, serverValue:3}] to fleetman-mongodb.default.svc.cluster.local:27017
illljffkkkkerror: unexpected EOF
```
Docker swarm uses a concept called a rooting, or routing, mesh to find the node that your web application is running on. None of that is used here, it uses a standard AWS load balancer to find the correct node.

## Setting up a real Domain Name

Add a CNAME record in your own domain to ELB address.

## Surviving Node Failure

### Requirement

---
Even in the vent of a Node (or Availability Zone) failure, the web site must be accessible

It doesn't matter if reports from vehicles stop coming in, as long as service is restored within a few minutes

```
[ec2-user@foobar ~]$ kubectl get pods
NAME                                  READY   STATUS    RESTARTS   AGE
api-gateway-5d445d6f69-5gmm4          1/1     Running   0          93m
mongodb-5559556bf-jjkpw               1/1     Running   0          112m
position-simulator-7ffd4f8f68-2gnjr   1/1     Running   0          93m
position-tracker-5ff4fb7479-jjj9f     1/1     Running   0          103m
queue-75f4ddd795-6vn9d                1/1     Running   0          93m
webapp-689dd9b4f4-ntdpz               1/1     Running   0          93m
[ec2-user@ip-172-31-4-9 ~]$ kubectl get pods -o wide
NAME                                  READY   STATUS    RESTARTS   AGE    IP           NODE                                                NOMINATED NODE
api-gateway-5d445d6f69-5gmm4          1/1     Running   0          93m    100.96.3.5   ip-172-20-39-212.ap-southeast-2.compute.internal    <none>
mongodb-5559556bf-jjkpw               1/1     Running   0          112m   100.96.1.4   ip-172-20-89-8.ap-southeast-2.compute.internal      <none>
position-simulator-7ffd4f8f68-2gnjr   1/1     Running   0          93m    100.96.2.5   ip-172-20-115-253.ap-southeast-2.compute.internal   <none>
position-tracker-5ff4fb7479-jjj9f     1/1     Running   0          103m   100.96.3.4   ip-172-20-39-212.ap-southeast-2.compute.internal    <none>
queue-75f4ddd795-6vn9d                1/1     Running   0          93m    100.96.1.5   ip-172-20-89-8.ap-southeast-2.compute.internal      <none>
webapp-689dd9b4f4-ntdpz               1/1     Running   0          93m    100.96.2.6   ip-172-20-115-253.ap-southeast-2.compute.internal   <none>
```
## Replicating Pods
For our example, take the queue pod, give it two replicas and therefore, in the event of a node failure, one of the nodes will always survive. Unfortunately you can't do that because this particular pod, the queue pod is stateful. In other words, it contains data. And because it contains data, if you replicate it, you're going to end up with a kind of a split brain situation, where half the data is in one part, half the data is in the other part. And all kinds of chaos will follow on from that. Really what you're aiming for with any pod is to make it stateless, so it's not holding data.

## Deleting the Cluster

```
[ec2-user@foobar ~]$ export NAME=fleetman.k8s.local
[ec2-user@foobar ~]$ kops delete cluster --name ${NAME} --yes

State Store: Required value: Please set the --state flag or export KOPS_STATE_STORE.
For example, a valid value follows the format s3://<bucket>.
You can find the supported stores in https://github.com/kubernetes/kops/blob/master/docs/state.md.
[ec2-user@ip-172-31-4-9 ~]$ export KOPS_STATE_STORE=s3://stanzhou-state-storage
[ec2-user@ip-172-31-4-9 ~]$ kops delete cluster --name ${NAME} --yes
TYPE                    NAME                                                                            ID
autoscaling-config      master-ap-southeast-2a.masters.fleetman.k8s.local-20190607062844                master-ap-southeast-2a.masters.fleetman.k8s.local-20190607062844
autoscaling-config      nodes.fleetman.k8s.local-20190607062844                                         nodes.fleetman.k8s.local-20190607062844
autoscaling-group       master-ap-southeast-2a.masters.fleetman.k8s.local                               master-ap-southeast-2a.masters.fleetman.k8s.local
autoscaling-group       nodes.fleetman.k8s.local                                                        nodes.fleetman.k8s.local
dhcp-options            fleetman.k8s.local                                                              dopt-0a0be88814a0c83a9
iam-instance-profile    masters.fleetman.k8s.local                                                      masters.fleetman.k8s.local
iam-instance-profile    nodes.fleetman.k8s.local                                                        nodes.fleetman.k8s.local
iam-role                masters.fleetman.k8s.local                                                      masters.fleetman.k8s.local
iam-role                nodes.fleetman.k8s.local                                                        nodes.fleetman.k8s.local
instance                master-ap-southeast-2a.masters.fleetman.k8s.local                               i-012c7c446b65343d4
instance                nodes.fleetman.k8s.local                                                        i-07583ee103342be9a
instance                nodes.fleetman.k8s.local                                                        i-079cea61e4a7736b9
instance                nodes.fleetman.k8s.local                                                        i-0bf949dbd290d81c3
internet-gateway        fleetman.k8s.local                                                              igw-0a939d66d6e93e0d5
keypair                 kubernetes.fleetman.k8s.local-fc:11:5b:a8:1d:16:4a:36:36:15:2d:9f:f3:69:d2:0a   kubernetes.fleetman.k8s.local-fc:11:5b:a8:1d:16:4a:36:36:15:2d:9f:f3:69:d2:0a
load-balancer                                                                                           a660e0c5e88f611e9b50902985f98381
load-balancer           api.fleetman.k8s.local                                                          api-fleetman-k8s-local-tkmafs
route-table             fleetman.k8s.local                                                              rtb-06b591f24a01973f6
security-group                                                                                          sg-07b79756088cf753c
security-group          api-elb.fleetman.k8s.local                                                      sg-005c9b49b63793004
security-group          masters.fleetman.k8s.local                                                      sg-07ef00367ce1a7b62
security-group          nodes.fleetman.k8s.local                                                        sg-01f81918cdbaba212
subnet                  ap-southeast-2a.fleetman.k8s.local                                              subnet-060ec2db19027cf6a
subnet                  ap-southeast-2b.fleetman.k8s.local                                              subnet-0def003bdbfd97915
subnet                  ap-southeast-2c.fleetman.k8s.local                                              subnet-0016862a30fe5f443
volume                  a.etcd-events.fleetman.k8s.local                                                vol-0d21c97044fcc06dd
volume                  a.etcd-main.fleetman.k8s.local                                                  vol-0f1c9f3e983c5848a
volume                  fleetman.k8s.local-dynamic-pvc-b2ed286e-88f3-11e9-b509-02985f983814             vol-074914e494e4b656d
vpc                     fleetman.k8s.local                                                              vpc-0775d4b463932d2f7

load-balancer:api-fleetman-k8s-local-tkmafs     ok
load-balancer:a660e0c5e88f611e9b50902985f98381  ok
autoscaling-group:nodes.fleetman.k8s.local      ok
keypair:kubernetes.fleetman.k8s.local-fc:11:5b:a8:1d:16:4a:36:36:15:2d:9f:f3:69:d2:0a   ok
internet-gateway:igw-0a939d66d6e93e0d5  still has dependencies, will retry
autoscaling-group:master-ap-southeast-2a.masters.fleetman.k8s.local     ok
instance:i-012c7c446b65343d4    ok
instance:i-07583ee103342be9a    ok
instance:i-0bf949dbd290d81c3    ok
instance:i-079cea61e4a7736b9    ok
iam-instance-profile:nodes.fleetman.k8s.local   ok
iam-instance-profile:masters.fleetman.k8s.local ok
iam-role:nodes.fleetman.k8s.local       ok
iam-role:masters.fleetman.k8s.local     ok
subnet:subnet-0def003bdbfd97915 still has dependencies, will retry
subnet:subnet-0016862a30fe5f443 still has dependencies, will retry
autoscaling-config:nodes.fleetman.k8s.local-20190607062844      ok
volume:vol-0d21c97044fcc06dd    still has dependencies, will retry
autoscaling-config:master-ap-southeast-2a.masters.fleetman.k8s.local-20190607062844     ok
volume:vol-0f1c9f3e983c5848a    still has dependencies, will retry
subnet:subnet-060ec2db19027cf6a still has dependencies, will retry
volume:vol-074914e494e4b656d    still has dependencies, will retry
security-group:sg-005c9b49b63793004     still has dependencies, will retry
security-group:sg-01f81918cdbaba212     still has dependencies, will retry
security-group:sg-07b79756088cf753c     still has dependencies, will retry
security-group:sg-07ef00367ce1a7b62     still has dependencies, will retry
Not all resources deleted; waiting before reattempting deletion
        route-table:rtb-06b591f24a01973f6
        internet-gateway:igw-0a939d66d6e93e0d5
        security-group:sg-005c9b49b63793004
        subnet:subnet-060ec2db19027cf6a
        security-group:sg-07b79756088cf753c
        subnet:subnet-0016862a30fe5f443
        subnet:subnet-0def003bdbfd97915
        security-group:sg-07ef00367ce1a7b62
        volume:vol-0d21c97044fcc06dd
        dhcp-options:dopt-0a0be88814a0c83a9
        volume:vol-074914e494e4b656d
        security-group:sg-01f81918cdbaba212
        volume:vol-0f1c9f3e983c5848a
        vpc:vpc-0775d4b463932d2f7
subnet:subnet-060ec2db19027cf6a still has dependencies, will retry
subnet:subnet-0def003bdbfd97915 still has dependencies, will retry
subnet:subnet-0016862a30fe5f443 still has dependencies, will retry
volume:vol-074914e494e4b656d    still has dependencies, will retry
volume:vol-0f1c9f3e983c5848a    still has dependencies, will retry
internet-gateway:igw-0a939d66d6e93e0d5  still has dependencies, will retry
volume:vol-0d21c97044fcc06dd    still has dependencies, will retry
security-group:sg-01f81918cdbaba212     still has dependencies, will retry
security-group:sg-005c9b49b63793004     still has dependencies, will retry
security-group:sg-07ef00367ce1a7b62     still has dependencies, will retry
security-group:sg-07b79756088cf753c     still has dependencies, will retry
Not all resources deleted; waiting before reattempting deletion
        volume:vol-074914e494e4b656d
        security-group:sg-01f81918cdbaba212
        volume:vol-0f1c9f3e983c5848a
        vpc:vpc-0775d4b463932d2f7
        route-table:rtb-06b591f24a01973f6
        internet-gateway:igw-0a939d66d6e93e0d5
        security-group:sg-005c9b49b63793004
        subnet:subnet-060ec2db19027cf6a
        subnet:subnet-0016862a30fe5f443
        security-group:sg-07b79756088cf753c
        volume:vol-0d21c97044fcc06dd
        subnet:subnet-0def003bdbfd97915
        security-group:sg-07ef00367ce1a7b62
        dhcp-options:dopt-0a0be88814a0c83a9
subnet:subnet-060ec2db19027cf6a still has dependencies, will retry
subnet:subnet-0def003bdbfd97915 still has dependencies, will retry
volume:vol-0f1c9f3e983c5848a    still has dependencies, will retry
volume:vol-0d21c97044fcc06dd    still has dependencies, will retry
internet-gateway:igw-0a939d66d6e93e0d5  still has dependencies, will retry
volume:vol-074914e494e4b656d    still has dependencies, will retry
subnet:subnet-0016862a30fe5f443 still has dependencies, will retry
security-group:sg-07b79756088cf753c     still has dependencies, will retry
security-group:sg-01f81918cdbaba212     still has dependencies, will retry
security-group:sg-07ef00367ce1a7b62     still has dependencies, will retry
security-group:sg-005c9b49b63793004     still has dependencies, will retry
Not all resources deleted; waiting before reattempting deletion
        route-table:rtb-06b591f24a01973f6
        internet-gateway:igw-0a939d66d6e93e0d5
        security-group:sg-005c9b49b63793004
        subnet:subnet-060ec2db19027cf6a
        subnet:subnet-0016862a30fe5f443
        security-group:sg-07b79756088cf753c
        volume:vol-0d21c97044fcc06dd
        subnet:subnet-0def003bdbfd97915
        security-group:sg-07ef00367ce1a7b62
        dhcp-options:dopt-0a0be88814a0c83a9
        volume:vol-074914e494e4b656d
        security-group:sg-01f81918cdbaba212
        volume:vol-0f1c9f3e983c5848a
        vpc:vpc-0775d4b463932d2f7
subnet:subnet-0def003bdbfd97915 still has dependencies, will retry
subnet:subnet-060ec2db19027cf6a still has dependencies, will retry
internet-gateway:igw-0a939d66d6e93e0d5  still has dependencies, will retry
volume:vol-074914e494e4b656d    still has dependencies, will retry
volume:vol-0d21c97044fcc06dd    ok
volume:vol-0f1c9f3e983c5848a    ok
security-group:sg-01f81918cdbaba212     still has dependencies, will retry
subnet:subnet-0016862a30fe5f443 ok
security-group:sg-07b79756088cf753c     still has dependencies, will retry
security-group:sg-07ef00367ce1a7b62     still has dependencies, will retry
security-group:sg-005c9b49b63793004     still has dependencies, will retry
Not all resources deleted; waiting before reattempting deletion
        dhcp-options:dopt-0a0be88814a0c83a9
        volume:vol-074914e494e4b656d
        security-group:sg-01f81918cdbaba212
        vpc:vpc-0775d4b463932d2f7
        route-table:rtb-06b591f24a01973f6
        internet-gateway:igw-0a939d66d6e93e0d5
        security-group:sg-005c9b49b63793004
        subnet:subnet-060ec2db19027cf6a
        security-group:sg-07b79756088cf753c
        subnet:subnet-0def003bdbfd97915
        security-group:sg-07ef00367ce1a7b62
internet-gateway:igw-0a939d66d6e93e0d5  still has dependencies, will retry
volume:vol-074914e494e4b656d    ok
subnet:subnet-060ec2db19027cf6a still has dependencies, will retry
security-group:sg-01f81918cdbaba212     still has dependencies, will retry
security-group:sg-005c9b49b63793004     still has dependencies, will retry
security-group:sg-07b79756088cf753c     still has dependencies, will retry
subnet:subnet-0def003bdbfd97915 ok
security-group:sg-07ef00367ce1a7b62     ok
Not all resources deleted; waiting before reattempting deletion
        security-group:sg-01f81918cdbaba212
        vpc:vpc-0775d4b463932d2f7
        route-table:rtb-06b591f24a01973f6
        internet-gateway:igw-0a939d66d6e93e0d5
        security-group:sg-005c9b49b63793004
        subnet:subnet-060ec2db19027cf6a
        security-group:sg-07b79756088cf753c
        dhcp-options:dopt-0a0be88814a0c83a9
subnet:subnet-060ec2db19027cf6a still has dependencies, will retry
security-group:sg-005c9b49b63793004     still has dependencies, will retry
security-group:sg-07b79756088cf753c     still has dependencies, will retry
internet-gateway:igw-0a939d66d6e93e0d5  ok
security-group:sg-01f81918cdbaba212     ok
Not all resources deleted; waiting before reattempting deletion
        dhcp-options:dopt-0a0be88814a0c83a9
        vpc:vpc-0775d4b463932d2f7
        route-table:rtb-06b591f24a01973f6
        security-group:sg-005c9b49b63793004
        subnet:subnet-060ec2db19027cf6a
        security-group:sg-07b79756088cf753c
subnet:subnet-060ec2db19027cf6a ok
security-group:sg-005c9b49b63793004     ok
security-group:sg-07b79756088cf753c     ok
route-table:rtb-06b591f24a01973f6       ok
vpc:vpc-0775d4b463932d2f7       ok
dhcp-options:dopt-0a0be88814a0c83a9     ok
Deleted kubectl config for fleetman.k8s.local

Deleted cluster: "fleetman.k8s.local"
```
### Restarting the Cluster

```
[ec2-user@foobar ~]$ history|grep export
    4  export NAME=fleetman.k8s.local
    6  export KOPS_STATE_STORE=s3://stanzhou-state-storage

kops create cluster --zones ap-southeast-2a,ap-southeast-2b,ap-southeast-2c ${NAME}

kops edit ig --name=fleetman.k8s.local nodes
kops update cluster ${NAME} --yes
kops validate cluster
kubectl apply -f .
kubectl get svc
kubectl describe svc fleetman-webapp
```
# Logging a Cluster

## Introducing the ELK/ElasticStack
![ELK Stack](https://i.imgur.com/YRocsbG.png)
![ELK Stack2](https://i.imgur.com/VZihLCY.png)
```
[ec2-user@ip-172-31-4-9 ~]$ kubectl get po
NAME                                  READY   STATUS    RESTARTS   AGE
api-gateway-5d445d6f69-kg68n          1/1     Running   0          19h
mongodb-5559556bf-ghq5p               1/1     Running   0          19h
position-simulator-7ffd4f8f68-kwlcb   1/1     Running   0          19h
position-tracker-5ff4fb7479-2ctzt     1/1     Running   0          19h
queue-75f4ddd795-wp2mh                1/1     Running   0          10h
webapp-689dd9b4f4-php92               1/1     Running   0          19h
webapp-689dd9b4f4-vgtpg               1/1     Running   0          19h
[ec2-user@ip-172-31-4-9 ~]$ kubectl get svc
NAME                        TYPE           CLUSTER-IP       EXTERNAL-IP                                                                    PORT(S)              AGE
fleetman-api-gateway        ClusterIP      100.67.195.111   <none>                                                                         8080/TCP             20h
fleetman-mongodb            ClusterIP      100.68.143.136   <none>                                                                         27017/TCP            20h
fleetman-position-tracker   ClusterIP      100.69.119.23    <none>                                                                         8080/TCP             20h
fleetman-queue              ClusterIP      100.65.16.172    <none>                                                                         8161/TCP,61616/TCP   20h
fleetman-webapp             LoadBalancer   100.70.168.148   ac185d1638c0311e9bef7028ed3b83c9-1696162465.ap-southeast-2.elb.amazonaws.com   80:31504/TCP         20h
kubernetes                  ClusterIP      100.64.0.1       <none>                                                                         443/TCP              20h
[ec2-user@ip-172-31-4-9 ~]$ kubectl get svc -n kube-system
NAME                    TYPE           CLUSTER-IP       EXTERNAL-IP                                                                   PORT(S)          AGE
elasticsearch-logging   ClusterIP      100.68.193.244   <none>                                                                        9200/TCP         19h
kibana-logging          LoadBalancer   100.67.153.183   a3c20f00a8c0811e9bef7028ed3b83c9-306741856.ap-southeast-2.elb.amazonaws.com   5601:32716/TCP   19h
kube-dns                ClusterIP      100.64.0.10      <none>                                                                        53/UDP,53/TCP    20h
[ec2-user@ip-172-31-4-9 ~]$ kubectl describe svc kibana-logging -n kube-system
Name:                     kibana-logging
Namespace:                kube-system
Labels:                   addonmanager.kubernetes.io/mode=Reconcile
                          k8s-app=kibana-logging
                          kubernetes.io/cluster-service=true
                          kubernetes.io/name=Kibana
Annotations:              kubectl.kubernetes.io/last-applied-configuration:
                            {"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"labels":{"addonmanager.kubernetes.io/mode":"Reconcile","k8s-app":"kibana...
Selector:                 k8s-app=kibana-logging
Type:                     LoadBalancer
IP:                       100.67.153.183
LoadBalancer Ingress:     a3c20f00a8c0811e9bef7028ed3b83c9-306741856.ap-southeast-2.elb.amazonaws.com
Port:                     <unset>  5601/TCP
TargetPort:               ui/TCP
NodePort:                 <unset>  32716/TCP
Endpoints:                100.96.1.5:5601
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

# MONITORING WITH PROMETHEUS AND GRAFANA
## Monitoring a Cluster
[Prometheus](https://prometheus.io/)
[Grafana](https://grafana.com)
Concerntrate solely on integrating Grafana with Prometheus.

## Helm Package Manager
The Kubernetes package manager [Helm](https://helm.sh)

Install Helm:
```
wget https://get.helm.sh/helm-v2.14.1-linux-amd64.tar.gz
tar zxvf helm-v2.14.1-linux-amd64.tar.gz
sudo mv linux-amd64/helm /usr/local/bin/
rm helm-v2.14.1-linux-amd64.tar.gz
rm -rf ./linux-amd64/
helm --help
helm version
helm init

kubectl create serviceaccount --namespace kube-system tiller

kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller

kubectl patch deploy --namespace kube-system tiller-deploy -p '{"spec":{"template":{"spec":{"serviceAccount":"tiller"}}}}'
```

Delete Helm:
```
helm ls
helm delete --purge my-special-installation
kubectl get po
```
## Installing Prometheus Operator

[install](https://github.com/helm/charts/tree/master/stable/prometheus-operator)

`helm install --name monitoring --namespace monitoring stable/prometheus-operator`

```
[ec2-user@ip-172-31-4-9 ~]$ kubectl get all -n monitoring
NAME                                                         READY   STATUS    RESTARTS   AGE
pod/alertmanager-monitoring-prometheus-oper-alertmanager-0   2/2     Running   0          4m
pod/monitoring-grafana-c768bb86f-cgmj8                       2/2     Running   0          4m27s
pod/monitoring-kube-state-metrics-6488587c6-zrjmg            1/1     Running   0          4m27s
pod/monitoring-prometheus-node-exporter-5cp7v                1/1     Running   0          4m27s
pod/monitoring-prometheus-node-exporter-75v99                1/1     Running   0          4m27s
pod/monitoring-prometheus-node-exporter-jcl7r                1/1     Running   0          4m27s
pod/monitoring-prometheus-node-exporter-vptns                1/1     Running   0          4m27s
pod/monitoring-prometheus-oper-operator-7b54f56766-j8k6c     1/1     Running   0          4m27s
pod/prometheus-monitoring-prometheus-oper-prometheus-0       3/3     Running   1          3m52s

NAME                                              TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)             AGE
service/alertmanager-operated                     ClusterIP   None             <none>        9093/TCP,6783/TCP   4m
service/monitoring-grafana                        ClusterIP   100.70.116.236   <none>        80/TCP              4m28s
service/monitoring-kube-state-metrics             ClusterIP   100.67.183.154   <none>        8080/TCP            4m28s
service/monitoring-prometheus-node-exporter       ClusterIP   100.68.145.110   <none>        9100/TCP            4m28s
service/monitoring-prometheus-oper-alertmanager   ClusterIP   100.70.75.68     <none>        9093/TCP            4m28s
service/monitoring-prometheus-oper-operator       ClusterIP   100.66.237.147   <none>        8080/TCP            4m28s
service/monitoring-prometheus-oper-prometheus     ClusterIP   100.67.205.22    <none>        9090/TCP            4m28s
service/prometheus-operated                       ClusterIP   None             <none>        9090/TCP            3m53s

NAME                                                 DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
daemonset.apps/monitoring-prometheus-node-exporter   4         4         4       4            4           <none>          4m27s

NAME                                                  DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/monitoring-grafana                    1         1         1            1           4m27s
deployment.apps/monitoring-kube-state-metrics         1         1         1            1           4m27s
deployment.apps/monitoring-prometheus-oper-operator   1         1         1            1           4m27s

NAME                                                             DESIRED   CURRENT   READY   AGE
replicaset.apps/monitoring-grafana-c768bb86f                     1         1         1       4m27s
replicaset.apps/monitoring-kube-state-metrics-6488587c6          1         1         1       4m27s
replicaset.apps/monitoring-prometheus-oper-operator-7b54f56766   1         1         1       4m27s

NAME                                                                    DESIRED   CURRENT   AGE
statefulset.apps/alertmanager-monitoring-prometheus-oper-alertmanager   1         1         4m
statefulset.apps/prometheus-monitoring-prometheus-oper-prometheus       1         1         3m53s

[ec2-user@ip-172-31-4-9 ~]$ kubectl edit -n monitoring service/monitoring-prometheus-oper-prometheus

Change type from ClusterIP to LoadBalancer

[ec2-user@ip-172-31-4-9 ~]$ kubectl get all -n monitoring
NAME                                                         READY   STATUS    RESTARTS   AGE
pod/alertmanager-monitoring-prometheus-oper-alertmanager-0   2/2     Running   0          16m
pod/monitoring-grafana-c768bb86f-cgmj8                       2/2     Running   0          17m
pod/monitoring-kube-state-metrics-6488587c6-zrjmg            1/1     Running   0          17m
pod/monitoring-prometheus-node-exporter-5cp7v                1/1     Running   0          17m
pod/monitoring-prometheus-node-exporter-75v99                1/1     Running   0          17m
pod/monitoring-prometheus-node-exporter-jcl7r                1/1     Running   0          17m
pod/monitoring-prometheus-node-exporter-vptns                1/1     Running   0          17m
pod/monitoring-prometheus-oper-operator-7b54f56766-j8k6c     1/1     Running   0          17m
pod/prometheus-monitoring-prometheus-oper-prometheus-0       3/3     Running   1          16m

NAME                                              TYPE           CLUSTER-IP       EXTERNAL-IP                                                                    PORT(S)             AGE
service/alertmanager-operated                     ClusterIP      None             <none>                                                                         9093/TCP,6783/TCP   16m
service/monitoring-grafana                        ClusterIP      100.70.116.236   <none>                                                                         80/TCP              17m
service/monitoring-kube-state-metrics             ClusterIP      100.67.183.154   <none>                                                                         8080/TCP            17m
service/monitoring-prometheus-node-exporter       ClusterIP      100.68.145.110   <none>                                                                         9100/TCP            17m
service/monitoring-prometheus-oper-alertmanager   ClusterIP      100.70.75.68     <none>                                                                         9093/TCP            17m
service/monitoring-prometheus-oper-operator       ClusterIP      100.66.237.147   <none>                                                                         8080/TCP            17m
service/monitoring-prometheus-oper-prometheus     LoadBalancer   100.67.205.22    a86903ccd8ce211e9bef7028ed3b83c9-1698452662.ap-southeast-2.elb.amazonaws.com   9090:32096/TCP      17m
service/prometheus-operated                       ClusterIP      None             <none>                                                                         9090/TCP            16m

NAME                                                 DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
daemonset.apps/monitoring-prometheus-node-exporter   4         4         4       4            4           <none>          17m

NAME                                                  DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/monitoring-grafana                    1         1         1            1           17m
deployment.apps/monitoring-kube-state-metrics         1         1         1            1           17m
deployment.apps/monitoring-prometheus-oper-operator   1         1         1            1           17m

NAME                                                             DESIRED   CURRENT   READY   AGE
replicaset.apps/monitoring-grafana-c768bb86f                     1         1         1       17m
replicaset.apps/monitoring-kube-state-metrics-6488587c6          1         1         1       17m
replicaset.apps/monitoring-prometheus-oper-operator-7b54f56766   1         1         1       17m

NAME                                                                    DESIRED   CURRENT   AGE
statefulset.apps/alertmanager-monitoring-prometheus-oper-alertmanager   1         1         16m
statefulset.apps/prometheus-monitoring-prometheus-oper-prometheus       1         1         16m
```

## Working with Grafana
change back from LoadBalancer to ClusterIP on service/monitoring-prometheus-oper-prometheus
change from ClusterIP to LoadBalancer on service/monitoring-grafana 


# The Alert Manager
## Alerting

