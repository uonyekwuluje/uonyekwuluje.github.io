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
*Note: update `dhcp-range` based on your network*


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

### **Operating System Boot Configs**
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
wget http://mirrors.mit.edu/centos/8.3.2011/isos/x86_64/CentOS-8.3.2011-x86_64-dvd1.iso
sudo mount -o loop CentOS-8.3.2011-x86_64-dvd1.iso /mnt/
sudo cp -R /mnt/* /var/www/html/centos8/
sudo cp /mnt/images/pxeboot/{initrd.img,vmlinuz} /var/lib/tftpboot/centos8/
```

### **Test Kickstart and Preseed Configs**
Ubutu 20 Preseed Configs
```
sudo bash -c 'cat <<EOF> /var/www/html/ubuntu20/preseed.cfg
### Automatic Installation
d-i auto-install/enable boolean true
d-i debconf/priority select critical

### Localization
d-i debian-installer/locale string en_US.UTF-8
d-i localechooser/supported-locales multiselect en_US.UTF-8
# Keyboard
d-i console-setup/ask_detect boolean false
d-i keyboard-configuration/xkb-keymap select us
d-i keyboard-configuration/layoutcode string us
d-i keyboard-configuration/modelcode string pc105

### Network config
d-i netcfg/choose_interface select auto
d-i netcfg/get_hostname string build
d-i netcfg/get_domain string home
d-i netcfg/hostname string build
d-i netcfg/dhcp_timeout string 15
d-i netcfg/dhcpv6_timeout string 15 
d-i hw-detect/load_firmware boolean true

### Mirror settings
d-i mirror/country string manual
d-i mirror/http/hostname string archive.ubuntu.com
d-i mirror/http/directory string /ubuntu
d-i mirror/http/proxy string

### Account setup
d-i passwd/user-fullname string Ubuntu User
d-i passwd/username string ubuntu
d-i passwd/user-password password ubuntu
d-i passwd/user-password-again password ubuntu
d-i user-setup/allow-password-weak boolean true

### Time zone setup
d-i time/zone string America/New_York
d-i clock-setup/utc boolean false
d-i clock-setup/ntp boolean true

# Finishing up the installation
d-i finish-install/reboot_in_progress note

# Boot loader installation
d-i grub-installer/only_debian boolean true
d-i grub-installer/with_other_os boolean true

### Partitioning
d-i partman-auto/disk string
d-i partman-auto/method string regular
d-i partman-lvm/device_remove_lvm boolean true
d-i partman-md/device_remove_md boolean true
d-i partman-lvm/confirm boolean true
d-i partman-basicfilesystems/no_swap boolean false
d-i partman-auto/expert_recipe string                         \
      boot-root ::                                            \
              500 10000 1000000000 ext4                       \
                      method{ format } format{ }              \
                      use_filesystem{ } filesystem{ ext4 }    \
                      mountpoint{ / }                         \
              .                                               \
d-i partman-partitioning/confirm_write_new_label boolean true
d-i partman/choose_partition select finish
d-i partman/confirm boolean true
d-i partman/confirm_nooverwrite boolean true

### Apt setup
# You can choose to install restricted and universe software, or to install
# software from the backports repository.
d-i apt-setup/restricted boolean true
d-i apt-setup/universe boolean true
d-i apt-setup/backports boolean true
d-i apt-setup/services-select multiselect security
d-i apt-setup/security_host string security.ubuntu.com
d-i apt-setup/security_path string /ubuntu

### Package selection
d-i tasksel/first multiselect openssh-server
d-i pkgsel/include string open-vm-tools curl jq net-tools openssh-server vim build-essential rsync curl software-properties-common
d-i pkgsel/language-packs multiselect en
d-i pkgsel/update-policy select none

# Verbose output and no boot splash screen.
d-i	debian-installer/quiet	boolean false
d-i	debian-installer/splash	boolean false

# Avoid that last message about the install being complete.
# This will just finish and reboot
d-i finish-install/reboot_in_progress note
#d-i debian-installer/exit/poweroff boolean true
EOF'
```
***NOTE: Adjust your configs as needed***

CentOS 8 kickstart Configs
```
sudo bash -c 'cat <<EOF> /var/www/html/centos8/centos8.cfg 
# Partition clearing information
clearpart --all --initlabel

# Use text install
text

# Accept Eula
eula --agreed

# Do not configure the X Window System
skipx

# System timezone
timezone America/New_York --isUtc

# Network Installation
url --url="http://mirror.centos.org/centos/8-stream/BaseOS/x86_64/os/"

# Root password Pxe@123# need to escape \$
rootpw --iscrypted \$1\$e2wrcGGX\$tZPQKPsXVhNmbiGg53MN41

# System authorization information
auth useshadow passalgo=sha512

# Disable the Setup Agent on first boot
firstboot --disable

# System keyboard
keyboard --vckeymap=us --xlayouts='us'

# System language
lang en_US.UTF-8

# Network information
network --onboot=yes --bootproto=dhcp --ipv6=auto --activate --hostname=build

# System bootloader configuration
zerombr

# partitioning
part /boot --fstype="ext4" --size=1024
part pv.01 --fstype="lvmpv" --grow
volgroup cl pv.01
logvol / --fstype="xfs" --name=root --vgname=cl --size=10000
logvol /tmp --fstype="xfs" --name=tmp --vgname=cl --size=3000
logvol /var/log --fstype="xfs" --name=log --vgname=cl --size=5000

# Package Install
%packages
@^minimal-environment
@standard
%end

# Reboot after installation
reboot

%anaconda
pwpolicy root --minlen=6 --minquality=1 --notstrict --nochanges --notempty
pwpolicy user --minlen=6 --minquality=1 --notstrict --nochanges --emptyok
pwpolicy luks --minlen=6 --minquality=1 --notstrict --nochanges --notempty
%end
EOF'
```
***NOTE: Adjust your configs as needed***


### **PXE Boot Menu***
Create PXE Boot Menu
```
sudo bash -c 'cat <<EOF> /var/lib/tftpboot/pxelinux.cfg/default 
DEFAULT vesamenu.c32
PROMPT 1
TIMEOUT 15
MENU TITLE PXE Menu
MENU WIDTH 80
MENU MARGIN 10
MENU PASSWORDMARGIN 3
MENU ROWS 10
MENU TABMSGROW 15
MENU CMDLINEROW 15
MENU ENDROW 24
MENU PASSWORDROW 11
MENU TIMEOUTROW 16
menu color title 1;34;49 #eea0a0ff #cc333355 std
menu color sel 7;37;40 #ff000000 #bb9999aa all
menu color border 30;44 #ffffffff #00000000 std
menu color pwdheader 31;47 #eeff1010 #20ffffff std
menu color hotkey 35;40 #90ffff00 #00000000 std
menu color hotsel 35;40 #90000000 #bb9999aa all
menu color timeout_msg 35;40 #90ffffff #00000000 none
menu color timeout 31;47 #eeff1010 #00000000 none

LABEL Ubuntu_20.04_LTS
  MENU LABEL Ubuntu 20.04 LTS
  KERNEL ubuntu20/linux
  APPEND hostname=unassigned locale=en_US.UTF-8 keyboard-configuration/layoutcode=us initrd=ubuntu20/initrd.gz url=http://192.168.1.105/ubuntu20/preseed.cfg splash toram ---

LABEL CentOS 8
  MENU LABEL CentOS 8
  KERNEL centos8/vmlinuz
  APPEND initrd=centos8/initrd.img inst.repo=http://192.168.1.105/centos8 inst.ks=http://192.168.1.105/centos8/centos8.cfg
EOF'
```
*NOTE: Remember to substitute the IP address of your server above*

### **References**
* [Ubuntu 18 Installer](http://archive.ubuntu.com/ubuntu/dists/bionic-updates/main/installer-amd64/current/images/netboot/netboot.tar.gz)
* [Ubuntu 20 Installer](http://archive.ubuntu.com/ubuntu/dists/focal-updates/main/installer-amd64/current/legacy-images/netboot/netboot.tar.gz)
* [CentOS 8 ISO](http://mirrors.mit.edu/centos/8.3.2011/isos/x86_64/CentOS-8.3.2011-x86_64-dvd1.iso)
* [CentOS 8 Missing Config](http://mirror.centos.org/centos/8-stream/BaseOS/x86_64/os/)
