---
layout: post
title:  "GitHub Actions Self Hosted Runner Setup"
categories: Automation CICD
---

GitHub Actions is an API for cause and effect on GitHub. Orchestrate and automate any workflow, based on any event. 
With GitHub Actions, workflows and steps are just code in a repository, so you can create, share, reuse your software 
development practices.
<br>
A core component of this process is the use of runners. Runners/Agents etc. are essentially the work horses of most
CICD systems and Github Actions is no exception. in this post, we will be focusing on installing and configuring 
self hosted runners.  

### **Hardware Requirements**
For the purpose of this post, setup or provision a VM with the following specs:
* Ubuntu 18.04/20.04 LTS.
* Admin Privilleges on the server
* Network and firewall configuration as needed. *In this post, we will be using CentOS 7 and CentOS8*
* GitHub Personal Access Token
* GitHub Organization Name

### **Package Requirements**
Update the server and install base packages
```
sudo apt upgrade
sudo apt install -y libncursesw5 libncursesw5-dev libncurses5 gcc-7 open-vm-tools open-vm-tools-dev \
gcc dkms libaio1 libcanberra-gtk-module build-essential linux-headers-$(uname -r) libssl-dev openssl \
zip zlib1g-dev cmake zlib1g-dev libxi-dev libxtst-dev curl gnupg2 libgtk2.0-dev jq 
```

**Docker Installation**<br>
This is optional but gives you an added dimension for container builds
```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo apt-key fingerprint 0EBFCD88
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-compose
sudo usermod -aG docker $USER
```

**Docker Machine Installation**<br>
This is optional but gives you an added dimension for extending functionality in other environments like building agents
in AWS, Azure, Goofle, VMware, Virtualbox etc
```
curl -L https://github.com/docker/machine/releases/download/v0.16.2/docker-machine-`uname -s`-`uname -m` > /tmp/docker-machine
chmod +x /tmp/docker-machine
sudo mv /tmp/docker-machine /bin/docker-machine
```
