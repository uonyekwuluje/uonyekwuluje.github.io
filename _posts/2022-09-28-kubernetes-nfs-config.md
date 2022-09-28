---
layout: post
title:  "NFS Persisent Volume Config and Setup"
categories: Kubernetes
---

This tutorials is useful for Kubernetes storage architecture and concepts. Specifically, this focuses on NFS. Running kubernetes On-Prem has its challenges.
One of the challenges centers on using Persistent Volumes. While options exists like Gluster, Ceph etc. NFS works pretty well. It's easy to setup and accomplishes
a lot with minimal effort relative to the others. See my previous post on [NFS Setup](http://blog.ucheonyekwuluje.com/nfs/2021/11/22/nfs-setup-config.html) before continuing with this tutorial.


#### **Server Installation & Config**
On the NFS server, install this.
```
sudo apt -y install nfs-kernel-server nfs-common
sudo systemctl enable nfs-server
sudo systemctl start nfs-server
```
Create the directory to share on the NFS server.
```
sudo mkdir /opt/data
```
Update the exports file `/etc/exports`
```
/opt/data  *(rw,sync,no_subtree_check,no_root_squash)
```
*NOTE:* In this case, I am allowing all because it's my private network. You have to be more restrictive on other networks<br>
Mount when done
```
sudo exportfs -ar
```

