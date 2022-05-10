---
layout: post
title:  "Helm Repository"
categories: Helm
---

Helm is a package manager for kubernetes. You can manage your applications using Helm. Helm accomplishes this by
using charts, templates, values.  Upon initialization, you have predefined configs to use. While it's possible using
package and artifact managers like Artifactory, Nexus, GitHub, GitLab etc it's also nice to have a better feel for
the building blocks on a bare server.

## Requirements
For this server, we will be using Linux
* 50GB Hard Drive
* 1 CPU
* 3 GB of RAM
* CentOS 8 Linux
* Network: *Configure as needed in your environment*

## Packages
Install the following packages on the server
```
yum install -y sysstat unzip gcc kernel-devel kernel-headers dnf-plugins-core python3-pip \
device-mapper-persistent-data lvm2 libxml2 libxml2-devel libxslt libxslt-devel wget libffi-devel \
openssl-devel make openssl-devel bzip2-devel vim nc yum-utils runc net-tools epel-release httpd-tools

yum groupinstall -y "Development Tools"
yum install -y nginx
```
**NOTE:** ***Make package changes as needed. Your must haves are nginx, httpd-tools and epel-release***

## Storage
We will be using the primary partion. I'll create a folder
```
sudo mkdir -p /data/helm/charts
```
**NOTE:** ***Configure folder permissions as needed. I am making it world accessible***

## Setup Basic Auth
Setup basic auth for our test helm chart
```
mkdir -p /data/helm/charts/appnginx
chmod 7777 /data/helm/charts/appnginx
htpasswd -c /data/helm/charts/appnginx/.htpasswd admin     [admin/password]
```

## Update Nginx
Create a new location and update nginx in `/etc/nginx/nginx.conf`
```
 location /appnginx {
    root /data/helm/charts;
    auth_basic "Helm Login";
    auth_basic_user_file /data/helm/charts/appnginx/.htpasswd;
    dav_methods  PUT DELETE MKCOL COPY MOVE;
    create_full_put_path   on;
    dav_access             group:rw  all:r;
    client_max_body_size 10000m;
    autoindex on;
    allow all;
 }
```
Stop and restart nginx. Test to ensure you can login with the credentials
```
sudo systemctl stop nginx
sudo systemctl start nginx

# Test
http://<server_name>/appnginx/
```

## Create Custom Nginx Package
Create chart directory
```
helm create appnginx
```
change directory `cd appnginx` and update as needed. In our case, we want something basic
```
rm -Rf *
mkdir template

bash -c 'cat <<EOF> Chart.yaml
apiVersion: v2
name: appnginx
description: Nginx 1.16.1 Helm Chart
type: application
version: 0.1.0
appVersion: "1.1.1"
EOF'

bash -c 'cat <<EOF> values.yaml 
namespace: nginx

replicaCount: 4

image:
  repository: "nginx"
  pullPolicy: "IfNotPresent"
  tag: "1.16.1"

imagePullSecrets: []
nameOverride: "nginx-app"
fullnameOverride: "nginx-chart"
EOF'


bash -c 'cat <<EOF> templates/namespace.yaml 
---
apiVersion: v1
kind: Namespace
metadata:
  name: {{ .Values.namespace }}
EOF'

bash -c 'cat <<EOF> templates/deployment.yaml 
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Values.namespace }}-deployment
  namespace: {{ .Values.namespace }}
spec:
  selector:
    matchLabels:
      app: {{ .Values.namespace }}
  replicas: {{ .Values.replicaCount }}
  template:
    metadata:
      labels:
        app: {{ .Values.namespace }}
    spec:
      containers:
      - name: {{ .Values.namespace }}
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
        imagePullPolicy: {{ .Values.image.pullPolicy }}
        ports:
        - containerPort: 80

---
apiVersion: v1
kind: Service
metadata:
  name: {{ .Values.namespace }}-service
  namespace: {{ .Values.namespace }}
  labels: 
    app: {{ .Values.namespace }}
spec:
  ports:
  - port: 80
    protocol: TCP
  selector:
    app: {{ .Values.namespace }} 
EOF'
```

Create Package
```
cd ../
helm package appnginx/
```

Initialize
```
mv appnginx-0.0.1.tgz appnginx/
helm repo index appnginx --url http://192.168.1.32/appnginx
```

Publish Chart
```
curl -u admin:password -T appnginx/index.yaml http://192.168.1.32/appnginx/
curl -u admin:password -T appnginx/appnginx-0.0.1.tgz http://192.168.1.32/appnginx/
```

Add Helm Repository
```
helm repo add appnginx http://192.168.1.32/appnginx/ --username admin --password password
```

Install Package
```
helm install appnginx appnginx/appnginx
```

List `helm list`
```
NAME        NAMESPACE   REVISION    UPDATED                                 STATUS      CHART           APP VERSION
appnginx    default     2           2022-05-10 05:16:24.19386226 -0400 EDT  deployed    appnginx-0.0.1  1.1.0      
```

Delete Chart `helm delete appnginx`




## Reference Links
* [Helm](https://helm.sh/)
* [Helm GitHub](https://github.com/helm/helm)
