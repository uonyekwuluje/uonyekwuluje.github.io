---
layout: post
title:  "Saltstack Setup and base configuration"
categories: Configuration Management
---

Saltstack or salt for for short is a configuration management tool based on python. It's primarily used in configuring servers.
This coveres on-prem and cloud servers. With appropriate modules, you can also extend coverage to include switches, routers, 
IoT devices etc.

#### **Definitions**

* **Master:** <>. 
* **Minion:** <>. 
* **Pillar:** <>.
* **Grains:** <>.


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
#!/bin/bash
SALT_VERSION="2019.2.2"

# Disabling Firewall
disable_firewall() {
  sudo systemctl disable firewalld 
  sudo sed -i s/^SELINUX=.*$/SELINUX=disabled/ /etc/selinux/config
}

# Folder Prep
folder_prep() {
  mkdir -p /srv/{salt,pillar,formulas,files}
  mkdir -p /srv/pillar/{dev,qa}
}

# cleanup
cleanup_tasks() {
  sudo yum remove -y salt-minion
}


disable_firewall
folder_prep


sudo yum update
curl -sL https://bootstrap.saltstack.com -o install_salt.sh
sudo sh install_salt.sh -P -M stable ${SALT_VERSION}
# updating config files
cat << 'EOF' > /etc/salt/master
interface: 0.0.0.0
auto_accept: True

file_roots:
  base:
    - /srv/salt
    - /srv/formulas
    - /srv/salt/roles
pillar_roots:
  base:
    - /srv/pillar
  dev:
    - /srv/pillar/dev
  qa:
    - /srv/pillar/qa
EOF

sudo systemctl start salt-master.service
sudo systemctl enable salt-master.service

cleanup_tasks

```




**Install Linux Salt Minion**
```
#!/bin/bash
# Substitute Salmaster IP Address and Salt Version
SALT_VERSION="2019.2.2"
SALTMASTER_IP="10.0.0.4"

# Disabling Firewall
disable_firewall() {
  sudo systemctl disable firewalld
  sudo sed -i s/^SELINUX=.*$/SELINUX=disabled/ /etc/selinux/config
}

# cleanup
cleanup_tasks() {
  sudo yum remove -y salt-master
}


disable_firewall

sudo yum update
curl -sL https://bootstrap.saltstack.com -o install_salt.sh
sudo sh install_salt.sh -P -A ${SALTMASTER_IP} stable ${SALT_VERSION}

# updating config files
cat << 'EOF' > /etc/salt/minion
master: ${SALTMASTER_IP}
EOF

sudo systemctl start salt-minion.service
sudo systemctl enable salt-minion.service

cleanup_tasks
```



**Install Windows Salt Minion**
note: change windows minion name as needed
```
$salt_version = "2019.2.2"
$saltmaster_ip = "10.0.0.4"
$salt_minion_name = "windowsvm-02"

$url = "https://raw.githubusercontent.com/saltstack/salt-bootstrap/stable/bootstrap-salt.ps1"
$path = "C:\Users\azureuser\Downloads\bootstrap-salt.ps1"

Set-ExecutionPolicy RemoteSigned
[Net.ServicePointManager]::SecurityProtocol = [Net.ServicePointManager]::SecurityProtocol -bor [Net.SecurityProtocolType]::Tls12

(New-Object System.Net.WebClient).DownloadFile($url, $path)
C:\Users\azureuser\Downloads\bootstrap-salt.ps1 -minion $salt_minion_name -master $saltmaster_ip -version $salt_version
```
