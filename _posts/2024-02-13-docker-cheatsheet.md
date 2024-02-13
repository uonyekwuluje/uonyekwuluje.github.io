---
layout: post
title:  "Docker CLI Cheet Sheet"
categories: Systems Administration
author: Uchechukwu Onyekwuluje
---

I use and leverage docker a lot for Development, POC, CICD, Builds etc. It's my go to tool for anything systems related that
needs to be done quickly and straight to the point. This is a collection of commands I use a lot and reference to some sites.
My cheet sheet of keeping my references in one place 

### Images 
This is a collection of commands for managing images.<br>
Build image from Dockerfile<br>
```
docker build -t <image_name>
```
Build an Image from a Dockerfile without the cache
```
docker build -t <image_name> . –no-cache
```
List local images
```
docker images
```
Delete an Image, all images respectively
```
docker rmi <image_name>
docker rmi $(docker images -q)
```
Remove all unused images
```
docker image prune
```

### Containers 
This is a collection of commands for managing containers.<br>
Create and run a container from an image, with a custom name
```
docker run --name <container_name> <image_name>
```
Run a container with and publish a container’s port(s) to the host
```
docker run -p <host_port>:<container_port> <image_name>
```
Run a container in the background. As a daemon
```
docker run -d <image_name>
```
Start or stop an existing container
```
docker start|stop <container_name> (or <container-id>)
```
Remove a stopped container
```
docker rm <container_name>
```
Open a shell inside a running container
```
docker exec -it <container_name> sh
docker exec -it <container_name> bash
```
Fetch and follow the logs of a container
```
docker logs -f <container_name>
```
Inspect a running container
```
docker inspect <container_name> (or <container_id>)
```
list currently running containers, all containers
```
docker ps
docker ps --all
```
View resource usage stats
```
docker container stats
```

### Cleanup Commands
Curation of cleanuo commands below:
```
docker rm -f $(docker ps -a -q)
docker network rm $(docker network list -q)
docker volume rm $(docker volume list -q)
docker rmi $(docker images -q)
docker system prune -a --volumes
```

### Docker Compose Commands
The commands work when there is a default `docker-compose.yaml` in the current path
```
docker-compose up -d      
docker-compose up -d --force-recreate
docker-compose up -d --force-recreate --remove-orphans
docker-compose down
docker-compose down --remove-orphans
```
