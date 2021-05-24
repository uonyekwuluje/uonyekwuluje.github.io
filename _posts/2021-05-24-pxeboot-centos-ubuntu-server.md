---
layout: post
title:  "PXE Boot Server | Kickstart & Preseed"
categories: Kickstart & Preseed
---


This post is a follow up from [CentOS PXE Boot Server](http://blog.infracid.com/automation,/pxe/2020/12/27/pxeboot-server.html). The aim and objective is to setup a PXE boot server
capable of using existing DHCP Server in the network and also serving both CentOS and Ubuntu servers.

### **Requirements**
In order to setup the PXE Boot Server, you need the following:
* Ubuntu 20.04 LTS
* Admin Privilleges 
* ISOs for your operating system. *In this post, we will be using CentOS 8 and Ubuntu 20.04 LTS* 

### **Setup**
Update and install the following packages on the PXE Server
```
sudo apt-get update 
sudo apt-get install -y jq net-tools npm ntp ntpdate ca-certificates \
dnsmasq curl software-properties-common pxelinux syslinux-common apache2
```

Setup Timezone
```
sudo timedatectl set-timezone America/New_York
```
reboot the server after this update is completed

### **PXE Configuration**
Create folders for PXE configs
```
sudo mkdir -p /var/lib/tftpboot
sudo mkdir -p /var/lib/tftpboot/pxelinux.cfg
sudo mkdir -p /var/lib/tftpboot/{ubuntu20,centos8}
sudo mkdir -p /var/lib/tftpboot/pxelinux.cfg/{centos8,ubuntu20}
sudo mkdir -p /var/www/html/{ubuntu20,centos8}
```

Copy boot configs to their required locations
```
sudo cp /usr/lib/PXELINUX/pxelinux.0 /var/lib/tftpboot/
sudo cp /usr/lib/syslinux/modules/bios/ldlinux.c32 /var/lib/tftpboot/
sudo cp /usr/lib/syslinux/modules/bios/libcom32.c32 /var/lib/tftpboot/
sudo cp /usr/lib/syslinux/modules/bios/libutil.c32 /var/lib/tftpboot/
sudo cp /usr/lib/syslinux/modules/bios/vesamenu.c32 /var/lib/tftpboot/
sudo cp /usr/lib/syslinux/modules/bios/menu.c32 /var/lib/tftpboot/
```

Backup and configure `dnsmasq.conf`
```
sudo cp /etc/dnsmasq.conf /etc/dnsmasq.conf-orig

sudo bash -c 'cat <<EOF> /etc/dnsmasq.conf
# Disable DNS Server
port=0

# Enable DHCP logging
log-dhcp

# Respond to PXE requests for the specified network;
# run as DHCP proxy
dhcp-range=192.168.1.0,proxy 
dhcp-boot=pxelinux.0

# Provide network boot option called "Network Boot".
pxe-service=x86PC,"Network Boot",pxelinux

enable-tftp
tftp-root=/var/lib/tftpboot
EOF'
```

Backup and configure `dnsmasq` default
```
sudo cp  /etc/default/dnsmasq /etc/default/dnsmasq.orig

sudo bash -c 'cat <<EOF> /etc/default/dnsmasq
ENABLED=1
DNSMASQ_EXCEPT=lo
CONFIG_DIR=/etc/dnsmasq.d,.dpkg-dist,.dpkg-old,.dpkg-new
EOF'
```

Enable, start `dnsmasq` service and check status
```
sudo systemctl enable dnsmasq.service
sudo systemctl start dnsmasq.service
sudo systemctl status dnsmasq.service
```

### **Operating System Boot Configs***
Setup Ubuntu 20 PXE Boot Configs
```
mkdir ubuntu20
wget http://archive.ubuntu.com/ubuntu/dists/focal-updates/main/installer-amd64/current/legacy-images/netboot/netboot.tar.gz -O ubuntu20/netboot.tar.gz
cd ubuntu20
tar -zxvf netboot.tar.gz
sudo cp ubuntu-installer/amd64/{initrd.gz,linux} /var/lib/tftpboot/ubuntu20/
```

Setup CentOS 8 boot configs
```
wget http://mirrors.mit.edu/centos/8.3.2011/isos/x86_64/CentOS-8.3.2011-x86_64-boot.iso
sudo mount -o loop CentOS-8.3.2011-x86_64-boot.iso /mnt/
sudo cp /mnt/images/pxeboot/{initrd.img,vmlinuz} /var/lib/tftpboot/centos8/
```



