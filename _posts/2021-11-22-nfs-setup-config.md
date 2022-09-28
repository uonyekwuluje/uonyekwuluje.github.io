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




#### **NFS Options**
NFS mount options

| NFS Options      | Description |
| ----------- | ----------- |
| rw          | Allow both read and write requests on a NFS volume.       |
| ro          | Allow only read requests on a NFS volume.        |
| sync        | Reply to requests only after the changes have been committed to stable storage. (Default) |
| async       |	Allows NFS server to reply to requests before any changes made by that request have been committed | 
| secure      |	This option requires that requests originate on an Internet port less than IPPORT_RESERVED (1024). (Default) |
| insecure    |	This option accepts all ports. |
| wdelay      |	Delay committing write request to disc slightly if it suspects that another related write request may be in progress |
| no_wdelay     |  |
| subtree_check    | This option enables subtree checking. (Default) |
| no_subtree_check | This option disables subtree checking |
| root_squash      | Map requests from uid/gid 0 to the anonymous uid/gid. |
| no_root_squash   | Turn off root squashing. |
| all_squash 	   | Map all uids and gids to the anonymous user. Useful for NFS exported public FTP directories |
| no_all_squash    | Turn off all squashing. (Default) |
