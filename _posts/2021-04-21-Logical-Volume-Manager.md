---
layout: post
title:  "Logical Volume Manager"
categories: LVM
---

Logical Volume Manager (LVM) is a storage management system/technology used on Linux to manage hard drives and other storage devices. 
In this guide, we will briefly look at how LVM works and the basic commands needed to get you up and running quickly.

### **Package Installation**
For RPM based distributions:
```
sudo dnf install lvm2
```
For DEB based distributions
```
sudo apt install lvm2
```

### **LVM Storage Terms**

**Physical Volumes:** 
* Prefix `pv`
* Description:  Physical disks. /dev/sda,/dev/hda etc

**Volume Group:**
* Prefix `vg`
* Description:  Combination of physical volumes
   
**Logical Volumes:**
* Prefix `lv`   
* Description:  Volume group broken down into slices, volume groups, partitions
