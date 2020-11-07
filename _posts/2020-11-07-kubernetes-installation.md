---
layout: post
title:  "Kubernetes Installation"
categories: Kubernetes
---
Kubernetes is an open-source tool for managing and orchestrating containers. These containers can be built with Docker or other equivallent tool.
From a IaaS and SaaS perspective, most major providers have solutions for this. For example OpenShift by RedHat, AKS by Azure, EKS by Amazon etc.
These solutions shields the grunt work and makes this easier to use and adopt. This is great but also comes at a cost.<br>
In this post we will be looking at setting up a Kubernetes Cluster from the ground up. This option gives us the most control and flexibility.


### **Server Requirements**
We will be building a 3 node cluster comprised of one master and 2 worker nodes:
|    Hostname          |  IP Address     |    Host Specifications       |   Operating System  |
|:-------------------: |:---------------:| :--------------------------: | :-----------------: |
|  kube8-master-node   | 192.168.1.160   |   2 CPU, 8GB RAM, 20GB HDD   |      CentOS 8       |  
|  kube8-node-1        | 192.168.1.161   |   2 CPU, 4GB RAM, 20GB HDD   |      CentOS 8       |
|  kube8-node-2        | 192.168.1.162   |   2 CPU, 4GB RAM, 20GB HDD   |      CentOS 8       |
*NOTE: The above should be updated based on your specification*

### **Package Installation and System Configuration**
Perform these tasks on all 3 hosts. This assumes you are logged in as a user with sudo privilleges.

**Update Hostname, Hosts file and install base packages:**<br>
```
sudo hostnamectl set-hostname <Hostname>

sudo bash -c 'cat <<EOF>> /etc/hosts
192.168.1.160 kube8-master-node
192.168.1.161 kube8-node-1
192.168.1.162 kube8-node-2
EOF'

sudo dnf update -y
sudo dnf group install -y 'Development Tools'
sudo dnf install -y libxml2 libxml2-devel libxslt libxslt-devel wget gcc \
libffi-devel openssl-devel make openssl-devel bzip2-devel nc jq
sudo dnf install -y yum-utils device-mapper-persistent-data lvm2
```

**Update Firewall & Selinux:**<br>
```
sudo sed -i 's/^SELINUX=.*/SELINUX=disabled/g' /etc/selinux/config
sudo systemctl disable firewalld
```
*NOTE: The above should not be done in Production. See below and update based on your needs*
```
firewall-cmd --add-masquerade --permanent
firewall-cmd --permanent --add-port=6443/tcp
firewall-cmd --permanent --add-port=2379-2380/tcp
firewall-cmd --permanent --add-port=10250/tcp
firewall-cmd --permanent --add-port=10251/tcp
firewall-cmd --permanent --add-port=10252/tcp
firewall-cmd --permanent --add-port=10255/tcp
firewall-cmd --permanent --add-port=6783/tcp
firewall-cmd --permanent --add-port=10255/tcp
firewall-cmd --permanent --add-port=30000-32767/tcp
firewall-cmd --reload
```
<br>
**Load Modules:**<br>
```
sudo modprobe br_netfilter
sudo modprobe ip6_udp_tunnel
sudo modprobe ip_set
sudo modprobe ip_set_hash_ip
sudo modprobe ip_set_hash_net
sudo modprobe iptable_filter
sudo modprobe iptable_nat
sudo modprobe iptable_mangle
sudo modprobe iptable_raw
sudo modprobe nf_conntrack_netlink
sudo modprobe nf_conntrack
sudo modprobe nf_conntrack_ipv4
sudo modprobe nf_defrag_ipv4
sudo modprobe nf_nat
sudo modprobe nf_nat_ipv4
sudo modprobe nf_nat_masquerade_ipv4
sudo modprobe nfnetlink
sudo modprobe udp_tunnel
sudo modprobe veth
sudo modprobe vxlan
sudo modprobe x_tables
sudo modprobe xt_addrtype
sudo modprobe xt_conntrack
sudo modprobe xt_comment
sudo modprobe xt_mark
sudo modprobe xt_multiport
sudo modprobe xt_nat
sudo modprobe xt_recent
sudo modprobe xt_set
sudo modprobe xt_statistic
sudo modprobe xt_tcpudp
```
<br>
**Create Custom sysctl:**<br>
```
sudo bash -c 'cat <<EOF>> /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF'
```
<br>
**Create Kubernetes Repository:**<br>
```
sudo bash -c 'cat <<EOF>> /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-$basearch
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kubelet kubeadm kubectl
EOF'

sudo dnf upgrade -y
sudo dnf install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
sudo systemctl enable kubelet 
```

<br>
**Install Docker:**<br>
```
sudo dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo
sudo dnf install docker-ce -y --nobest
sudo usermod -aG docker $USER

sudo mkdir /etc/docker
sudo mkdir -p /etc/systemd/system/docker.service.d

sudo bash -c 'cat <<EOF>> /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ]
}
EOF'

sudo systemctl enable docker
```

<br>
**Disable SWAP:**<br>
```
sudo vim /etc/fstab
```
Reboot all nodes when this is completed

#### **Requirements**
For our barebones install

* Local installation of [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl)
* Local installation of [helm](https://helm.sh/docs/intro/install)

**kubectl test** <br>
```
kubectl version --short
```
You should see this:
```
Client Version: v1.16.0
Server Version: v1.14.8
```
<hr>
#### **Command Cheat Sheet**

|         Command                 |      Description             |              Reference Links               |
|:------------------------------: |:----------------------------:| :----------------------------------------: | 
|  `kubectl config view`          | View current config          |   [kubernetes](https://kubernetes.io/)     |
|  `kubectl config get-contexts`  | Get all installed contexts   |   [kubernetes](https://kubernetes.io/)     | 

<hr>




#### **Reference Links**
* Kubernetes Cheat Sheet [Kubernetes](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
* Rancher Kubernetes Cheat Sheet [Rancher](https://rancher.com/learning-paths/how-to-manage-kubernetes-with-kubectl/)
