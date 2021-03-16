---
layout: post
title:  "Docker Tips, Tricks and Howto"
categories: Docker
---
Collection of Docker Commands for development and administrative tasks.  

### **Stop Containers**
```
# stop a container
docker stop <container id>

# stop all containers
docker stop $(docker ps -aq)
```

### **Remove all Containers**
```
# remove all containers
docker rm $(docker ps -aq)
```

### **Remove Images**
```
# remove docker image
docker rmi <image id>

# remove all images
docker rmi --force $(docker images -q)
```
