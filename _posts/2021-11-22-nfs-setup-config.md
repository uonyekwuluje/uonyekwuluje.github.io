---
layout: post
title:  "NFS Config and Setup"
categories: NFS 
---

NFS **Network File System** is a distributed file system protocol that allows you to mount remote directories on your desktop, machine. 
In this post, I’ll go over how to install the software needed for NFS functionality on Debian. While this focuses on Debian, the process
applies to Ubuntu, Centos and others.

#### **Server Installation & Config**
On the NFS server, install this.
```
sudo apt -y install nfs-kernel-server nfs-common
```
Create the directory to share on the NFS server.
```
sudo mkdir /opt/data
```
Update the exports file `/etc/exports`
```
/opt/data  *(rw,sync,no_subtree_check)
```
*NOTE:* In this case, I am allowing all because it's my private network. You have to be more restrictive on other networks<br>
Mount when done
```
sudo exportfs -a
```



#### **Client Installation & Config**
To configure a client, install the client tools.
```
sudo apt-get install nfs-common
```
Create a directory to mount the remote share. `NFS Server IP => 192.168.1.10`. Client mount point `/opt/data_mount`
I like mounting in `/etc/fstab`.
```
192.168.1.10:/opt/data /opt/data_mount  nfs      defaults    0       0
```
Type this command. `sudo mount -a`. If you type this command `ls /opt/data_mount` you should see the remote content
