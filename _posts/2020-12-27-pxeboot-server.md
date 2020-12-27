---
layout: post
title:  "PXE Boot Server Setup"
categories: Automation, PXE
---

A PXE (Preboot eXecution Environment) Server allows unattended installation of Operating Systems over the Network. This helps when there is a need to setup lots of servers without physically attaching a CD or installation media on the servers. In this post we will setup a PXE Server on CentOS 8 Stream and then drive multiple installations using it

### **Requirements**
In order to setup the PXE Boot Server, you need the following:
* CentOS 8 Stream
* Admin Privilleges 
* ISO for your operating system. *In this post, we will be using CentOS 7 and CentOS8*

### **Setup**
Update and install the following packages on the PXE Server
```
sudo yum update -y
sudo yum group install -y 'Development Tools'
sudo yum install -y libxml2 libxml2-devel libxslt libxslt-devel wget gcc \
libffi-devel openssl-devel make openssl-devel bzip2-devel
sudo yum install -y dhcp-server tftp tftp-server syslinux vsftpd xinetd
```
***NOTE:*** *Adjust your firewall rules and selinux as needed*
<br>
**Configure DHCP**<br>
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
 next-server 192.168.1.7;
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
