---
layout: post
title:  "Kubernetes Command Tips & Cheat Sheet"
categories: Kubernetes
---

#### **Requirements**
Ensure kubectl is installed and configured. See links below

* Local installation of [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl)
* Local installation of [helm](https://helm.sh/docs/intro/install)
```
kubectl version --short
```
You should see this:
```
Client Version: v1.16.0
Server Version: v1.14.8

```
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
