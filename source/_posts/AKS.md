---
title: AKS
date: 2021-02-24 20:39:06
tags: AKS
---
# 1. K8s on Azure

## Deployment models
![architecure](https://i.imgur.com/XQNAABL.png "architecture")

## Create the Azure Container Registry
```
az group create --name myResourceGroup --location australiaeast
az acr create --resource-group myResourceGroup --name cwzhou --sku Basic
```

## Push a container to the registry
### Login to your ACR instance:
```
az acr login --name akscourse
```

### Verify that you have a local copy of your application image:
```
docker images
```

If you don't have an application, you can quickly build your own mini-application image (see the app_example/README.md file):
docker build app_example/ -t hostname:v1

We need to tag the image with the registry login server address which we can get with:
```
export aLS=`az acr list --resource-group myResourceGroup --query "[].{acrLoginServer:loginServer}" --output tsv`
```

### Tag your nginx image with the server address and verion (v1 in this case):
```
docker tag hostname:v1 ${aLS}/hostname:v1
```

### Verify that your tags have been applied:
```
docker images
```

Push the image to the Registry:
```
docker push ${aLS}/hostname:v1
```

Verify that the image has been pushed correctly:
```
az acr repository list --name cwzhou --output tsv
repository=<repository_output>
```

You can verify that the image is appropriately tagged with (repository is the output of the previous step):
```
az acr repository show-tags --name cwzhou --repository ${repository} --output tsv
```

### Verify container registry image
```
docker rmi <registry/image:version>
docker pull <registry/image:version>

It should also then be possible to run the image locally to verify operation:

docker run --rm --name hostname -p 8080:80 <registry/image:version>
curl localhost:8080
docker stop hostname
docker rmi <registry/image:version>

```

### Establish AKS specific credentials
To allow an AKS cluster to interact with other Azure resources such as the Azure Container Registry we created in a previous chapter, an Azure Active Directory (ad) service principal is used. To create the service principal:
```
az ad sp create-for-rbac --skip-assignment
```

Make a note of the appId and password, you will need these. Better yet, save this credential somewhere secure.

Get the ACR resource ID:
```
az acr show --resource-group myResourceGroup --name cwzhou --query "id" --output tsv
```

Create a role assignment:

```
az role assignment create --assignee <appId> --scope <acrId> --role Reader
```

It is also possible to integrate AKS with Azure Active Directory in a much deeper fashion, which may be appropriate in an enterprise environment. Further instructions can be found here:

https://docs.microsoft.com/en-us/azure/aks/aad-integration

### Launch an AKS cluster
```
Finally we can create an AKS cluster (you will need your <appId> and <password> from the previous section):

cat ../sp.txt

az aks create \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --node-count 1 \
    --max-pods 20 \
    --kubernetes-version 1.12.4 \
    --generate-ssh-keys \
    --enable-vmss \
    --enable-cluster-autoscaler \
    --min-count 1 \
    --max-count 3 \
    --service-principal <appId> --client-secret <password> 

This will create a cluster (which may take 5-10 minutes). Once done, we can connect to the kubernetes environment via the Kubernetes CLI. If you are using the Azure Cloud Shell, the kubernetes client (kubectl) is already installed. You can also install locally if you haven't previously installed a version of kubectl:
az aks install-cli

az aks get-credentials --resource-group myResourceGroup --name myAKSCluster --admin

The resource-group and name were set during the creation process, and you should use those if different from what we're using here.

Check your connection and that the kubernetes cli is working with:
kubectl get nodes

If you have issues with connecting, one thing you can check is the version of your kubernetes client:
kubectl version

This will tell you both the local client, and the configured kubernetes service version, make sure the client is at least the same if not newer than the server.
```
