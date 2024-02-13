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
**Build image from Dockerfile**<br>
```
docker build -t <image_name>
```
**Build an Image from a Dockerfile without the cache**
```
docker build -t <image_name> . –no-cache
```
**List local images**
```
docker images
```
**Delete an Image, all images respectively**
```
docker rmi <image_name>
docker rmi $(docker images -q)
```
**Remove all unused images**
```
docker image prune
```
