---
layout: post
title:  "Saltstack Setup and base configuration"
categories: Configuration Management
---

Saltstack or salt for for short is a configuration management tool based on python. It's primarily used in configuring servers.
This coveres on-prem and cloud servers. With appropriate modules, you can also extend coverage to include switches, routers, 
IoT devices etc.

#### **Why Salt**
With options in a crowded space that includes the likes of Ansible, Puppet, Chef, Fabric etc. Why Salt? Well, the answer is long but I'll keep it
simple. Of all the other tools I found salt to be much better in mixed environments. An environment of this nature includes Linux, Windows, MACs.
With the right modules I can also use this for cloud deployments and orchestration.
Triggering of actions in response to events is one I found to be very good with the use of reactor. Some of the other options are not as robust as
salt. I also found myself doing lots of scripting and plumbing to get other tools working. 


**Server Inventory**

|Server Name                |  Ip Address      |   CPU   |    RAM
|---------------------------|------------------|------------------------
|saltmaster                 |  192.168.1.167   |    1    |    2GB
|saltminion-privlinux-vm0   |  192.168.1.168   |    1    |    2GB 
|saltminion-privlinux-vm1   |  192.168.1.169   |    1    |    2GB
|saltminion-privwindo-vm1   |  192.168.1.170   |    2    |    2GB


**Install and Configure Saltmaster**
```
# Folder Preparation
mkdir -p /srv/{salt,pillar,files}/{dev,prod}

# Firewall Setup
sudo firewall-cmd --permanent --zone=public --add-port=4505-4506/tcp
sudo firewall-cmd --reload

# Package Requirements
sudo yum update -y
sudo yum group install -y 'Development Tools'
sudo yum install -y libxml2 libxml2-devel libxslt libxslt-devel wget gcc
sudo yum install -y libffi-devel openssl-devel make openssl-devel bzip2-devel python-devel

# Install Python Pip
cd /tmp
curl "https://bootstrap.pypa.io/get-pip.py" -o "get-pip.py"
sudo python get-pip.py
sudo pip install --upgrade pip
pip -V

Pip Azure Install Requirements
sudo pip install packaging cryptography --upgrade
sudo pip install requests~=2.20.0 azure==4.0.0 
sudo pip install boto awscli gitfs pygit2 GitPython

# Install Salt Repository for latest salt
sudo yum install -y https://repo.saltstack.com/yum/redhat/salt-repo-latest.el7.noarch.rpm
sudo yum clean expire-cache

# Install SaltStack
sudo yum install -y salt-master salt-minion salt-ssh salt-syndic salt-cloud salt-api

# updating config files
cat << 'EOF' > /etc/salt/master
interface: 0.0.0.0
hash_type: sha256
auto_accept: True
file_recv: True
file_recv_max_size: 100

file_roots:
   base:
     - /srv/salt/
   dev:
     - /srv/salt/dev/
   prod:
     - /srv/salt/prod/

pillar_roots:
   base:
     - /srv/pillar
   dev:
     - /srv/pillar/dev/
   prod:
     - /srv/pillar/prod/
EOF

# Enable and Restart Services
sudo systemctl enable salt-master.service
sudo systemctl start salt-master.service
sudo systemctl restart salt-master.service
sudo systemctl status salt-master.service
```


**Install and Configure Linux Salt Minion**
```
SALTMASTER_IP="xx.xx.xx.xx"
# Update firewall
sudo firewall-cmd --permanent --zone=public --add-port=4505-4506/tcp
sudo firewall-cmd --reload

# Package Requirements
sudo yum update -y
sudo yum group install -y 'Development Tools'
sudo yum install -y libxml2 libxml2-devel libxslt libxslt-devel wget gcc
sudo yum install -y libffi-devel openssl-devel make openssl-devel bzip2-devel python-devel

# Install Python Pip
cd /tmp
curl "https://bootstrap.pypa.io/get-pip.py" -o "get-pip.py"
sudo python get-pip.py
sudo pip install --upgrade pip

# Install Latest Salt Repo
sudo yum install -y https://repo.saltstack.com/yum/redhat/salt-repo-latest.el7.noarch.rpm
sudo yum clean expire-cache

# Install SaltStack
sudo yum install -y salt-minion

# updating config files
cat << 'EOF' > /etc/salt/minion
master: ${SALTMASTER_IP}
EOF

# Update and Restart service
sudo systemctl enable salt-minion.service
sudo systemctl start salt-minion.service
sudo systemctl restart salt-minion.service
sudo systemctl status salt-minion.service
```


**Install and Configure Windows Salt Minion**
note: change windows minion name as needed
```
$salt_version = "2019.2.2"
$saltmaster_ip = "${SALTMASTER_IP}"
$salt_minion_name = "saltminion-privwindo-vm1"

$url = "https://raw.githubusercontent.com/saltstack/salt-bootstrap/stable/bootstrap-salt.ps1"
$path = "C:\Users\azureuser\Downloads\bootstrap-salt.ps1"

Set-ExecutionPolicy RemoteSigned
[Net.ServicePointManager]::SecurityProtocol = [Net.ServicePointManager]::SecurityProtocol -bor [Net.SecurityProtocolType]::Tls12

(New-Object System.Net.WebClient).DownloadFile($url, $path)
C:\Users\azureuser\Downloads\bootstrap-salt.ps1 -minion $salt_minion_name -master $saltmaster_ip -version $salt_version
```


**Test Setup**
Run the command below to verify your setup:
```
sudo salt '*' test.ping
```
you should find something like this:
```
saltminion-privlinux-vm0.internal.cloudapp.net:
    True
saltminion-privlinux-vm1.internal.cloudapp.net:
    True
saltminion-privwindo-vm1.internal.cloudapp.net:
    True
```

**NOTE**: *The above setup is designed to be used for a POC. This is not secure since the master accepts any connection. A more secure approach will be to update the minions with the Masters Fingerprint. To do this, please use the setup below*


**Grab Masters Fingerprint**: <br>
To get the salt masters fingerprint, type the command below:
```
sudo salt-key -F master
```
you should see something like:
```
Local Keys:
master.pem:  8e:c4:f7:2d:83:17:5c:d1:7c:77:98:66:14:9d:da:02:87:37:03:8a:42:03:5f:9d:9d:bd:d0:f3:82:f9:9a:03
master.pub:  ee:2f:8c:25:42:48:fa:90:8a:f4:78:f2:8d:8a:d3:c5:12:38:73:62:7a:fe:96:0b:2d:96:57:38:1e:da:56:62
```

In the Saltminions, update the config with this option:
```
master_finger: <master.pub> # the master.pub fingerprint from above
```
restart the minion:
```
sudo systemctl restart salt-minion
```
That's all
