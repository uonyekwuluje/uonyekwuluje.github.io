---
layout: post
title:  "Kafaka Multinode Cluster Setup"
categories: Kafka
---

Kafka is a stream processing software platform. It is written in scala and java. Kafka aims to provide a unified, high-throughput, low-latency platform for handling real-time data feeds. Kafka is capable of connecting to external systems. Kafka can be setup on a single node or in cluster mode. For this post, we will focus on Multinode cluster setup.

### Assumptions
This post assumes the following
* You have basic systems administration skills
* You have a basic understanding of streams. See [Apache Kafka](https://kafka.apache.org/intro) for a condensed introduction
* You have configured a seperate partition for storage. We will use **/data** in this post
* You have configured your firewall and ports as needed.

### Terminology
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

### Cluster Layout
We will be building a 3 node cluster with the following specs
| Specification     | Default   |
| ------------------| --------- |
| CPU               | 2Ghz      |
| Memory            | 8GB       |
| Disk              | 50Gb      |

### Cluster Servers
Our cluster will be made up of the following
| Hostname     |  IP Address |
| -------------|-------------| 
| kafkanode1   | 10.0.2.4    |
| kafkanode2   | 10.0.2.5    |
| kafkanode3   | 10.0.2.6    |


