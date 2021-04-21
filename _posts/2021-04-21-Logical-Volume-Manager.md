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
* Description:  Combination of physical volumes into storage pools
   
**Logical Volumes:**
* Prefix `lv`   
* Description:  Volume group broken down into slices, volume groups, partitions

**Extents:**
* Description:  Each volume within a volume group is segmented into small, fixed-size chunks called extents. The size of the extents is 
  determined by the volume group (all volumes within the group conform to the same extent size). 
  The extents on a physical volume are called physical extents, while the extents of a logical volume are called logical extents. A logical 
  volume is simply a mapping that LVM maintains between logical and physical extents. Because of this relationship, the extent size represents 
  the smallest amount of space that can be allocated by LVM.


### **Scan System**
Scan the system for block devices
```
sudo lvmdiskscan
```
You should see this depending on your drives
```
  /dev/sda2 [     <13.00 GiB] 
  /dev/sdb  [      13.00 GiB] 
  /dev/sdc  [      13.00 GiB] 
  /dev/sdd  [      13.00 GiB] 
  3 disks
  1 partition
  0 LVM physical volume whole disks
  0 LVM physical volumes
```

### **Create Partitions**
List Block Devices
```
lsblk
```
You should see this
```
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0   13G  0 disk 
├─sda1   8:1    0    1M  0 part 
└─sda2   8:2    0   13G  0 part /
sdb      8:16   0   13G  0 disk 
sdc      8:32   0   13G  0 disk 
sdd      8:48   0   13G  0 disk 
sr0     11:0    1 1024M  0 rom  
```
Run `fdisk` on `/dev/sdb, /dev/sdc, /dev/sdd` and accept defaults. You should see this when completed
```
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0   13G  0 disk 
├─sda1   8:1    0    1M  0 part 
└─sda2   8:2    0   13G  0 part /
sdb      8:16   0   13G  0 disk 
└─sdb1   8:17   0   13G  0 part 
sdc      8:32   0   13G  0 disk 
└─sdc1   8:33   0   13G  0 part 
sdd      8:48   0   13G  0 disk 
└─sdd1   8:49   0   13G  0 part 
sr0     11:0    1 1024M  0 rom  
```
