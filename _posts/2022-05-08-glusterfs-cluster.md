---
layout: post
title:  "GlusterFS"
categories: Storage
---

GlusterFS (Gluster File System) is an open source Distributed File System. Can be used for Cloud Computing, Streaming,
Backup and all sorts of storage related tasks. In summary, it's a scalable network filesystem.

## Update Hosts file
**Update Hosts File:** Update `/etc/hosts` on all nodes
```
sudo bash -c 'cat <<EOF>> /etc/hosts
192.168.1.146 gluster01
192.168.1.24  gluster02
192.168.1.30  gluster03
EOF'
```

**Package Update:** Upgrade and install required packages on all nodes
```
sudo yum update && sudo yum upgrade -y
sudo yum install -y wget centos-release-gluster epel-release 
sudo yum install -y gcc make kernel-devel kernel-headers wget net-tools yum-versionlock glusterfs-server
```

**Enable and start service**
```
sudo systemctl enable glusterd
sudo systemctl start glusterd
```

**Prepare and mount volumes**
Create mount point and file system for new drive. This has to be done for all servers
```
sudo mkdir -p /gfsvolume/
blkid

vim /etc/fstab 
>>
UUID=fb97e89e-b9ab-43f8-8ab1-0b691a96b3af   /gfsvolume/  ext4 defaults 0 0

mount -a
```

**Trust Gluster Pool**
```
--> [From gluster01]
sudo gluster peer probe gluster02
sudo gluster peer probe gluster03
```

Check Status with `sudo gluster peer status`
```
Number of Peers: 2

Hostname: gluster02
Uuid: a4b5e364-929b-4ddf-b3f5-43223bf73651
State: Peer in Cluster (Connected)

Hostname: gluster03
Uuid: a28b7d2a-b307-4392-abab-be26d0fc6875
State: Peer in Cluster (Connected)
```

Check pool list with `sudo gluster pool list`
```
UUID                    Hostname    State
a4b5e364-929b-4ddf-b3f5-43223bf73651    gluster02   Connected 
a28b7d2a-b307-4392-abab-be26d0fc6875    gluster03   Connected 
d5038bbf-8a1d-49ea-8934-90243928e870    localhost   Connected 
```

## Reference
* [Gluster](https://www.gluster.org/)
