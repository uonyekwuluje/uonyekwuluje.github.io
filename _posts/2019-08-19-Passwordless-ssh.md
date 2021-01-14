---
layout: post
title:  "Passwordless SSH Setup"
categories: Systems Administration
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
[sysadmin@server1 ~]$ ssh-keygen -t rsa -b 4096 -C "sysadmin"

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

#### **Disable host key checking**

To prevent continuous prompts for accepting known hosts, we can disable this on server1. In this case, the source system. Create a new file or update yours. in this case `/home/sysadmin/.ssh/config`
```
Host *
   StrictHostKeyChecking no
   UserKnownHostsFile=/dev/null
```

change file attributes when you are through.
```
chmod 0600 .ssh/config
```


#### **Server2 Config**

Create ssh folder, create authorized_keys and update attributes
```
[sysadmin@server2 ~]$ mkdir ~/.ssh
[sysadmin@server2 ~]$ chmod 700 ~/.ssh
[sysadmin@server2 ~]$ touch ~/.ssh/authorized_keys
[sysadmin@server2 ~]$ chmod 600 ~/.ssh/authorized_keys

```
Update authorized_keys with public keys from server 1 `/home/sysadmin/.ssh/id_rsa.pub`


#### **Test SSH Connection**

ssh into server2 from server1 and it should work without prompting you for a password om user `sysadmin`
```
[sysadmin@server1 ~]$ ssh server2
Warning: Permanently added 'server2,192.168.1.165' (ECDSA) to the list of known hosts.
Last login: Mon Aug 19 06:53:31 2019 from 192.168.1.164
[sysadmin@server2 ~]$ 
```
