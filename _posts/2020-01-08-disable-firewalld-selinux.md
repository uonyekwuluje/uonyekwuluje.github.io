---
layout: post
title:  "Disabling Firewalld and Selinux in Centos 8"
categories: Security
---

When configuring your Centos/RHEL server for any purpose, securing the server has to be a top priority. That said, sometimes it makes
sense turning them off to test packages, services, configs, access etc.
This post is geared at accomplishing that task

**Check Selinux Status**
```
sudo sestatus
```
You should have something like this:
```
SELinux status:                 enabled
SELinuxfs mount:                /sys/fs/selinux
SELinux root directory:         /etc/selinux
Loaded policy name:             targeted
Current mode:                   enforcing
Mode from config file:          enforcing
Policy MLS status:              enabled
Policy deny_unknown status:     allowed
Memory protection checking:     actual (secure)
Max kernel policy version:      31
```
<hr>

**Disable Selinux Temporarily**
```
sudo setenforce 0
```
You can also use the command below:
```
sudo setenforce Permissive
```
<hr>

**Disable Selinux Permanently and Reboot**
```
sudo sed -i 's/^SELINUX=.*/SELINUX=disabled/g' /etc/selinux/config
sudo reboot
```
<hr>

**Verify**
When the server is up login and verify
```
sudo sestatus
```
you should see something like this
```
SELinux status:                 disabled
```
