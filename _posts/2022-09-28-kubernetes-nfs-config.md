---
layout: post
title:  "NFS Persisent Volume Config and Setup"
categories: Kubernetes
---

This tutorials is useful for Kubernetes storage architecture and concepts. Specifically, this focuses on NFS. Running kubernetes On-Prem has its challenges.
One of the challenges centers on using Persistent Volumes. While options exists like Gluster, Ceph etc. NFS works pretty well. It's easy to setup and accomplishes
a lot with minimal effort relative to the others. See my previous post on [NFS Setup](http://blog.ucheonyekwuluje.com/nfs/2021/11/22/nfs-setup-config.html) before continuing with this tutorial.


## Connecting to NFS directly with Pod manifest 
To connect to the NFS volume directly with Pod manifest, use the NFSVolumeSource in the Pod. Here is an example:
```
---
apiVersion: v1
kind: Pod
metadata:
  name: test-nfs-pod
  labels:
    app.kubernetes.io/name: alpine
    app.kubernetes.io/part-of: kubernetes-complete-reference
spec:
  containers:
  - name: alpine
    image: alpine:latest
    command:
      - touch
      - /data/test
    volumeMounts:
    - name: nfs-volume
      mountPath: /data
  volumes:
  - name: nfs-volume
    nfs:
      server: 192.168.1.50
      path: /data
      readOnly: no
```

