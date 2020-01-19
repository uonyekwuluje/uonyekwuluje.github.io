---
layout: post
title:  "Kubernetes and Docker on Azure (AKS) and Azure (ACR)"
categories: Containers, Kubernetes, Docker
---

The realms and domain of Software Development,  Application Development, Systems Development, Web Development etc has gone through lots
of transformations over the years. Two of the staples of this transformation are Kubernetes and docker. The beauty of this is that you
now have the tools at your disposal to build and possibliy architect your apps and tools at scale. 

#### **Requirements**
Ensure the following requirements are in place before you begin:

* A valid Azure Subscription
* The subscription ID of the given subscription
* Permissions to perform administrative tasks in the given subscription 
* Local installation of [docker](https://docs.docker.com/install)
* Local installation of [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl)
* Local installation of [helm](https://helm.sh/docs/intro/install)
* Local installation of azure [cli](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)
* Valid SSH Key Pair
* Valid service principal 

<hr>
## **Setup**
Setup the following environmental variables
```
PREFIX="DevK8ACR"
LOCATION="eastus"
SUBSCRIPTION_ID="xxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
RESOURCE_GROUP_NAME="${PREFIX}-rg"
AKS_CLUSTER_NAME="${PREFIX}Cluster"
ACR_REGISTRY_NAME="${PREFIX}Registry"
KUBERNETES_VERSION="1.14.8"
NODE_POOL_NAME="pythonpool1"
NODE_POOL_SKU="Standard_DS2_v2"
CLIENT_ID="xxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
CLIENT_SECRET="xxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
DISK_SIZE="50"
SSH_PUB_KEY="~/.ssh/id_rsa.pub"
```
<hr>

## **Service Principal**
If you don't have a service principal handy you can create one as long as you have the permissions to do so.
```
az ad sp create-for-rbac --name admin --role="Contributor" --scopes="/subscriptions/${SUBSCRIPTION_ID}"
```
Take note of the following; **appId**, **password** and **tenant**
<hr>

## **Create Resource Group**
Create the resource group with the command below:
```
az group create --location eastus --name "${RESOURCE_GROUP_NAME}" --subscription "${SUBSCRIPTION_ID}"
```
<hr>

## **Azure Container Registry**
Create the registry
```
az acr create --name "${ACR_REGISTRY_NAME}" \
--resource-group "${RESOURCE_GROUP_NAME}" \
--sku Premium \
--admin-enabled true \
--default-action Allow \
--location "${LOCATION}" \
--subscription "${SUBSCRIPTION_ID}" 
```
<hr>


## **Azure Kubernetes Service**
Create Kubernetes Cluster
```
az aks create --name ${AKS_CLUSTER_NAME} \
--ssh-key-value "${SSH_PUB_KEY}" \
--resource-group "${RESOURCE_GROUP_NAME}" \
--location "${LOCATION}" \
--attach-acr "${ACR_REGISTRY_NAME}" \
--kubernetes-version "${KUBERNETES_VERSION}" \
--node-count 3 \
--load-balancer-sku standard \
--disable-rbac --enable-cluster-autoscaler \
--min-count 3 --max-count 40 \
--node-osdisk-size "${DISK_SIZE}" \
--node-vm-size "${NODE_POOL_SKU}" \
--nodepool-name "${NODE_POOL_NAME}" \
--subscription "${SUBSCRIPTION_ID}" \
--vm-set-type VirtualMachineScaleSets \
--service-principal "${CLIENT_ID}" \
--client-secret "${CLIENT_SECRET}"
```
<hr>


## **Cluster Tear Down**
Well, cloud resource(s) costs money. Remember to tear the environment down when you are through.
```
az group delete --name ${RESOURCE_GROUP_NAME} \
--no-wait --subscription ${SUBSCRIPTION_ID} --yes
```
<hr>



## **Ansible Implementation**
You can also setup your cluster locally. Click [ansible kubernetes](https://github.com/uonyekwuluje/ansible-kubernetes)
