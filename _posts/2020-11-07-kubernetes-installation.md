---
layout: post
title:  "Kubernetes Installation"
categories: Kubernetes
---
Kubernetes is an open-source tool for managing and orchestrating containers. These containers can be built with Docker or other equivallent tool.
From a IaaS and SaaS perspective, most major providers have solutions for this. For example OpenShift by RedHat, AKS by Azure, EKS by Amazon etc.
These solutions shields the grunt work and makes this easier to use and adopt. This is great but also comes at a cost.<br>
In this post we will be looking at setting up a Kubernetes Cluster from the ground up. This option gives us the most control and flexibility.


## **Server Requirements**
We will be building a 3 node cluster comprised of one master and 2 worker nodes:

| Hostname      |  Ip Address      | Host Specifications | Operating System |
|---------------|------------------|---------------------|------------------|
|kube8-master-node |  192.168.1.160   |2 CPU, 8GB RAM, 20GB HDD | CentOS 7  |
|kube8-node-1 |  192.168.1.161   | 2 CPU, 4GB RAM, 20GB HDD | CentOS 7 |
|kube8-node-2 |  192.168.1.162   | 2 CPU, 4GB RAM, 20GB HDD | CentOS 7 |

*NOTE: The above should be updated based on your specification*

## **Package Installation and System Configuration**
Perform these tasks on all 3 hosts. This assumes you are logged in as a user with sudo privilleges.

**Update Hostname, Hosts file and install base packages:**<br>
```
sudo hostnamectl set-hostname <Hostname>

sudo bash -c 'cat <<EOF>> /etc/hosts
192.168.1.160 kube8-master-node
192.168.1.161 kube8-node-1
192.168.1.162 kube8-node-2
EOF'

sudo yum update -y
sudo yum group install -y 'Development Tools'
sudo yum install -y libxml2 libxml2-devel libxslt libxslt-devel wget gcc \
libffi-devel openssl-devel make openssl-devel bzip2-devel nc jq
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
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

sudo yum upgrade -y
sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
sudo systemctl enable kubelet 
sudo systemctl start kubelet 
```

<br>
**Install Docker:**<br>
```
sudo sudo yum-config-manager  --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo yum install docker-ce -y 
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
sudo systemctl start docker
```

<br>
**Disable SWAP:**<br>
```
sudo sed -i '/swap/d' /etc/fstab
sudo swapoff -a
```
Reboot all nodes when this is completed


### **Initialize Cluster:**
Log into the kubernetes Master and initialize the cluster.
```
sudo kubeadm init
```
This will initialize the cluster and output the connection string. This section is what you want to focus on

```
....
....
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.1.160:6443 --token ty30fx.l6kbncckfazhpgot \
    --discovery-token-ca-cert-hash sha256:9aa1ac3b77d34c9bee94f9c1282149308dd760fe1b45f959c1d6686abe2a91fc 
```
<br>
Test your installation by typeing this command ```kubectl get nodes```. You should see
```
NAME                STATUS     ROLES    AGE    VERSION
kube8-master-node   NotReady   master   4m9s   v1.19.3
```

Now, login to the other nodes and type this command:
```
kubeadm join 192.168.1.160:6443 --token ty30fx.l6kbncckfazhpgot \
    --discovery-token-ca-cert-hash sha256:9aa1ac3b77d34c9bee94f9c1282149308dd760fe1b45f959c1d6686abe2a91fc
``` 

Now log back to the master node and run this command ```kubectl get nodes```. If all goes well, you should see this
```
NAME                STATUS     ROLES    AGE     VERSION
kube8-master-node   NotReady   master   9m52s   v1.19.3
kube8-node-1        NotReady   <none>   41s     v1.19.3
kube8-node-2        NotReady   <none>   8s      v1.19.3
```
For detailed node information, type ```kubectl get nodes -o wide```. You should see
```
NAME                STATUS   ROLES    AGE    VERSION   INTERNAL-IP     EXTERNAL-IP   OS-IMAGE                KERNEL-VERSION                CONTAINER-RUNTIME
kube8-master-node   Ready    master   35m    v1.19.3   192.168.1.163   <none>        CentOS Linux 7 (Core)   3.10.0-1127.19.1.el7.x86_64   docker://19.3.13
kube8-node-1        Ready    <none>   114s   v1.19.3   192.168.1.164   <none>        CentOS Linux 7 (Core)   3.10.0-1127.19.1.el7.x86_64   docker://19.3.13
kube8-node-2        Ready    <none>   62s    v1.19.3   192.168.1.165   <none>        CentOS Linux 7 (Core)   3.10.0-1127.19.1.el7.x86_64   docker://19.3.13
```


**Create Pod network:**<br> 
We will be using Weavnet for our pod network. For more information on its use and configuration, see [Weave Net](https://www.weave.works/docs/net/latest/kubernetes/kube-addon/). For other options, see [Kubernetes Network Options](https://kubernetes.io/docs/concepts/cluster-administration/addons/)
```
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```
*NOTE: Wait for about 1 minutes and check all namespace pods*
Run this command:
```
kubectl get pods --all-namespaces
```
You should see this:
```
NAMESPACE     NAME                                        READY   STATUS    RESTARTS   AGE
kube-system   coredns-f9fd979d6-8kxcj                     1/1     Running   0          7m48s
kube-system   coredns-f9fd979d6-d877x                     1/1     Running   0          7m48s
kube-system   etcd-kube8-master-node                      1/1     Running   0          7m50s
kube-system   kube-apiserver-kube8-master-node            1/1     Running   0          7m50s
kube-system   kube-controller-manager-kube8-master-node   1/1     Running   0          7m50s
kube-system   kube-proxy-dlndf                            1/1     Running   0          7m48s
kube-system   kube-scheduler-kube8-master-node            1/1     Running   0          7m49s
kube-system   weave-net-wk5nd                             2/2     Running   0          3m15s
```
Your cluster is now ready.

## **Kubernetes Dashboard**
To enable the Kubernetes Dashboard, See [Kubernetes Web UI](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/). In summary, type the following:
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0/aio/deploy/recommended.yaml
```
Test and ensure all pods are up by typing this command ```kubectl get pods --all-namespaces```. You should see this
```
NAMESPACE              NAME                                         READY   STATUS    RESTARTS   AGE
kube-system            coredns-f9fd979d6-8kxcj                      1/1     Running   0          42m
kube-system            coredns-f9fd979d6-d877x                      1/1     Running   0          42m
kube-system            etcd-kube8-master-node                       1/1     Running   0          42m
kube-system            kube-apiserver-kube8-master-node             1/1     Running   0          42m
kube-system            kube-controller-manager-kube8-master-node    1/1     Running   0          42m
kube-system            kube-proxy-dlndf                             1/1     Running   0          42m
kube-system            kube-proxy-fnv9r                             1/1     Running   0          9m5s
kube-system            kube-proxy-mcppx                             1/1     Running   0          8m13s
kube-system            kube-scheduler-kube8-master-node             1/1     Running   0          42m
kube-system            weave-net-26q6n                              2/2     Running   0          8m13s
kube-system            weave-net-f9vrs                              2/2     Running   1          9m5s
kube-system            weave-net-wk5nd                              2/2     Running   0          38m
kubernetes-dashboard   dashboard-metrics-scraper-7b59f7d4df-q4hx4   1/1     Running   0          73s
kubernetes-dashboard   kubernetes-dashboard-74d688b6bc-2mnqp        1/1     Running   0          73s
```
At this point run ```kubectl proxy``` and check your browser on ```http://127.0.0.1:8001``` or what ever the prompt gives you



## **Reference Links**
* Kubernetes Cheat Sheet [Kubernetes](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
* Rancher Kubernetes Cheat Sheet [Rancher](https://rancher.com/learning-paths/how-to-manage-kubernetes-with-kubectl/)
* [kubectl Installation](https://kubernetes.io/docs/tasks/tools/install-kubectl)
* [Helm Installation](https://helm.sh/docs/intro/install)
