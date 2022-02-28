---
layout: post
title:  "Find Host IP Addreess"
categories: Systems Administration
---

When performing various automation tasks, there is usually a need to work with IP Addresses. 
To find the private IP address on a linux server, the following commands work 
```
hostname -I | awk '{print $1}'
ip route get 1.2.3.4 | awk '{print $7}'
ip addr show eno1 | grep "inet\b" | awk '{print $2}' | cut -d/ -f1
```
