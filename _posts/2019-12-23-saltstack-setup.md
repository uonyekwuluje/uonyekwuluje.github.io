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

#### **Manual Installation**

**Server Inventory**

|Server Name    |  Ip Address      |   CPU   |    RAM
|---------------|------------------|------------------------
|saltmaster     |  192.168.1.167   |    1    |    2GB
|linuxminion1   |  192.168.1.168   |    1    |    2GB 
|windowsminion1 |  192.168.1.169   |    2    |    2GB


**Install Saltmaster**
```
# Folder Preparation
mkdir -p /srv/{salt,pillar}/{dev,prod}

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
sudo pip install requests==2.7.0 azure==4.0.0
sudo pip install boto awscli

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




**Install Linux Salt Minion**
```
# Update firewall
sudo firewall-cmd --permanent --zone=public --add-port=4505-4506/tcp
sudo firewall-cmd --reload

# Package Requirements
sudo yum update -y
sudo yum group install -y 'Development Tools'
sudo yum install -y libxml2 libxml2-devel libxslt libxslt-devel wget gcc
sudo yum install -y libffi-devel openssl-devel make openssl-devel bzip2-devel python-devel

Install Python Pip
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


**Install Windows Salt Minion**
note: change windows minion name as needed
```
$salt_version = "2019.2.2"
$saltmaster_ip = "${SALTMASTER_IP}"
$salt_minion_name = "windowsvm-02"

$url = "https://raw.githubusercontent.com/saltstack/salt-bootstrap/stable/bootstrap-salt.ps1"
$path = "C:\Users\azureuser\Downloads\bootstrap-salt.ps1"

Set-ExecutionPolicy RemoteSigned
[Net.ServicePointManager]::SecurityProtocol = [Net.ServicePointManager]::SecurityProtocol -bor [Net.SecurityProtocolType]::Tls12

(New-Object System.Net.WebClient).DownloadFile($url, $path)
C:\Users\azureuser\Downloads\bootstrap-salt.ps1 -minion $salt_minion_name -master $saltmaster_ip -version $salt_version
```
