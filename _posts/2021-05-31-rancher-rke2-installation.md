---
layout: post
title:  "Rancher Kubernetes Installation"
categories: Kubernetes
---
Rancher is an open-source container management platform, providing an graphical interface making container management easier. It has a 
smaller footprint and makes the installation process much easier.

## **Server Requirements**
We will be building a 3 node cluster comprised of one master and 2 worker nodes:

| Hostname     |   Ip Address     |  Host Specifications     | Operating System |
|--------------|------------------|--------------------------|------------------|
| kube-master  |  192.168.1.160   | 2 CPU, 5GB RAM, 30GB HDD | Ubuntu 20.04 LTS |
| kube-node1   |  192.168.1.161   | 2 CPU, 7GB RAM, 30GB HDD | Ubuntu 20.04 LTS |
| kube-node2   |  192.168.1.162   | 2 CPU, 7GB RAM, 30GB HDD | Ubuntu 20.04 LTS |

*NOTE: The above should be updated based on your specification*

## **Package Installation and System Configuration**
Perform these tasks on all 3 hosts. This assumes you are logged in as a user with sudo privilleges.

**Update Hostname, Hosts file and install base packages:**<br>
Update Hostname with this command `sudo hostnamectl set-hostname <Hostname>`

Update Hosts file
```
sudo bash -c 'cat <<EOF>> /etc/hosts
192.168.1.160 kube-master
192.168.1.161 kube-node1
192.168.1.162 kube-node2
EOF'
```

Install Packages
```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo apt-key fingerprint 0EBFCD88
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
sudo apt-get update && sudo apt-get install -y docker-ce docker-ce-cli containerd.io jq net-tools npm \
docker-compose ntp ntpdate linux-image-extra-virtual ca-certificates curl software-properties-common 
sudo usermod -aG docker $USER

sudo timedatectl set-timezone America/New_York
sudo apt upgrade -y && sudo reboot
```

## **Install RKe2 Master:**<br>
Install Helm3.
```
sudo curl -sL https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
```

Install RKe2
```
curl -sfL https://get.rke2.io | sudo sh - 
```

Enable & start service
```
sudo systemctl enable rke2-server.service
sudo systemctl start rke2-server.service
```

Create Kube Config
```
mkdir ~/.kube
sudo cp /etc/rancher/rke2/rke2.yaml ~/.kube/config
sudo chown -R $USER:$USER ~/.kube 

sudo bash -c 'cat <<EOF>> /etc/profile
PATH=$PATH:/var/lib/rancher/rke2/bin
EOF'
```

Wait for a few seconds and check for rke2 join token
```
sudo cat /var/lib/rancher/rke2/server/node-token 
```
if all goes well, you should see something like `K1035fb94adee7823fcc6d1d5af6fd1f0f10611348a14dc2e7b8ccd2fcdcff36703::server:83b140f0544ac7598acb7b2e2d46a29f`



## **Install RKe2 Agents:**<br>
Initialize and install agent
```
curl -sfL https://get.rke2.io | INSTALL_RKE2_TYPE="agent" sudo sh -
```

Update & configure agent
```
sudo mkdir -p /etc/rancher/rke2/
sudo bash -c 'cat <<EOF> /etc/rancher/rke2/config.yaml
server: https://kube-master:9345
token: K1035fb94adee7823fcc6d1d5af6fd1f0f10611348a14dc2e7b8ccd2fcdcff36703::server:83b140f0544ac7598acb7b2e2d46a29f
EOF'
```

Enable & start agent
```
sudo systemctl enable rke2-agent.service
sudo systemctl start rke2-agent.service
```


## **Master RKe2 Verification:**<br>
Log back into the master and type this command `kubectl get nodes`. You should see this
```
NAME          STATUS   ROLES                       AGE     VERSION
kube-master   Ready    control-plane,etcd,master   12m     v1.20.7+rke2r2
kube-node1    Ready    <none>                      3m12s   v1.20.7+rke2r2
kube-node2    Ready    <none>                      3m18s   v1.20.7+rke2r2
```

## **Reference Links**
* Kubernetes Cheat Sheet [Kubernetes](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
* Rancher Kubernetes Cheat Sheet [Rancher](https://rancher.com/learning-paths/how-to-manage-kubernetes-with-kubectl/)
* [kubectl Installation](https://kubernetes.io/docs/tasks/tools/install-kubectl)
* [Helm Installation](https://helm.sh/docs/intro/install)
