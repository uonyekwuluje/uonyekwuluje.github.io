---
layout: post
title:  "Kafaka Multinode Cluster Setup"
categories: Kafka
---

Kafka is a stream processing software platform. It is written in scala and java. Kafka aims to provide a unified, high-throughput, low-latency platform for handling real-time data feeds. Kafka is capable of connecting to external systems. Kafka can be setup on a single node or in cluster mode. For this post, we will focus on Multinode cluster setup.

### **Assumptions**
This post assumes the following
* You have basic systems administration skills
* You have a basic understanding of streams. See [Apache Kafka](https://kafka.apache.org/intro) for a condensed introduction
* You have configured a seperate partition for storage. We will use **/data** in this post
* You have configured your firewall and ports as needed.

### **Terminology**
To have a better understanding of kafka it is important to understand it's components, terminologies and how they fit together
* **Topics:** A Topic is a feed name to which messages are stored and published. We can read from and write to Topics.
* **Producers:** The application or system that sends messages to kafka.
* **Consumers:** The application or system that reads or consumes the data from the specific topic in kafka.
* **Broker:** The kafka instance tasked with message exchange. Depends heavily on storage
* **Zookeeper:** Zookeeper is used for managing cluster state and coordinating kafka brokers.
<br>
Kafka has a few APIs. I'll focus on the major apis relevant to this post:
    * Producer API – Publish streams of records.
    * Consumer API – Subscribe to topics and processes streams of records.
    * Connector API – Executes the reusable producer and consumer APIs that can link the topics to the existing applications.
    * Streams API – Converts the input streams to output and produces the result.
    * Admin API – Manage Kafka topics,brokers and other Kafka objects.

### **Cluster Layout**
We will be building a 3 node cluster with the following specs:<br>

| Specification      | Defaults |
| ----------- | ----------- |
| CPU      | 2Ghz       |
| Memory   | 8GB        |
| Disk   | 50Gb        |


### **Cluster Servers**
Our cluster will be made up of the following:<br>

| Hostname      | IP Address |
| ----------- | ----------- |
| kafkanode1      | 10.0.2.4       |
| kafkanode2   | 10.0.2.5        |
| kafkanode3   | 10.0.2.6        |

### **Operating System**
This post will be focused on CentOS/RHEL. At the time of this post, I'll be using CentOS 7/RHEL7

### **Firewall Configurations**
This update needs to be done on all 3 Nodes
* Using your favorite editor, create **/etc/firewalld/services/zooKeeper.xml**
```
    <?xml version="1.0" encoding="utf-8"?>
    <service>
      <short>ZooKeeper</short>
      <description>Firewall rule for ZooKeeper ports</description>
      <port protocol="tcp" port="2888"/>
      <port protocol="tcp" port="3888"/>
      <port protocol="tcp" port="2181"/>
    </service>
```
and **/etc/firewalld/services/kafka.xml**
```
    <?xml version="1.0" encoding="utf-8"?>
    <service>
      <short>Kafka</short>
      <description>Firewall rule for Kafka port</description>
      <port protocol="tcp" port="9092"/>
    </service>
```
* Activate the rules and restart the firewall
```
sudo firewall-cmd --permanent --add-service=zooKeeper
sudo firewall-cmd --permanent --add-service=kafka
sudo service firewalld restart
sudo firewall-cmd --list-services
```

### **Systems User**
Create kafka user in all the nodes and update permissions on data folder
```
sudo adduser kafka
sudo passwd kafka
sudo usermod -aG wheel kafka
sudo mkdir /data/zookeeper
sudo chown -R kafka:kafka /data/zookeeper
```

### **Package Requirements**
Run these on all 3 Nodes
```
sudo yum update -y
sudo yum install -y libxml2 libxml2-devel libxslt libxslt-devel wget vim make gcc telnet \
libffi-devel openssl-devel openssl-devel bzip2-devel java-11-openjdk-devel java-11-openjdk
```

### **Create Broker IDs**
Create Broker IDs on each node:<br>

| Hostname      | Command |
| ----------- | ----------- |
| kafkanode1      | sudo echo '1' > /data/zookeeper/myid       |
| kafkanode2   | sudo echo '2' > /data/zookeeper/myid        |
| kafkanode3   | sudo echo '3' > /data/zookeeper/myid       |

### **Install Kafka**
Download and install kafka on all 3 nodes:<br>
```
cd /tmp
wget https://downloads.apache.org/kafka/2.6.0/kafka_2.13-2.6.0.tgz
tar -zxvf kafka_2.13-2.6.0.tgz
sudo mv kafka_2.13-2.6.0 /opt/kafka
sudo mkdir /opt/kafka/logs
sudo chown -R kafka:kafka /opt/kafka
sudo rm -Rf kafka_2.13-2.6.0.tgz
```

### **Service Updates**
Create kafka and zookeeper service on all 3 nodes :<br>
**/lib/systemd/system/zookeeper.service**
```
[Unit]
Requires=network.target remote-fs.target
After=network.target remote-fs.target

[Service]
Type=simple
User=kafka
ExecStart=/opt/kafka/bin/zookeeper-server-start.sh /opt/kafka/config/zookeeper.properties
ExecStop=/opt/kafka/bin/zookeeper-server-stop.sh
Restart=on-abnormal

[Install]
WantedBy=multi-user.target
```
**/etc/systemd/system/kafka.service**
```
[Unit]
Requires=network.target remote-fs.target zookeeper.service
After=network.target remote-fs.target zookeeper.service

[Service]
Type=simple
User=kafka
ExecStart=/opt/kafka/bin/kafka-server-start.sh /opt/kafka/config/server.properties
ExecStop=/opt/kafka/bin/kafka-server-stop.sh
Restart=on-abnormal

[Install]
WantedBy=multi-user.target
```

### **Update Zookeeper**
Update zookeeper config on all 3 nodes: **/opt/kafka/config/zookeeper.properties**<br>
```
# The directory where the snapshot is stored.
dataDir=/data/zookeeper
# the port at which the clients will connect
clientPort=2181
clientPortAddress=kafkanode1
# disable the per-ip limit on the number of connections since this is a non-production config
maxClientCnxns=0
# Disable the adminserver by default to avoid port conflicts.
# Set the port to something non-conflicting if choosing to enable this
admin.enableServer=false
# admin.serverPort=8080
# The number of milliseconds of each tick
tickTime=2000

# The number of ticks that the initial synchronization phase can take
initLimit=10

# The number of ticks that can pass between 
# sending a request and getting an acknowledgement
syncLimit=5

# zoo servers
server.1=kafkanode1:2888:3888
server.2=kafkanode2:2888:3888
server.3=kafkanode3:2888:3888
```
*note: Change **clientPortAddress** as needed per node*


### **Update Server Properties**
Update Server Properties Config on all 3 nodes: **/opt/kafka/config/server.properties**. The following should be updated as needed
* ```broker.id=<id as configured above```
* ```listeners=PLAINTEXT://0.0.0.0:9092```
* ```advertised.listeners=PLAINTEXT://<kafka node name>:9092```
* ```zookeeper.connect=kafkanode1:2181,kafkanode2:2181,kafkanode3:2181```


### **Enable Services**
Enable Kafka and Zookeeper services on all nodes
```
sudo systemctl enable zookeeper.service
sudo systemctl enable kafka.service

sudo systemctl start zookeeper.service
sudo systemctl start kafka.service
```
After starting the services, verify they are working correctly.<br>
**Verify Zookeeper**
```
sudo systemctl status zookeeper.service
```
if this works well, you should see this
```
● zookeeper.service
   Loaded: loaded (/usr/lib/systemd/system/zookeeper.service; enabled; vendor preset: disabled)
   Active: active (running) since Thu 2020-10-22 08:03:55 UTC; 12s ago
 Main PID: 1859 (java)
   CGroup: /system.slice/zookeeper.service
           └─1859 java -Xmx512M -Xms512M -server -XX:+UseG1GC -XX:MaxGCPauseMillis=20 -XX:InitiatingHeapOccupancyPercent=35 -XX:+ExplicitGCInvokesConcurrent -XX:MaxInl...

Oct 22 08:04:03 kafkanode1 zookeeper-server-start.sh[1859]: at java.base/java.net.AbstractPlainSocketImpl.connectToAddress(AbstractPlainSocketImpl.java:242)
Oct 22 08:04:03 kafkanode1 zookeeper-server-start.sh[1859]: at java.base/java.net.AbstractPlainSocketImpl.connect(AbstractPlainSocketImpl.java:224)
Oct 22 08:04:03 kafkanode1 zookeeper-server-start.sh[1859]: at java.base/java.net.SocksSocketImpl.connect(SocksSocketImpl.java:403)
Oct 22 08:04:03 kafkanode1 zookeeper-server-start.sh[1859]: at java.base/java.net.Socket.connect(Socket.java:609)
Oct 22 08:04:03 kafkanode1 zookeeper-server-start.sh[1859]: at org.apache.zookeeper.server.quorum.QuorumCnxManager.initiateConnection(QuorumCnxManager.java:373)
Oct 22 08:04:03 kafkanode1 zookeeper-server-start.sh[1859]: at org.apache.zookeeper.server.quorum.QuorumCnxManager$QuorumConnectionReqThread.run(QuorumCnxManag...ava:436)
Oct 22 08:04:03 kafkanode1 zookeeper-server-start.sh[1859]: at java.base/java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1128)
Oct 22 08:04:03 kafkanode1 zookeeper-server-start.sh[1859]: at java.base/java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:628)
Oct 22 08:04:03 kafkanode1 zookeeper-server-start.sh[1859]: at java.base/java.lang.Thread.run(Thread.java:834)
Oct 22 08:04:06 kafkanode1 zookeeper-server-start.sh[1859]: [2020-10-22 08:04:06,382] WARN Exception causing close of session 0x0: ZooKeeperServer not running ...verCnxn)
Hint: Some lines were ellipsized, use -l to show in full.
```
**Verify Kafka**
```
sudo systemctl status kafka.service
```
if all works well, you should see
```
● kafka.service
   Loaded: loaded (/etc/systemd/system/kafka.service; enabled; vendor preset: disabled)
   Active: active (running) since Thu 2020-10-22 08:04:01 UTC; 13s ago
 Main PID: 2229 (java)
   CGroup: /system.slice/kafka.service
           └─2229 java -Xmx1G -Xms1G -server -XX:+UseG1GC -XX:MaxGCPauseMillis=20 -XX:InitiatingHeapOccupancyPercent=35 -XX:+ExplicitGCInvokesConcurrent -XX:MaxInlineL...

Oct 22 08:04:10 kafkanode1 kafka-server-start.sh[2229]: [2020-10-22 08:04:10,524] INFO Socket error occurred: kafkanode3/10.0.2.6:2181: Connection refused (or...ientCnxn)
Oct 22 08:04:12 kafkanode1 kafka-server-start.sh[2229]: [2020-10-22 08:04:12,005] INFO Opening socket connection to server kafkanode2/10.0.2.5:2181. Will not ...ientCnxn)
Oct 22 08:04:12 kafkanode1 kafka-server-start.sh[2229]: [2020-10-22 08:04:12,009] INFO Socket error occurred: kafkanode2/10.0.2.5:2181: Connection refused (or...ientCnxn)
Oct 22 08:04:12 kafkanode1 kafka-server-start.sh[2229]: [2020-10-22 08:04:12,670] INFO Opening socket connection to server kafkanode1/10.0.2.4:2181. Will not ...ientCnxn)
Oct 22 08:04:12 kafkanode1 kafka-server-start.sh[2229]: [2020-10-22 08:04:12,670] INFO Socket connection established, initiating session, client: /10.0.2.4:47...ientCnxn)
Oct 22 08:04:12 kafkanode1 kafka-server-start.sh[2229]: [2020-10-22 08:04:12,675] INFO Unable to read additional data from server sessionid 0x0, likely server...ientCnxn)
Oct 22 08:04:12 kafkanode1 kafka-server-start.sh[2229]: [2020-10-22 08:04:12,935] INFO Opening socket connection to server kafkanode3/10.0.2.6:2181. Will not ...ientCnxn)
Oct 22 08:04:12 kafkanode1 kafka-server-start.sh[2229]: [2020-10-22 08:04:12,937] INFO Socket error occurred: kafkanode3/10.0.2.6:2181: Connection refused (or...ientCnxn)
Oct 22 08:04:14 kafkanode1 kafka-server-start.sh[2229]: [2020-10-22 08:04:14,621] INFO Opening socket connection to server kafkanode2/10.0.2.5:2181. Will not ...ientCnxn)
Oct 22 08:04:14 kafkanode1 kafka-server-start.sh[2229]: [2020-10-22 08:04:14,622] INFO Socket error occurred: kafkanode2/10.0.2.5:2181: Connection refused (or...ientCnxn)
Hint: Some lines were ellipsized, use -l to show in full.
```


### **Tests and Verification**
sudo as user kafka and run the following 
* ```jps```. You should see this
```
1859 QuorumPeerMain
2742 Kafka
3276 Jps
```
* Verify Brokers **/opt/kafka/bin/zookeeper-shell.sh kafkanode1:2181 ls /brokers/ids**. You should see
```
    Connecting to kafkanode1:2181

    WATCHER::

    WatchedEvent state:SyncConnected type:None path:null
    [1, 2, 3]
```
you can repeat for node 2 and 3*
* Create a Topic with 3 partitions and replicationfactor of 3
```/opt/kafka/bin/kafka-topics.sh --create --zookeeper kafkanode1:2181,kafkanode2:2181,kafkanode3:2181 --topic testtopic --partitions 3 --replication-factor 3 --config cleanup.policy=delete --config delete.retention.ms=60000```
* Describe Topic. ```/opt/kafka/bin/kafka-topics.sh --describe --zookeeper kafkanode1:2181 --topic testtopic```. You should see this
```
    Topic: testtopic	PartitionCount: 3	ReplicationFactor: 3	Configs: cleanup.policy=delete,delete.retention.ms=60000
	    Topic: testtopic	Partition: 0	Leader: 1	Replicas: 1,3,2	Isr: 1,3,2
	    Topic: testtopic	Partition: 1	Leader: 2	Replicas: 2,1,3	Isr: 2,1,3
	    Topic: testtopic	Partition: 2	Leader: 3	Replicas: 3,2,1	Isr: 3,2,1
```
* Start Producer on one node and Consumer on another:
    * Producer on kafkanode1
    ```
    /opt/kafka/bin/kafka-console-producer.sh --broker-list kafkanode1:9092,kafkanode2:9092,kafkanode3:9092 --topic testtopic
    ```
    * Consumer on kafkanode2:
    ```
    /opt/kafka/bin/kafka-console-consumer.sh --bootstrap-server kafkanode1:9092,kafkanode2:9092,kafkanode3:9092 --topic testtopic --from-beginning
    ```
    Type a string on the producer console and you should see the output on the consumer
    ```
    [kafka@kafkanode1 ~]$ /opt/kafka/bin/kafka-console-producer.sh --broker-list kafkanode1:9092,kafkanode2:9092,kafkanode3:9092 --topic testtopic
>Men This feels good

    [kafka@kafkanode3 ~]$ /opt/kafka/bin/kafka-console-consumer.sh --bootstrap-server kafkanode1:9092,kafkanode2:9092,kafkanode3:9092 --topic testtopic --from-beginning
Men This feels good
    ```
