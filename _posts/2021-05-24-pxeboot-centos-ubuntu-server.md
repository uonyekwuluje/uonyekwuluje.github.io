---
layout: post
title:  "PXE Boot Server | Kickstart & Preseed"
categories: Kickstart & Preseed
---

This post is a follow up from (CentOS PXE Boot Server)[http://testdocs]. The aim and objective is to setup a PXE boot server
capable of using existing DHCP Server in the network and also serving both CentOS and Ubuntu servers.

### **Requirements**
In order to setup the PXE Boot Server, you need the following:
* Ubuntu 20.04 LTS
* Admin Privilleges 
* ISOs for your operating system. *In this post, we will be using CentOS 7, CentOS 8 and Ubuntu 18.04 LTS* 

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




***NOTE:*** *Adjust your firewall rules and selinux as needed*
<br>
**Configure DHCP**<br>
***NOTE:*** *Change 192.168.1.196 to your PXE Boot Address*
<br>
Create your DHCP Config. ```/etc/dhcp/dhcpd.conf```.
```
# dhcpd.conf
# Sample configuration file for ISC dhcpd

# option definitions common to all supported networks...

default-lease-time 600;
max-lease-time 7200;

# Use this to enble / disable dynamic dns updates globally.
#ddns-update-style none;
ddns-update-style interim;
ignore client-updates;
authoritative;
allow booting;
allow bootp;
allow unknown-clients;

# If this DHCP server is the official DHCP server for the local
# network, the authoritative directive should be uncommented.
#authoritative;

# Use this to send dhcp log messages to a different log file (you also
# have to hack syslog.conf to complete the redirection).
log-facility local7;

# No service will be given on this subnet, but declaring it helps the 
# DHCP server to understand the network topology.

# internal subnet for my DHCP Server
subnet 192.168.1.0 netmask 255.255.255.0 {
 range 192.168.1.21 192.168.1.151;
 option domain-name-servers 192.168.1.1;
 option domain-name "pxe.labenv.com";
 option routers 192.168.1.1;
 option broadcast-address 192.168.1.255;
 default-lease-time 600;
 max-lease-time 7200;

 # IP of PXE Server
 next-server 192.168.1.196;
 filename "pxelinux.0";
}
```
***NOTE:*** *Update options as needed*
<br>
Create ```tftp``` config. Create ```/etc/xinetd.d/tftp```.
```
# default: off
# description: The tftp server serves files using the trivial file transfer \
#       protocol.  The tftp protocol is often used to boot diskless \
#       workstations, download configuration files to network-aware printers, \
#       and to start the installation process for some operating systems.
service tftp
{
        socket_type             = dgram
        protocol                = udp
        wait                    = yes
        user                    = root
        server                  = /usr/sbin/in.tftpd
        server_args             = -s /var/lib/tftpboot
        disable                 = no
        per_source              = 11
        cps                     = 100 2
        flags                   = IPv4
}
```
<br>
Copy PXE configs and create required folders
```
cd /usr/share/syslinux/
sudo cp pxelinux.0 menu.c32 memdisk mboot.c32 chain.c32 ldlinux.c32 libutil.c32 /var/lib/tftpboot/

sudo mkdir -p /var/lib/tftpboot/pxelinux.cfg/{centos7,centos8}
sudo mkdir -p /var/lib/tftpboot/networkboot/{centos7,centos8}
sudo mkdir /var/ftp/pub/centos7
sudo mkdir /var/ftp/pub/centos8
```

### **Distribution ISO Updates**
Download CentOS 7 and CentOS 8 ISO. Extract the files and copy them to the shared mounts.
```
sudo mkdir /mnt/test
sudo mount -o loop CentOS-7-x86_64-DVD-2009.iso /mnt/test/
sudo cp -R /mnt/test/* /var/ftp/pub/centos7/
sudo cp /var/ftp/pub/centos7/images/pxeboot/initrd.img /var/lib/tftpboot/networkboot/centos7
sudo cp /var/ftp/pub/centos7/images/pxeboot/vmlinuz /var/lib/tftpboot/networkboot/centos7

sudo umount /mnt/test
sudo mount -o loop CentOS-8.3.2011-x86_64-dvd1.iso /mnt/test/
sudo cp -R /mnt/test/* /var/ftp/pub/centos8/
sudo cp /var/ftp/pub/centos8/images/pxeboot/initrd.img /var/lib/tftpboot/networkboot/centos8
sudo cp /var/ftp/pub/centos8/images/pxeboot/vmlinuz /var/lib/tftpboot/networkboot/centos8
sudo umount /mnt/test
```
<br>
Create default password 
```
openssl passwd -1 SecurePassword

$1$e2wrcGGX$tZPQKPsXVhNmbiGg53MN41
```

### **Create Default Kickstart Configs**
***NOTE:*** *Change 192.168.1.196 to your PXE Boot Address*
<br>
Kickstart Config for CentOS 7. ```/var/ftp/pub/centos7/centos7.cfg```
```
#platform=x86, AMD64, or Intel EM64T
#version=DEVEL
# Firewall configuration
firewall --disabled
# Install OS instead of upgrade
install

# Use FTP installation media
url --url="ftp://192.168.1.196/pub/centos7"

# Root password
rootpw --iscrypted $1$e2wrcGGX$tZPQKPsXVhNmbiGg53MN41

# System authorization information
auth useshadow passalgo=sha512

# Use graphical install
#graphical
# Use Text install
text
firstboot disable

# System keyboard
keyboard us

# System language
lang en_US

# SELinux configuration
selinux disabled

# Installation logging level
logging level=info

# System timezone
timezone America/New_York --isUtc

# Network Information
network --bootproto=dhcp

# System bootloader configuration
bootloader location=mbr

# Partition clearing information
clearpart --all --initlabel
zerombr

# Disk partitioning information
part swap --fstype="swap" --size=2048
part /boot --fstype=ext4 --size=512
part / --fstype="ext4" --size=12408 --grow

%packages --ignoremissing
@base
@core
@development-tools
@print-client
chrony
kexec-tools

%end
%addon com_redhat_kdump --disable --reserve-mb='auto'
%end

# Reboot after installation
reboot
```
<br>
Kickstart Config for CentOS 8. ```/var/ftp/pub/centos8/centos8.cfg```
```
#platform=x86, AMD64, or Intel EM64T
#version=DEVEL

# Partition clearing information
clearpart --all --initlabel

# Use Text install
text

# Accept Eula
eula --agreed

# Network information
network  --bootproto=dhcp --device=link --ipv6=auto --activate
network  --hostname=localhost

# Use FTP installation media
url --url="ftp://192.168.1.196/pub/centos8/BaseOS"

# Root password
rootpw --iscrypted $1$e2wrcGGX$tZPQKPsXVhNmbiGg53MN41

# Disable the Setup Agent on first boot
firstboot --disable

# Do not configure the X Window System
skipx

# System keyboard
keyboard us

# System language
lang en_US.UTF-8

# System timezone
timezone America/New_York --utc

# Disk partitioning information
part swap --fstype="swap" --size=2048
part /boot --fstype=ext4 --size=512
part / --fstype="ext4" --size=12408 --grow

%packages --ignoremissing
@core
@base
@development-tools
@^minimal-environment
@standard 
bc
authconfig
cifs-utils
cryptsetup
dosfstools
expect
firewalld
iotop
-iwl*-firmware
mailx
man
mlocate
net-tools
nfs-utils
parted
rsync
strace
yum-utils
python
%end
%addon com_redhat_kdump --disable --reserve-mb='auto'
%end

# Reboot after installation
reboot
```

### **PXE Boot Menu**
***NOTE:*** *Change 192.168.1.196 to your PXE Boot Address*
<br>
Create the PXE boot menu. Edit/Create ```/var/lib/tftpboot/pxelinux.cfg/default```
```
default menu.c32
prompt 0
timeout 30
MENU TITLE BootLabs.com PXE Menu

LABEL CentOS7_x64
MENU LABEL CentOS 7_X64
KERNEL /networkboot/centos7/vmlinuz
APPEND initrd=/networkboot/centos7/initrd.img inst.repo=ftp://192.168.1.196/pub/centos7 ks=ftp://192.168.1.196/pub/centos7/centos7.cfg

LABEL CentOS8_x64
MENU LABEL CentOS 8_X64
KERNEL /networkboot/centos8/vmlinuz
APPEND initrd=/networkboot/centos8/initrd.img inst.repo=ftp://192.168.1.196/pub/centos8/AppStream ks=ftp://192.168.1.196/pub/centos8/centos8.cfg
``` 

### **Enable Services**
Enable xinetd, dhcp and vsftpd service
```
sudo systemctl daemon-reload

sudo systemctl start xinetd
sudo systemctl enable xinetd
sudo systemctl restart xinetd

sudo systemctl start dhcpd.service
sudo systemctl enable dhcpd.service
sudo systemctl restart dhcpd.service

sudo systemctl start vsftpd
sudo systemctl enable vsftpd
sudo systemctl restart vsftpd
```

### **BootStrap CentOS 7 or 8 Servers**
On the new servers, ensure that network boot is enabled and configured. Start the server. If all goes well,
you should see the pxe boot menu. Select the OS Label of your choice and that should be it.
<br>
*NOTE: For customized installations, configure your kickstart config as needed*
