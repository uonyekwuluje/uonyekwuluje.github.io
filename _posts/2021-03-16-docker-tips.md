---
layout: post
title:  "Docker commands and Howto"
categories: Docker
---
Collection of Docker Commands for development and administrative tasks.  

## Stop Containers
```
# stop a container
docker stop <container id>

# stop all containers
docker stop $(docker ps -aq)
```

## Remove all Containers
```
# remove all containers
docker rm $(docker ps -aq)
```

## Remove Images
```
# remove docker image
docker rmi <image id>

# remove all images
docker rmi --force $(docker images -q)
```

## Clean all
```
# Prune All Cached Images
docker system prune -a

# Clean Images and Containers
docker rm $(docker ps -q -a) -f
docker rmi $(docker images -q) -f
``` 
