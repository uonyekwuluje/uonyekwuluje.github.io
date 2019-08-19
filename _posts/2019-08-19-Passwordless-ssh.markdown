---
layout: post
title:  "Passwordless SSH Setup"
categories: systems_administration
---

SSH is a client/server protocol used for remote access of linux systems. It does so over a secure encrypted tunnel.
In its default state you have to provide a valid username and password in the terminal. This becomes a big problem
when you think of this at scale. 
To solve this problem we can configure SSH to work with passwordless login. All we have to do is update the target
system's authorized_keys file with the public key of the source system

#### **Assumptions**

* You are working off a Centos server. That said, this should work for other distributions. 
* You have a valid user on this servers. We will use **sysadmin** in our example. 
* SSH is installed and the daemon is up and running

#### **Setup**

|Hostname &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;|
|:------------ : |
|server1         |
|server2         |


#### **Server1 Config**

Generate keypair on server1
```
ssh-keygen -t rsa -b 4096 -C "sysadmin"

Generating public/private rsa key pair.
Enter file in which to save the key (/home/sysadmin/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/sysadmin/.ssh/id_rsa.
Your public key has been saved in /home/sysadmin/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:iygLGkBGxrrJl57gOTUyUJw7vFUYySQ/ARdtXfPAZN8 sysadmin
The key's randomart image is:
+---[RSA 4096]----+
|.+o=** . +*      |
|o.+++.+ ...= .   |
|.= .oo      o E  |
|= + ..           |
|+o +.   S        |
|+=.= . . .       |
|+ X + . .        |
|.* =             |
|. o              |
+----[SHA256]-----+
```
**note** : enter a secure passphrase

#### **Setup**

|Component &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; | Size &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; | Systems Specification |
|:------------- |:----:| --------------------: |
|Kibana         |   1  |  2 CPU   4GB RAM      |
|Elasticsearch  |   3  |  2 CPU   4GB RAM      |
|Filebeat       |   2  |  1 CPU   2GB RAM      |

**NOTE:**
You can make changes as needed. The above is just a base systems spec.

#### **Ansible Implementation**

Ansible Implementation of Elastic Stack [ansible-elastic-stack](https://github.com/uonyekwuluje/ansible-elastic-cluster)
