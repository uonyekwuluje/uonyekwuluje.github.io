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
#### **Command Cheat Sheet**

|         Command                  |      Description             |              Reference Links               |
|:-------------------------------: |:----------------------------:| :----------------------------------------: | 
|  `kubectl config view`           | View current config          |   [kubernetes](https://kubernetes.io/)     |
|  `kubectl config get-contexts`   | Get all installed contexts   |   [kubernetes](https://kubernetes.io/)     | 
|  `kubectl delete ns <namesapce>` | Delete kubernetes namespace  |   [kubernetes](https://kubernetes.io/)     | 

**Utilities** <br>
Deleting resources in bulk using regEx
```
# Delete all pods matching a given pattern
kubectl get pods -n mynamespace --no-headers=true | awk '/pattern1|pattern2/{print $1}'| xargs  kubectl delete -n mynamespace pod
```

**Port Forwarding** <br>
Port-Forward allows you to access and interact with internal Kubernetes cluster processes from your localhost. You can use this 
method to investigate issues and make changes locally without the need to expose the services.
```
# Port Forwarding Command
kubectl port-forward TYPE/NAME [options] LOCAL_PORT:REMOTE_PORT

# List Services
kubectl get svc -n argocd
NAME                    TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
argocd-dex-server       ClusterIP   10.108.149.200   <none>        5556/TCP,5557/TCP,5558/TCP   12h
argocd-metrics          ClusterIP   10.103.23.208    <none>        8082/TCP                     12h
argocd-redis            ClusterIP   10.111.143.18    <none>        6379/TCP                     12h
argocd-repo-server      ClusterIP   10.98.189.78     <none>        8081/TCP,8084/TCP            12h
argocd-server           ClusterIP   10.97.109.37     <none>        80/TCP,443/TCP               12h
argocd-server-metrics   ClusterIP   10.108.87.103    <none>        8083/TCP                     12h
```
If we want to expose port 443 for argocd-server, we can run the following
```
kubectl port-forward svc/argocd-server -n argocd 8080:443
```
If you want to access port 8080 outside localhost or any local address, we can run the following
```
kubectl port-forward --address 0.0.0.0 svc/argocd-server -n argocd 8080:443
```
If you want to access port 8080 on localhost and specific addresses, we can run the following
```
kubectl port-forward --address localhost,192.168.1.34 svc/argocd-server -n argocd 8080:443
```
You can also adjust for deployments or other service as needed.


**Kubernetes Context** <br>
Kubernetes assumes the default after a clean setup. As you add more namespaces, you have to pass ```-n <namespace>``` as part of the command.
You can change the context to avoid repeated use of that option
```
kubectl get ns
NAME              STATUS   AGE
poc1              Active   19h
default           Active   20h
poc2              Active   4h
kube-node-lease   Active   20h
kube-public       Active   20h
kube-system       Active   20h

#change context
kubectl config set-context --current --namespace=poc1
```


**Clean Stale Kubernetes Namespace** <br>
Delete stale namespace and all resource in it
```
export NAMESPACE="test"
kubectl get namespace $NAMESPACE -o json > $NAMESPACE.json
# edit $NAMESPACE.json and change finilizer to an empty list
kubectl replace --raw "/api/v1/namespaces/$NAMESPACE/finalize" -f ./$NAMESPACE.json
kubectl delete pods -n $NAMESPACE --force
kubectl get namespace
```




#### **Reference Links**
* Kubernetes Cheat Sheet [Kubernetes](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
* Rancher Kubernetes Cheat Sheet [Rancher](https://rancher.com/learning-paths/how-to-manage-kubernetes-with-kubectl/)
