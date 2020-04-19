---
layout: post
title:  "Kubernetes Command Tips & Cheat Sheet"
categories: Kubernetes
---

#### **Requirements**
Ensure kubectl is installed and configured. See links below

* Local installation of [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl)
* Local installation of [helm](https://helm.sh/docs/intro/install)

**kubectl test** <br>
```
kubectl version --short
```
You should see this:
```
Client Version: v1.16.0
Server Version: v1.14.8
```
<hr>
#### **Command Cheat Sheet**

|         Command                 |      Description             |              Reference Links               |
|:------------------------------: |:----------------------------:| :----------------------------------------: | 
|  `kubectl config view`          | View current config          |   [kubernetes](https://kubernetes.io/)     |
|  `kubectl config get-contexts`  | Get all installed contexts   |   [kubernetes](https://kubernetes.io/)     | 

<hr>


**Utilities** <br>
Deleting resources in bulk using regEx
```
# Delete all pods matching a given pattern
kubectl get pods -n mynamespace --no-headers=true | awk '/pattern1|pattern2/{print $1}'| xargs  kubectl delete -n mynamespace pod
```
<hr>



#### **Reference Links**
* Kubernetes Cheat Sheet [Kubernetes](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
* Rancher Kubernetes Cheat Sheet [Rancher](https://rancher.com/learning-paths/how-to-manage-kubernetes-with-kubectl/)
