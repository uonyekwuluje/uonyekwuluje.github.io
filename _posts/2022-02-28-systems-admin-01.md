---
layout: post
title:  "Find Host IP Addreess"
categories: Systems-Administration-vol1
---
Find the private IP address on a linux server
```
hostname -I | awk '{print $1}'
ip route get 1.2.3.4 | awk '{print $7}'
```
