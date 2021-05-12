---
layout: post
title:  "Kubernetes Multi Cluster Management with kubectl"
categories: Kubernetes
---

Installing a kubernetes cluster involves a lot of steps and can be very complex. When you have a running cluster, it's easy to manage
that cluster from the master node or a node with `kubectl` configured. This becomes cumbersome and difficult to manage as the cluster
increases in size and also when there is a need to integrate kubernetes with CI/CD tools. This post is geared at simplifying the `kubectl` 
configuration for managing multiple clusters

### **Setup Requirements**
This post assumes the following:
* At least two working Kubernetes Clusters
* An external node with kubectl installed
* Network Access.

### **Config Components**
Create the `.kube` forlder on the node with kubectl installed
```
mkdir /home/$USER/.kube
```
Copy the kubernetes admin config to the admin node.
```
export SERVER1="server-ip-address" #192.168.1.2
export SERVER2="server-ip-address" #192.168.1.3
export KUSER="admin-user"

# download the Kubernetes admin config for work dev
scp $KUSER@$SERVER1:/etc/rancher/rke2/rke2.yaml ~/.kube/dev-kube-context`
scp $KUSER@$SERVER2:/etc/rancher/rke2/rke2.yaml ~/.kube/prod-kube-context`
```
*NOTE: The config location may be different based on the version of kubernetes installed*

### **Context Configuration**
After copying the config files over, the following sections need to be updated:<br>
Cluster Section.<br>
<code>
- cluster:
    server: https://192.168.1.x:6443
  name: dev-cluster
</code>

Users Section.<br>
<code>
users:
- name: dev-admin
</code>

Contexts Section.<br>
<code>
contexts:
- context:
    cluster: dev-cluster
    user: dev-admin
  name: dev-admin-context
</code>

* Export `KUBECONFIG`
```
export KUBECONFIG="$HOME/.kube/dev-kube-context:$HOME/.kube/prod-kube-context"
```

* **Get Contexts:** Run the command `kubectl config get-contexts` to get the contests
```
CURRENT   NAME                   CLUSTER        AUTHINFO     NAMESPACE
*         dev-cluster-context    dev-cluster    dev-admin    
          prod-cluster-context   prod-cluster   prod-admin   
```
List Nodes in default `dev-cluster-context`
```
kubectl get nodes

NAME              STATUS   ROLES                       AGE     VERSION
dev-kube-master   Ready    control-plane,etcd,master   4d16h   v1.20.6+rke2r1
dev-kube-node1    Ready    <none>                      4d16h   v1.20.6+rke2r1
dev-kube-node2    Ready    <none>                      4d16h   v1.20.6+rke2r1
```

List Nodes in `prod-cluster-context`
```
kubectl config use-context prod-cluster-context
# Switched to context "prod-cluster-context".

kubectl config get-contexts
# CURRENT   NAME                   CLUSTER        AUTHINFO     NAMESPACE
#          dev-cluster-context    dev-cluster    dev-admin    
#*         prod-cluster-context   prod-cluster   prod-admin   

kubectl get nodes
#NAME               STATUS     ROLES                       AGE   VERSION
#prod-kube-master   Ready      control-plane,etcd,master   50m   v1.20.6+rke2r1
#prod-kube-node1    Ready      <none>                      47m   v1.20.6+rke2r1
```
