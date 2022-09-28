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

##  Connecting using the PersistentVolume resource:
To create the PersistentVolume object for the NFS volume, use the following manifest. 
```
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-volume
  labels:
    storage.k8s.io/name: nfs
    storage.k8s.io/part-of: kubernetes-complete-reference
spec:
  accessModes:
  - ReadWriteOnce
  - ReadOnlyMany
  - ReadWriteMany
  capacity:
    storage: 4Gi
  storageClassName: ""
  persistentVolumeReclaimPolicy: Recycle
  volumeMode: Filesystem
  nfs:
    server: 192.168.1.50 
    path: /data
    readOnly: no
```

## Dynamic provisioning using StorageClass:
To provision PersistentVolume dynamically using the StorageClass, you need to install the NFS provisioner. 
The nfs-subdir-external-provisioner achieves that. Install with command below.
```
helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner

helm install nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner \
  --create-namespace \
  --namespace nfs-provisioner \
  --set nfs.server=192.168.1.50 \
  --set nfs.path=/data
```
To create the PersistentVolumeClaim use the following manifest:
```
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nfs-test
  labels:
    storage.k8s.io/name: nfs
    storage.k8s.io/part-of: kubernetes-complete-reference
spec:
  accessModes:
  - ReadWriteMany
  storageClassName: nfs-client
  resources:
    requests:
      storage: 5Gi
```

## Reference Links
* https://itnext.io/kubernetes-storage-part-1-nfs-complete-tutorial-75e6ac2a1f77
* https://fabianlee.org/2022/01/12/kubernetes-nfs-mount-using-dynamic-volume-and-storage-class/
* https://medium.com/@myte/kubernetes-nfs-and-dynamic-nfs-provisioning-97e2afb8b4a9
* https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner
