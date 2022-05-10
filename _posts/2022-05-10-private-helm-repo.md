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

