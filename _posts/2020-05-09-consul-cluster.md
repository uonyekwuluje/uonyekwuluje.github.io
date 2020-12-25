---
layout: post
title:  "Consul Cluster Setup"
categories: consul, monitoring, service discovery
---

Consul is a service networking solution to connect and secure services. This tutorial will walk you through the steps for 
installing and configuring a 3 Node Consul Cluster on CentOS/RHEL 8.<br>
I put this together because I ran into an issue of dynamic host and service configuration using prometheus. Consul solved this
problem with relative ease. It's not perfect yet but I'll be updating this blog as I make progress. This ensures I have a point
of reference based on what has worked well for me.

#### **Use Case**
Consul is primary used for service discovery but it can also be configured to address other use case. Some of these are
* **Service Discovery:** Agents register configured services and other applications can use ir consume them as needed. 
* **Key/Value Store:** The Consul’s hierarchical key/value store can be used for dynamic configuration, coordination, leader election, feature flagging, and more. It has a simple and easy to use HTTP API. 
* **Health Checking:** Consul clients can be configured to do health checks, both for services and nodes. This information is helpful to monitor cluster health and routing of traffic away from unhealthy nodes.


#### **Server Specifications**

|Component &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; | Systems Specification |
|:------------- | --------------------: |
|Consul Servers |  2 CPU   4GB RAM      |

**NOTE:**
You can make changes as needed. The above is just a base systems spec.

#### **Server Inventory**

|Server Name    |  Ip Address      |
|---------------|------------------|
|consulserver01 |  192.168.1.137   |
|consulserver02 |  192.168.1.138   |
|consulserver03 |  192.168.1.139   |

#### **Port & Firewall Requirements**
The following ports should be enabled in your network

| Ports & Specs    | 
|------------------|
| 8300  – TCP      |
| 8301  – TCP & UDP|
| 8302  – TCP & UDP|
| 8400  – TCP      |
| 8500  – TCP      |
| 8600  – TCP & UDP|

iptable command below:
```
sudo firewall-cmd  --add-port={8300,8301,8302,8400,8500,8600}/tcp --permanent
sudo firewall-cmd  --add-port={8301,8302,8600}/udp --permanent
sudo firewall-cmd --reload
```

#### **Package Requirements**
Install required packages in all servers
```
sudo yum update -y
sudo yum group install -y 'Development Tools'
sudo yum install -y libxml2 libxml2-devel libxslt libxslt-devel wget gcc \
libffi-devel openssl-devel make openssl-devel bzip2-devel 
```

#### **Hosts & Name Resolution**
If a DNS server is absent you can make these changes in your ```/etc/hosts``` file to help with name resolution for the cluster
```
192.168.1.137  consulserver01
192.168.1.138  consulserver02
192.168.1.139  consulserver03
```

#### **Service Account**
We need a service account for our consul service. Type the commands below to create this account on all consul servers
```
sudo groupadd --system consul
sudo useradd -s /sbin/nologin --system -g consul consul
```

#### **Download and Install Consul**
Install and configure consul
```
CONSUL_VERSION="1.9.1"
cd /usr/local/bin
sudo curl -o consul.zip https://releases.hashicorp.com/consul/${CONSUL_VERSION}/consul_${CONSUL_VERSION}_linux_amd64.zip

unzip consul.zip
sudo rm -f consul.zip
```

**Lib, Service and Config Setup**<br>
Create config directory and update file ownership
```
consul -autocomplete-install
complete -C /usr/local/bin/consul consul
sudo mkdir -p /var/lib/consul /etc/consul.d/server
sudo chown -R consul:consul /var/lib/consul /etc/consul.d/server
sudo chmod -R 775 /var/lib/consul /etc/consul.d/server
```
** Generate Cluster Key**<br>
Generate cluster key:
```
consul keygen
```
You should see something like ```mxFwtM5XekGqbWmXBgknv4Up6FXaTVLS1Gy2aj2sK2s=```

**Create Consul Service**<br>
Create the consul service. ```/etc/systemd/system/consul-master.service``` ***NOTE: Update the node server entry for each server***
```
[Unit]
Description=Consul Service Discovery Agent
Documentation=https://www.consul.io/
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=consul
Group=consul
ExecStart=/usr/local/bin/consul agent \
	-node=<consule servername> \
	-config-dir=/etc/consul.d/server

ExecReload=/bin/kill -HUP $MAINPID
KillSignal=SIGINT
TimeoutStopSec=5
Restart=on-failure
SyslogIdentifier=consul

[Install]
WantedBy=multi-user.target

```

**Create Consul Config**<br>
Create the consul service config. ```/etc/consul.d/server/config.json``` ***NOTE: Update the node server entry for each server***
this section assumes you have configured your ```/etc/hosts``` file as directed above
```
{
     "advertise_addr": "<your consul server ip address>",
     "bind_addr": "<your consul server ip address>",
     "bootstrap_expect": 3,
     "client_addr": "0.0.0.0",
     "datacenter": "<Your DataCenter Name>",
     "data_dir": "/var/lib/consul",
     "domain": "consul",
     "enable_script_checks": true,
     "dns_config": {
         "enable_truncate": true,
         "only_passing": true
     },
     "enable_syslog": true,
     "encrypt": "<consul cluster key>",
     "leave_on_terminate": true,
     "log_level": "INFO",
     "rejoin_after_leave": true,
     "retry_join": [
         "<cluster server hostname 1>",
         "<cluster server hostname 2>",
         "<cluster server hostname 3>"
     ],
     "server": true,
     "start_join": [
         "<cluster server hostname 1>",
         "<cluster server hostname 2>",
         "<cluster server hostname 3>"
     ],
     "ui": true
}
```

**Validate Config**<br>
Validate the consul config
```
consul validate /etc/consul.d/server/config.json
```
if all goes well you should see this ```Configuration is valid!```. Restart the consul service on each server and test membership
```
sudo systemctl restart consul-master
```
Check Cluster Membership with ```/usr/local/bin/consul members``` and you should see this:
```
Node                       Address             Status  Type    Build  Protocol  DC       Segment
consulserver01             192.168.1.137:8301  alive   server  1.9.1  2         DCProd01  <all>
consulserver02             192.168.1.138:8301  alive   server  1.9.1  2         DCProd01  <all>
consulserver03             192.168.1.139:8301  alive   server  1.9.1  2         DCProd01  <all>
```


#### **Test UI**
Finally open your browser and test the UI with ```http://<any server ip>:8500/ui```



#### **Reference**
* [Consul](https://www.consul.io/)
* [Consul Downloads](https://releases.hashicorp.com/consul/)
