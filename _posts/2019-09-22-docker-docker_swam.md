---
layout: post
title:  "Docker, Docker Swam Setup"
categories: cicd
---

Docker is simply a tool for managing containers. Containers allow developers, sysadmins, devops engineers etc to package their application along with libraries and other dependencies.

This short tutorial walks you through the installtion process along with configuring docker in swam mode. From a development stand point you can get a lot done before commiting your code. Swam affords you tha ability to test self-healing, load balancing, container scale operations, service discovery etc. A lot of these basics are necessary for devops

#### **Setup**
For our setup we will be using 3 nodes. One acting as master and the other two as swam task nodes. Our operating system of choice is linux and our distribution will be CentOS 7

|IP Address &nbsp; &nbsp; &nbsp; | Hostname |
|------------- | ----------------- |
|192.168.1.173 |  cicdmaster       |
|192.168.1.187 |  cicdnode1        |
|192.168.1.188 |  cicdnode2        |

**NOTE:**
*Ensure you have the appropriate ports and selinux configured. In AWS, Azure or Google cloud, remember to update security groups as needed.*


**Install Docker in all nodes**
```
sudo yum update -y
sudo yum install -y wget curl unzip
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo yum install -y docker-ce docker-ce-cli containerd.io

# Enable Services
systemctl enable docker
systemctl start docker
```

**Check docker service**
```
docker --version
```
You should see something like this
```
Docker version 19.03.2, build 6a30dfc
```


**Initialize swam cluster**
```
docker swarm init --advertise-addr 192.168.1.173
```
You should see something like this
```
Swarm initialized: current node (vzynd0x0ocomta3mxajsf205g) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-5zkyqoxpk6qizmob8ngjsbxy4auvyup4lup0ne8m1tevld0o3e-2p17010s0rcskhpqw86p61icx 192.168.1.173:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```

**Check nodes and join nodes**
```
docker node ls
```
You should see something like this
```
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
vzynd0x0ocomta3mxajsf205g *   cicdmaster          Ready               Active              Leader              19.03.2
```

On the other two nodes, run the command:
```
swarm join --token SWMTKN-1-5zkyqoxpk6qizmob8ngjsbxy4auvyup4lup0ne8m1tevld0o3e-2p17010s0rcskhpqw86p61icx 192.168.1.173:2377
```
You should see this:
```
This node joined a swarm as a worker.
```

Log back into the master and verify:
```
docker node ls
```
You should see something like this:
```
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
vzynd0x0ocomta3mxajsf205g *   cicdmaster          Ready               Active              Leader              19.03.2
6yo9csctoqvc6wwedilkaeyvk     cicdnode1           Ready               Active                                  19.03.2
bv7pru83z0vp3akdp2p6hm5u1     cicdnode2           Ready               Active                                  19.03.2
```

**Retriving  Token**
To retrieve the swam token run this command in the master node:
```
docker swarm join-token manager -q
``` 
You should see something like this
```
SWMTKN-1-5zkyqoxpk6qizmob8ngjsbxy4auvyup4lup0ne8m1tevld0o3e-7diouknaivmrss6dydhibvrv4
```
