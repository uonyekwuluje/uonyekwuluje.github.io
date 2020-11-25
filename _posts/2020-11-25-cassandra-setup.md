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


#### **Multi Node Cassandra Setup**
The following steps outlines base configs and updates for a multi node clister setup
























#### **Ansible Implementation**
For Elastic Stack automation see [ansible-elastic-stack](https://github.com/uonyekwuluje/ansible-elastic-stack) for ansible deployment






#### **Manual Installation**

**Server Inventory**

|Server Name  |  Ip Address |
|-------------|-------------|
|kibanaserver |  10.0.0.4   |
|elknode-01   |  10.0.1.4   |
|elknode-02   |  10.0.1.5   |
|elknode-03   |  10.0.1.6   |

**Install Kibana and Elasticsearch on kibana Node**
```
sudo yum update -y
sudo yum install -y wget git pyOpenSSL
sudo yum install -y java-11-openjdk-devel java-11-openjdk
sudo yum group install -y "Development Tools"
rpm -ivh https://artifacts.elastic.co/downloads/kibana/kibana-7.7.0-x86_64.rpm
rpm -ivh https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.7.0-x86_64.rpm

# Enable Services
systemctl enable kibana.service
systemctl enable elasticsearch.service

# Update Kibana Config
cat << EOF > /etc/kibana/kibana.yml
server.host: 0.0.0.0
server.name: kibana
elasticsearch.hosts: ["http://localhost:9200"]
EOF

# Update Elasticsearch Config
cat << EOF > /etc/elasticsearch/elasticsearch.yml
cluster.name: "testcluster"
node.name: "kibanaserver"
node.master: false
node.data: false
node.ingest: false
search.remote.connect: true
network.host: 0.0.0.0
path.data: /var/lib/elasticsearch
path.logs: /var/log/elasticsearch
discovery.seed_hosts: ["kibanaserver", "elknode-01", "elknode-02", "elknode-03"]
cluster.initial_master_nodes: ["kibanaserver", "elknode-01", "elknode-02", "elknode-03"]
EOF 

# Restart Services
systemctl restart kibana.service
systemctl restart elasticsearch.service

# Check Service Status
systemctl status kibana.service
systemctl status elasticsearch.service
```




**Install Elasticsearch on Elasticsearch Nodes**
```
sudo yum update -y
sudo yum install -y wget git pyOpenSSL
sudo yum install -y java-11-openjdk-devel java-11-openjdk
sudo yum group install -y "Development Tools"
rpm -ivh https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.7.0-x86_64.rpm

# Enable Services
systemctl enable elasticsearch.service

# Update Elasticsearch Config
# update <server name> with hostname of the elasticsearch node. elknode-01,elknode-02,elknode-03
cat << EOF > /etc/elasticsearch/elasticsearch.yml
cluster.name: "testcluster"
node.name: "<server name>"
node.master: true
node.data: true
node.ingest: true
search.remote.connect: false
network.host: 0.0.0.0   
path.data: /var/lib/elasticsearch
path.logs: /var/log/elasticsearch
discovery.seed_hosts: ["kibanaserver", "elknode-01", "elknode-02", "elknode-03"]
cluster.initial_master_nodes: ["kibanaserver", "elknode-01", "elknode-02", "elknode-03"]
EOF

# Restart Services
systemctl restart elasticsearch.service

# Check Service Status
systemctl status elasticsearch.service
```



**Test Cluster**
```
# Cluster Health
curl -XGET http://localhost:9200/_cluster/health?pretty
{
  "cluster_name" : "testcluster",
  "status" : "green",
  "timed_out" : false,
  "number_of_nodes" : 1,
  "number_of_data_nodes" : 1,
  "active_primary_shards" : 4,
  "active_shards" : 4,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 0,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 100.0
}


# Test Elasticsearch
curl -XGET http://localhost:9200
{
  "name" : "kibanaserver",
  "cluster_name" : "testcluster",
  "cluster_uuid" : "Su58cux7RACl8JeYLvwFdQ",
  "version" : {
    "number" : "7.7.0",
    "build_flavor" : "default",
    "build_type" : "rpm",
    "build_hash" : "81a1e9eda8e6183f5237786246f6dced26a10eaf",
    "build_date" : "2020-05-12T02:01:37.602180Z",
    "build_snapshot" : false,
    "lucene_version" : "8.5.1",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}


# List Nodes
curl -XGET http://localhost:9200/_cat/nodes?v

ip       heap.percent ram.percent cpu load_1m load_5m load_15m node.role master name
10.0.1.4           11          89   4    0.04    0.22     0.29 dilmt     -      elknode-01
10.0.1.5            9          89   4    0.03    0.21     0.31 dilmt     *      elknode-02
10.0.1.6           30          89   4    0.02    0.21     0.32 dilmt     -      elknode-03
10.0.0.4           15          94   9    0.23    0.54     0.81 l         -      kibanaserver


# List indices
curl -XGET http://localhost:9200/_cat/indices?v

health status index                    uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   .apm-custom-link         yJD2qz7gQSmm8tEfK8LKxQ   1   1          0            0       416b           208b
green  open   .kibana_task_manager_1   IW_yNFW4QhK8fUdGrrQPYw   1   1          5           13    135.8kb         70.8kb
green  open   .apm-agent-configuration t31AlzFvS4CzTcuylMGzLg   1   1          0            0       416b           208b
green  open   .kibana_1                gBzQjJjeS6iQMfFVKLjcwA   1   1          4            0     61.1kb         30.5kb


# Shard allocation per node
curl -XGET http://localhost:9200/_cat/allocation?v

shards disk.indices disk.used disk.avail disk.total disk.percent host     ip       node
     3       71.2kb     8.5gb     22.9gb     31.4gb           27 10.0.1.4 10.0.1.4 elknode-01
     2       95.5kb     8.5gb     22.9gb     31.4gb           27 10.0.1.6 10.0.1.6 elknode-03
     3       30.9kb     8.5gb     22.9gb     31.4gb           27 10.0.1.5 10.0.1.5 elknode-02
```

# Reference Links
* [Elastic Site](https://www.elastic.co/)
* [Elastic Downloads](https://www.elastic.co/downloads/)
* [Elastic Elasticsearch](https://www.elastic.co/downloads/past-releases#elasticsearch)
