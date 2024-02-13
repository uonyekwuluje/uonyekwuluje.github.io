---
layout: post
title:  "Docker Compose Cheet Sheet"
categories: Systems Administration
author: Uchechukwu Onyekwuluje
---

Docker Compose is a tool for defining and running multi-container applications. I run into situations where I have to balance the 
use of standard compose file and flexibility of CLI options. This is my notes or cheet sheets on some use case I run into.

### Run and Pipe Commands 
Run nginx image, get version and pipe that version to a file.
```
sudo mkdir -p /opt/dkmount/
sudo chmod 777 /opt/dkmount/

docker run --rm --name nginxt -v /opt/dkmount:/tmp/ nginx:latest sh -c "nginx -v > /tmp/ng.out 2>&1"

ls -l /opt/dkmount/
total 4

cat /opt/dkmount/ng.out 
nginx version: nginx/1.25.3
```
