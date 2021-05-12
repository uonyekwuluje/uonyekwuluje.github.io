---
layout: post
title:  "Ubuntu Hacks"
categories: Sysadmin
---

This is a collection of hacks and commands for solving basic but painful issues in Ubuntu.

### **apt-get/apt stuck at 0 OR stalled install/update**
Sometimes you run into stuck installs in ubuntu. This could be a result of DNS, Network etc. issues.
I have found these to work always. Updating one or both these files with the entries below should work.<br>
* Update `/etc/gai.conf` by uncommenting this line
```
#precedence ::ffff:0:0/96  100
to
precedence ::ffff:0:0/96  100
```
* Update kernel parameters to allow `IPv6` via `/etc/sysctl.conf`
```
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1
```
reload by typing `sudo sysctl -p` or restart the server.
