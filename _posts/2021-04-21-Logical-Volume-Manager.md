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

### **Create Physical Volumes**
**Create physical volumes**
```
sudo pvcreate /dev/sdb1
sudo pvcreate /dev/sdc1
sudo pvcreate /dev/sdd1
```
You should have these respectively
```
Physical volume "/dev/sdb1" successfully created.
Physical volume "/dev/sdc1" successfully created.
Physical volume "/dev/sdd1" successfully created.
```

**List Physical Volumes**
```
sudo pvdisplay
```
You should see this
```
  "/dev/sdb1" is a new physical volume of "<13.00 GiB"
  --- NEW Physical volume ---
  PV Name               /dev/sdb1
  VG Name               
  PV Size               <13.00 GiB
  Allocatable           NO
  PE Size               0   
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               nBxpTQ-ZCDY-ScZy-oZj2-0WC8-QbbF-jJ4965
   
  "/dev/sdd1" is a new physical volume of "<13.00 GiB"
  --- NEW Physical volume ---
  PV Name               /dev/sdd1
  VG Name               
  PV Size               <13.00 GiB
  Allocatable           NO
  PE Size               0   
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               KV0CFJ-EwcP-yjKK-3ejR-y69y-6Icc-rY53th
   
  "/dev/sdc1" is a new physical volume of "<13.00 GiB"
  --- NEW Physical volume ---
  PV Name               /dev/sdc1
  VG Name               
  PV Size               <13.00 GiB
  Allocatable           NO
  PE Size               0   
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               e4WEPR-Rtxk-yECB-Z4je-cDt6-J3w9-u0JQ9I
```

### **Create Volume Group**
With our physical volumes in place, we can now create our volume group
```
sudo vgcreate devpoc_vg /dev/sdb1
OR
sudo vgcreate prodpoc_vg /dev/sdb1 /dev/sdc1 /dev/sdd1
```
*Not Both*
using `prodpoc_vg` in this case, we have
```
  Volume group "prodpoc_vg" successfully created
```
List volume group
```
sudo vgdisplay
```
You should seee this
```
  --- Volume group ---
  VG Name               prodpoc_vg
  System ID             
  Format                lvm2
  Metadata Areas        3
  Metadata Sequence No  1
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                0
  Open LV               0
  Max PV                0
  Cur PV                3
  Act PV                3
  VG Size               <38.99 GiB
  PE Size               4.00 MiB
  Total PE              9981
  Alloc PE / Size       0 / 0   
  Free  PE / Size       9981 / <38.99 GiB
  VG UUID               CaLrye-GWnD-Kmo1-2MZb-rdEV-hJtP-fso1LD
```
If we go thr route of `sudo vgcreate devpoc_vg /dev/sdb1` we can do this
```
sudo vgcreate devpoc_vg /dev/sdb1
sudo vgextend devpoc_vg /dev/sdc1
```
