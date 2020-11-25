---
layout: post
title:  "Cassandra 4.x Setup"
categories: Data, Database
---

Cassandra is an open-source, distributed, wide column store, NoSQL database management system. It is designed to handle large amounts of data across compute resource or servers. It provides high availability with no single point of failure. It  offers robust support for clusters spanning multiple datacenters with asynchronous masterless replication allowing low latency operations for all clients. <br>

This post is geared at a base installation of Apache Cassandra 4.x in one of two modes:
* Standalone Mode
* Cluster Mode

#### **Systems Requirement for clustered/multi node setup** 

|  Hosts       | Systems Specification |
|--------------|--------------------------------|
|cassandra01   |  2 CPU  4GB RAM  40GB Storage  |
|cassandra02   |  2 CPU  4GB RAM  40GB Storage  |
|cassandra03   |  2 CPU  4GB RAM  40GB Storage  |

#### **Systems Requirement for standalone node setup** 

|  Hosts    | Systems Specification |
|-----------|--------------------------------|
|cassandra  |  2 CPU  4GB RAM  40GB Storage  |

***NOTE:***  *You can make changes as needed. The above is just a base systems spec. for our POC*

#### **Package Installation**
We will be running our setup on CentOS based servers. Run this command on all servers
```
sudo yum update -y
sudo yum group install -y 'Development Tools'
sudo yum install -y libxml2 libxml2-devel libxslt libxslt-devel wget gcc libffi-devel \
openssl-devel make openssl-devel bzip2-devel java-11-openjdk-devel java-11-openjdk 
```
Configure Cassandra Repository
```
sudo bash -c 'cat <<EOF>> /etc/yum.repos.d/cassandra.repo
[cassandra]  
name=Apache Cassandra
baseurl=https://www.apache.org/dist/cassandra/redhat/40x/
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://www.apache.org/dist/cassandra/KEYS
EOF'
```
Install Cassandra
```
sudo yum update -y && sudo yum install -y cassandra
```
Enable and start cassandra services
```
sudo systemctl enable cassandra 
sudo systemctl start cassandra
```
Test out installation by running this command ```nodetool status```. if all goes well, you should see this:
```
Datacenter: datacenter1
=======================
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address    Load       Tokens  Owns (effective)  Host ID                               Rack 
UN  127.0.0.1  73.28 KiB  256     ?                 74aed20b-6a72-4b88-9d55-997aa8e00283  rack1
```
At this point you should be all set for the standalone setup


<br><br>
## **Multi Node Cassandra Setup**
The following steps outlines base configs and updates for a multi node clister setup
* Stop the service and delete the contents of the data folder
```
sudo service cassandra stop
sudo rm -rf /var/lib/cassandra/data/system/*
```
* Edit ```/etc/cassandra/default.conf/cassandra.yaml``` using your favorite editor. Update the following:
  - cluster_name: '<Your Cluster Name>'
  - num_tokens: '<desired num token>'  # default 256
  - - seeds: "<comma seperated list of cassandra servers>"
  - listen_address: "<host IP or Hostname>"
  - rpc_address: "<host IP or Hostname>"
  - endpoint_snitch: GossipingPropertyFileSnitch 
* Repeat previous steps on all servers changing the following as needed
  - listen_address: "<host IP or Hostname>"
  - rpc_address: "<host IP or Hostname>"
* Edit ```/etc/cassandra/default.conf/cassandra-rackdc.properties``` using your favorite editor. Update the following:
  - dc=<datacenter name>
  - rack=<rack name>
* Restart the service on all nodes
```
sudo systemctl restart cassandra
```
verify status to be sure everything is working correctly
```
sudo systemctl status cassandra
```
* Loginto any node and check the node status. Type this command ```nodetool status```. If all goes well, you should see this:
```
Datacenter: DEVLABS_DC
======================
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address        Load        Tokens  Owns (effective)  Host ID                               Rack 
UN  192.168.1.235  296.15 KiB  256     ?                 437cc9b6-a528-4454-9655-5e775895dfc2  RACK1
UN  192.168.1.236  296.12 KiB  256     ?                 577a9114-433a-42fb-9784-1926f4798c52  RACK1
UN  192.168.1.234  275.66 KiB  256     ?                 b7f42478-12bc-439c-b565-b4d070c3f344  RACK1
```
<br><br>











