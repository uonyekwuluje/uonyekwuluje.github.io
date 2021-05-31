---
layout: post
title:  "Traefik v2 Ingress Installation"
categories: Kubernetes
---
The Traefik Kubernetes Ingress provider is a Kubernetes Ingress controller; that is to say, it manages access to 
cluster services by supporting the Ingress specification.
In this post we will be installing Traefik V2 and MetalLB. We will test name based routing in our cluster 

### **Systems Requirements**
* Existing Kubernetes Cluster
* Helm 3
* RBAC Access on existing cluster


### **Install MetalLB**
Since this is a local kubernetes install, we will need a load balancer for this. We will be using MetalLB. This is
a basic install. For more details and customizations, see [MetalLB](https://metallb.universe.tf/concepts/).
```
kubectl apply -f https://raw.githubusercontent.com/google/metallb/v0.9.6/manifests/namespace.yaml
kubectl apply -f https://raw.githubusercontent.com/google/metallb/v0.9.6/manifests/metallb.yaml
kubectl create secret generic -n metallb-system memberlist --from-literal=secretkey="$(openssl rand -base64 128)"
```

base config
```
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: default
      protocol: layer2
      addresses:
      - 192.168.1.30-192.168.1.40
```

apply config
```
kubectl apply -f config.yml
```

Check status `kubectl get all -n metallb-system`



### **Install Traefik v2**
Add traefik repository
```
helm repo add traefik https://helm.traefik.io/traefik
helm repo update
```

Install Traefik
```
kubectl create ns traefik-v2
helm install --namespace=traefik-v2 traefik traefik/traefik
```

Check status `kubectl get all -n traefik-v2`. Ensure  the resources are in a healthy state.

```
NAME                           READY   STATUS    RESTARTS   AGE
pod/traefik-5d7c8ddd5d-cptdm   1/1     Running   0          76m

NAME              TYPE           CLUSTER-IP     EXTERNAL-IP    PORT(S)                      AGE
service/traefik   LoadBalancer   10.43.10.130   192.168.1.30   80:31902/TCP,443:32766/TCP   76m

NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/traefik   1/1     1            1           76m

NAME                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/traefik-5d7c8ddd5d   1         1         1       76m
```
*NOTE: Take note of the `EXTERNAL-IP`*<br>


### **DNS Update**
Update your DNS with the Ingress IP. In this case `192.168.1.30`. We will be using the `/etc/hosts` for this
```
192.168.1.30   nginx.infracid.com
192.168.1.30   jenkins.infracid.com
```
*NOTE: We are defining this upfront because we will be testing out nginx and jenkins. Adjust based on your needs*<br>


### **Applications and Deployments**
Nginx Deployment. `nginx-deployment.yaml`
```
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 1
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2 
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
  - name: nginx
    port: 8080
    targetPort: 80

---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: nginx-web-ui
  annotations:
    kubernetes.io/ingress.class: traefik
spec:
  rules:
  - host: nginx.infracid.com
    http:
      paths:
      - path: /
        backend:
          serviceName: nginx-service
          servicePort: nginx
```
Apply config. `kubectl apply -f nginx-deployment.yaml`
