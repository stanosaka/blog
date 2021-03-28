---
title: kubernetes on the cloud
date: 2019-02-01 20:39:06
tags: 
  - k8s
  - GKE
  - devops
---
# Kubernetes on the GKE
**use cloud shell**
```
#setup zone
gcloud config set compute/zone australia-southeast1-a
#setup region
gcloud config set compute/region australia-southeast1
# create a single node cluster
gcloud container clusters create my-first-cluster --num-nodes 1
# see the list of clusters
gcloud compute instances list
# deploy a wordpress Docker container to our single node cluster
# --image=tutum/wordpress: An out-fo-the-box image which includes everything you need
# to run this site-including a MySQL database
kubectl run wordpress --image=tutum/wordpress --port=80
# see the status of pods running
stancloud9@cloudshell:~ (scenic-torch-250909)$ kubectl get pods
NAME                         READY   STATUS    RESTARTS   AGE
wordpress-7fff795d48-kzbfr   1/1     Running   0          5m36s
```
By default a pod is accessible to only other internal machines in the cluster
```
#kubectl expose pod: Exposed the pod a a service so it can be accessed externally
#--type=LoadBalancer: Creates an external IP that this pod can use to accept traffic
kubectl expose pod wordpress-7fff795d48-kzbfr --name=wordpress --type=LoadBalancer
# check 
stancloud9@cloudshell:~ (scenic-torch-250909)$ kubectl describe svc wordpress
Name:                     wordpress
Namespace:                default
Labels:                   pod-template-hash=7fff795d48
                          run=wordpress
Annotations:              <none>
Selector:                 pod-template-hash=7fff795d48,run=wordpress
Type:                     LoadBalancer
IP:                       10.27.245.246
LoadBalancer Ingress:     35.189.22.57
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  31344/TCP
Endpoints:                10.24.0.11:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:
  Type    Reason                Age    From                Message
  ----    ------                ----   ----                -------
  Normal  EnsuringLoadBalancer  2m17s  service-controller  Ensuring load balancer
  Normal  EnsuredLoadBalancer   81s    service-controller  Ensured load balancer
```
**How Kubernetes Works**
Kubernetes: Orchestration technology- convert isolated containers running on different hardware into a cluster

Pod: Atomic unit of deployment in Kubernetes.
- Consists of 1 or more tightly coupled containers.
- Pod runs on node, which is controlled by master.
- Kubernetes only knows about pods
- Cannot start container without a pod
- Pod => Sandbox for 1 or more cntainers

![pods](https://i.imgur.com/m8uoP3H.png)
Kubernetes::Hadoop
- Docker Container Engine -> Java Runtime
- Docker containers -> Jars
- Kubernetes -> Hadoop

**Kubernetes for Orchestration**
- Fault-tolerance: Pod/Nod failures
- Auto-scaling: More clients? More demand
- Rollback: Advanced deployment options
- Auto-healing: Crashed containers restart
- Load-balancing: Distribute client requests
- Isolation: Sandboxes so that containers don't interfere

**kubectl**
![kubectl](https://i.imgur.com/GryAqTK.png)
Telling k8s what the desire state is.

**What does the k8s master do?**
Kubernetes Master
- One or more nodes designated as master
- Several k8s processes run on master
- Multi-master for high-availability

**kube-apiserver**
- Communicates with user
- RESTful API end-points
- Manifest yaml files are accepted by apiserver
![apiserver](https://i.imgur.com/GA0zKFs.png)
**etcd**
Cluster Store for Metadata
- Metadata about spec and status of cluster
- etcd is consistent and highly available key-value store
- source-of-truth for cluster state
![etcd](https://i.imgur.com/kQQe478.png)

**kube-scheduler**
- Handle pod creation and management
- Kube-scheduler match/assign nodes to pods
- Complex-affinities, taints, tolerations,...
![scheduler](https://i.imgur.com/erz1Rpw.png)

**controller-manager**
- Different master processes
- Actual state <-> Desired State
- cloud-controller-manager
- kube-controller-manager
- Node controller
- Replication controller
- Route controller
- Volume controller
![controller](https://i.imgur.com/iQpH2WD.png)

**Control Plane**
- Apiserver
- etc
- scheduler
- controller-manager
   - cloud-controller-manager
   - kube-controller-manager
![control plane](https://i.imgur.com/ql9VYxt.png)

**What runs on each node of a cluster?**
Kubernetes Node (Minion)
- Kubelet
  - Agent running on this node
  - Listen to k8s master
  - Port 10255
- Kube-proxy
  - Needed because pod IP addresses are ephemeral
  - Networking- will make sense when we discuss Service objects
- Container engine
  - Works with kubelet
  - Pulling images
  - Start/stop
  - Could be Docker or rkt
![Minion](https://i.imgur.com/lXOojJr.png)
![Pod Creation Request](https://i.imgur.com/dft7Zwy.png)

**What are Pods?**
![pod creation yaml](https://i.imgur.com/5qt1aB6.png)
Multi-Container Pods
- Share access to memory space
- Connect to each other using localhost
- Share access to the same volumes (storage abstraction)
- Same parameters such as configMaps
- Tight coupling is dangerous
- Once crashes, all crash

**Use cases for multi-container Pod**
- Main container, "sidecar" supporting containers
![main container](https://i.imgur.com/ollFuAy.png)
- Proxies, Bridges, Adapters
![Proxies](https://i.imgur.com/kan5QHv.png)

**Anti-Patterns for multi-container pods**
- Avoid packing three-tier web app into 1 pod
- Do not pack multiple similar containers  in the same pod for scaling
- Use Deployments or ReplicaSet instead
- Micro0services: Simple, independent components

**Pods limitations**
- No auto-healing or scaling
- Pod crashes? Must be handled at higher level
   - ReplicaSet,Deployment,Service
- Ephemeral: Ip address are ephemeral
**Higer level k8s objects**
- ReplicaSet, ReplicationController: Scaling and healing
- Deployment: Versioning and rollback
- Service: Static (non-ephemeral) IP and networking
- Volume: Non-ephemeral storage
## How do master nodes communicate?
**Cluster to master**
- All cluster -> master communication only with apiserver
- HTTPS (443)
- Relatively secure
**Master to cluster**
- apiserver->kubelet
   - Certicate not verified by default
   - Vulnerable to man-in-the-middle attacks
   - Don't run on public network
   - To harden
     - set -kubelet-certificate-authority
     - use SSH tunelling
- apiserver->nodes/pods/services
   - Not safe
   - Plain HTTP
   - Neither authenticated for encrypted
   - On public clouds, SSH tunnelling provided by cloud provider e.g. GCP
## Where can we run Kubernetes?
Public Cloud: 
- aws(kops)
- azure 
- gcp(kubectl)
   - GKE:Google Kubernetes Engine
   - Infra on GCE Virtual machines
   - GCE: Google Compute Engine (IaaS)
Bootstrap(kubeadm): On-prem, private cloud
Playgrounds:PWK(Browser-based,time-limited sessions), Minikube(Windows or Mac, Sets up VM on your machine)

## Can K8s be used in a hybird, multi-cloud world?
Hybrid=On-prem+Public Cloud
  - On-premise: vare metal or VMs
  - Legacy infra, large on-prem datacenters
  - Medium-term importance

Multi-Cloud
  - More than 1 public cloud provider
  - Strategic reasons, vendor lock-in (Amazon buying Whole Foods)

## Interacting with K8s
**How do we work with k8s**
- kubectl
  - Most common command line utility
  - Make POST requests to apiserver of control plane
- kubeadm
  - Bootstrap cluster when not on cloud Kubernetes service
  - To create cluster out of individual infra nodes
- kubefed
  - Administer federated clusters
  - Federated cluster-> group of multiple clusters (multi-cloud,hybrid)
- kubelet
- kube-proxy

**Objects**
- K8s objects are persistent entities
- Everything is an object...
- Pod,ReplicaSet, Deployment, Node ... all are objects
- Send object specification (usually in .yaml or .json)

**Three object management methods**
- Imperative commands
  - No .yaml or config files
  - kubectl run ...
  - kubectl expose ...
  - kubectl autosacle ...
```
kubectl run ningx --image nginx
kubectl create deployment nginx --image nginx
```
No config file

Imperative: intent is in command

Pro:
- Simple
Cons:
- No audit trail or review mechanism
- Can't reuse or use in template


- Imperative object configuration
  - kubectl + yaml or config files used
  - kubectl create -f config.yaml
  - kubectl replace -f config.yaml
  - kubectl delete -f config.yaml

Config file required
Still Imperative: intent is in command

Pros:
- Still simple
- Robust - files checked into repo
- One file for multiple operations


- Declarative object configuration
  - Only .yaml or config files used
  - kubectl apply -f config.yaml
```
kubectl apply -f configs/
```
Config files(s) are all that is required
Declarative not imperative

Pros:
- Most robust-review, repos, audit trails
- K8S will automatically figure out intents
- Can specify multiple files/directories recursively

- Live object configuration: The live configuration values of an object, as observed by the Kubernetes cluster
- Current object configuration files: The config file we are applying in the current command
- List-applied object configuration file: The last config file what was applied to the object

*Don't mix and match!*
*Declarative is preferred*

**Merging Changes**
- Primitive fields
  - String, init, boolean, images or replicas
  - Replace old state with Current object configuration file
- Map fields
  - Merge old state with Current object configuration file
- List field
  - Complex-varies by field

**The Pros and Cons of Declarative and Imperative object management**
Declarative

kubectl apply -f config.yaml
- Robust
- Track in repos, review
- Recursively apply to directories

Imperative
kubectl run ...
kubectl expose...
kubectl autosacle...
- Simple, intention is very clear
- Preferred for deletion

## Names and UIDs
**How are objects named?**
- Objects-persistent entities
  - pods,replicasets,services,volumes,nodes,...
  - information maintained in etcd
- Identified using
  - names: client-given
  - UIDs: system-generated
## Namespaces
- Divide physical cluster into multiple virtual clusters
- Three pre-defined namespaces:
  - default: if non specified
  - kube-system: for internal k8s objects
  - kub-public: auto-readable by all users
- Name scopes: names need to be unique only inside a namespace
- Future version: namespace-> common access control
- Don't use namespaces for versioning
  - Just use labels instead

**Objects without namespaces**
- Nodes
- PersistentVolumes
- Namespace themselves

## Labels
- key/value pairs attached to objects
- metadata
- need not be unique (same label to mutiple objects)
- No sematics for k8s (meaningful only to humans)
- Loose coupling via selectors
- Can be added
  - at creation time
  - after creation
- Multiple objects can have same label
- Same label can't repeat key though

## Volumes
- Permanence: Storage that lasts longer than lifetime of a pod
  - a container inside pod (true for all volumes)
  - a pod (only true for persistent volumes)
- Shared State: multiple containers in a pod need to share state/files
- Volumes: address both these needs
- Volumes (in general): lifetime of abstraction=lifetime of pod
  - Note that this is longer than lifetime of any container inside pod
- Persistent Volumes: lifetime of abstraction independent of pod lifetime
![using volume](https://i.imgur.com/0DJ10wq.png)
**Types of Volumes**
- awsElasticBlockStore
- azureDisk
- azureFile
- gcePersistentDisk
- Many non-cloud-specific volume types as well
**Important type of volumes**
- configMap
- emptyDir
- gitRepo
- secret
- hostPath
## Lab: Volumes and the exmptydir volume
```
stancloud9@cloudshell:~ (scenic-torch-250909)$ cat pod-redis.yaml
apiVersion: v1
kind: Pod
metadata:
   name: redis
spec:
   containers:
   - name: redis
     image: redis
     volumeMounts:
     - name: redis-storage
       mountPath: /data/redis
   volumes:
   - name: redis-storage
     emptyDir: {}
stancloud9@cloudshell:~ (scenic-torch-250909)$ k get pod redis --watch        NAME    READY   STATUS    RESTARTS   AGE
redis   0/1     Pending   0          7m36s
redis   0/1   Pending   0     7m36s
redis   0/1   ContainerCreating   0     7m36s
redis   1/1   Running   0     7m40s
^Cstancloud9@cloudshell:~ (scenic-torch-250909)$ k exec -it redis -- /bin/bash
root@redis:/data# cd /data/redis
root@redis:/data/redis# echo Hello > test-file
root@redis:/data/redis# kill 1
root@redis:/data/redis# command terminated with exit code 137
stancloud9@cloudshell:~ (scenic-torch-250909)$ k get pod redis --watch
NAME    READY   STATUS    RESTARTS   AGE
redis   1/1     Running   1          18m
^Cstancloud9@cloudshell:~ (scenic-torch-250909)$ k exec -it redis -- /bin/bash
root@redis:/data# ls /data/redis
test-file
```
## Persistent Volumes
- Low-level objects, like nodes
- Two types of provisioning:
 - Static: administrator pre-creates the volumes
 - Dynamic: containers need to file a PersistentVolumeClaim
![Cloud specific persistent volumes](https://i.imgur.com/Yk4bu0J.png)
![persisten volumes](https://i.imgur.com/KnjqWkU.png)
```
gcloud compute disks create my-disk-1 --zone australia-southeast1-a
gcloud compute disks list
stancloud9@cloudshell:~ (scenic-torch-250909)$ cat volum-sample.yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pd
spec:
  containers:
  - image: k8s.gcr.io/test-webserver
    name: test-container
    volumeMounts:
    - mountPath: /test-pd
      name: test-volume
  volumes:
  - name: test-volume
    gcePersistentDisk:
      pdName: my-disk-1
      fsType: ext4
k create -f volum-sample.yaml --validate=false
pod/test-pd created
stancloud9@cloudshell:~ (scenic-torch-250909)$ k get pod test-pd --watch
NAME      READY   STATUS              RESTARTS   AGE
test-pd   0/1     ContainerCreating   0          23s
test-pd   1/1   Running   0     29s
^Cstancloud9@cloudshell:~ (scenic-torch-250909)$ k describe pod test-pd
Name:               test-pd
Namespace:          default
Priority:           0
PriorityClassName:  <none>
Node:               gke-my-first-cluster-default-pool-8ca613e9-w13m/10.152.0.2
Start Time:         Sat, 31 Aug 2019 11:39:22 +1000
Labels:             <none>
Annotations:        kubernetes.io/limit-ranger: LimitRanger plugin set: cpu request for container test-container
Status:             Running
IP:                 10.24.0.22
Containers:
  test-container:
    Container ID:   docker://8b08b4cd144d256a078fe3c7b4031844e2c3355d854a711ec454611c0edea30b
    Image:          k8s.gcr.io/test-webserver
    Image ID:       docker-pullable://k8s.gcr.io/test-webserver@sha256:f63e365c13646f231ec4a16791c6133ddd7b80fcd1947f41ab193968e02b0745
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Sat, 31 Aug 2019 11:39:50 +1000
    Ready:          True
    Restart Count:  0
    Requests:
      cpu:        100m
    Environment:  <none>
    Mounts:
      /test-pd from test-volume (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-gw57h (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  test-volume:
    Type:       GCEPersistentDisk (a Persistent Disk resource in Google Compute Engine)
    PDName:     my-disk-1
    FSType:     ext4
    Partition:  0
    ReadOnly:   false
  default-token-gw57h:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-gw57h
    Optional:    false
QoS Class:       Burstable
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason                  Age    From                                                      Message
  ----    ------                  ----   ----                                                      -------
  Normal  Scheduled               2m15s  default-scheduler                                         Successfully assigned default/test-pd to gke-my-first-cluster-default-pool-8ca613e9-w13m
  Normal  SuccessfulAttachVolume  2m9s   attachdetach-controller                                   AttachVolume.Attach succeeded for volume "test-volume"
  Normal  Pulling                 109s   kubelet, gke-my-first-cluster-default-pool-8ca613e9-w13m  pulling image "k8s.gcr.io/test-webserver"
  Normal  Pulled                  107s   kubelet, gke-my-first-cluster-default-pool-8ca613e9-w13m  Successfully pulled image "k8s.gcr.io/test-webserver"
  Normal  Created                 107s   kubelet, gke-my-first-cluster-default-pool-8ca613e9-w13m  Created container
  Normal  Started                 107s   kubelet, gke-my-first-cluster-default-pool-8ca613e9-w13m  Started container
```
**What are some important types of volumes?**
- emptyDir
  - Not persistent
  - Created when pod is created on node
  - Initially empty
  - Share space/state across containers in same pod
  - Containers acan mount at different times
  - When pod removed/crashes, data lost
  - When container carshes data remains
  - Usecases: Scratch space, checkpointing...
![emptyDir](https://i.imgur.com/GGAyCl0.png)

- hostPath
![hostPath](https://i.imgur.com/pzfvzAb.png)
  - Mount file/directory from node filesystem into pod
  - Uncommon- pods should be independent of nodes
  - Makes pod-node coupling tight
  - Usecases: Access docker internals, running cAdvisor
  - Block devices or sockets on host

- gitRepo
![gitRepo](https://i.imgur.com/EPHpRzS.png)

- configMap
![conigMap](https://i.imgur.com/GkC7kjP.png)
  - configMap volume mounts data from ConfigMap object
  - configMap objects define key-value pairs
  - configMap objects inject paramters into pods
  - Two main usecases:
    - Providing config information for apps running inside pods
    - Specifying config information for control plane (controllers)

- secret
  - Pass ensitive inforamtion to pods
  - First create secret so it is stored in control plane
    - kubectl create secret
  - Mount that secret as a volume so that it is available inside pod
  - Secret is stored in RAM storage (not written to persistent disk)
## Lab: use of secrets pass information to pods
```
stancloud9@cloudshell:~ (scenic-torch-250909)$ cat secrets.yaml
apiVersion: v1
kind: Secret
metadata:
  name: test-secret
data:
  username: bXktYMxw
  password: Mzk1MjqkdmRnN0pi
stancloud9@cloudshell:~ (scenic-torch-250909)$ cat secrets-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-test-pod
spec:
  containers:
    - name: test-container
      image: nginx
      volumeMounts:
         # name must match the volume name below
         - name: secret-volume
           mountPath: /etc/secret-volume
  # The secret data is exposed to Containers in the Pod through a Volume.
  volumes:
    - name: secret-volume
      secret:
        secretName: test-secret
stancloud9@cloudshell:~ (scenic-torch-250909)$ k get pod secret-test-pod
NAME              READY   STATUS    RESTARTS   AGE
secret-test-pod   1/1     Running   0          19m
stancloud9@cloudshell:~ (scenic-torch-250909)$ k exec -it secret-test-pod -- /bin/bash
root@secret-test-pod:/# cd /etc/secret-volume/
root@secret-test-pod:/etc/secret-volume# ls
password  username
```


# Debugging
## Insufficient cpu
```
stancloud9@cloudshell:~ (scenic-torch-250909)$ k get pod secret-test-pod
NAME              READY   STATUS    RESTARTS   AGE
secret-test-pod   0/1     Pending   0          51s
stancloud9@cloudshell:~ (scenic-torch-250909)$ k describe pod secret-test-pod
Name:               secret-test-pod
Namespace:          default
Priority:           0
PriorityClassName:  <none>
Node:               <none>
Labels:             <none>
Annotations:        kubernetes.io/limit-ranger: LimitRanger plugin set: cpu request for container test-container
Status:             Pending
IP:
Containers:
  test-container:
    Image:      nginx
    Port:       <none>
    Host Port:  <none>
    Requests:
      cpu:        100m
    Environment:  <none>
    Mounts:
      /etc/secret-volume from secret-volume (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-gw57h (ro)
Conditions:
  Type           Status
  PodScheduled   False
Volumes:
  secret-volume:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  test-secret
    Optional:    false
  default-token-gw57h:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-gw57h
    Optional:    false
QoS Class:       Burstable
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type     Reason            Age                  From               Message
  ----     ------            ----                 ----               -------
  Warning  FailedScheduling  11s (x8 over 4m25s)  default-scheduler  0/1 nodes are available: 1 Insufficient cpu.

stancloud9@cloudshell:~ (scenic-torch-250909)$ gcloud container clusters resize my-first-cluster --num-nodes=3 --zone australia-southeast1-a
Pool [default-pool] for [my-first-cluster] will be resized to 3.

Do you want to continue (Y/n)?  Y

Resizing my-first-cluster...done.
Updated [https://container.googleapis.com/v1/projects/scenic-torch-250909/zones/australia-southeast1-a/clusters/my-first-cluster].
stancloud9@cloudshell:~ (scenic-torch-250909)$ k get pod secret-test-pod
NAME              READY   STATUS    RESTARTS   AGE
secret-test-pod   1/1     Running   0          19m
```
