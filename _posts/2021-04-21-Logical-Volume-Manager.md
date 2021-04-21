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
If we go the route of `sudo vgcreate devpoc_vg /dev/sdb1` we can do this
```
sudo vgcreate devpoc_vg /dev/sdb1
sudo vgextend devpoc_vg /dev/sdc1
```
in this case, we create and extend


### **Create Logical Volumes**
With our volume group in place, we can now create our logical volumes. Lets create three logical volumes
with 200MB,400MB and 2GB
```
sudo lvcreate -L 200 -n store1 prodpoc_vg
sudo lvcreate -L 400 -n store2 prodpoc_vg
sudo lvcreate -L 2000 -n store3 prodpoc_vg
```
You should see this respectively
```
Logical volume "store1" created.
Logical volume "store2" created.
Logical volume "store3" created.
```
**To list your logical volumes**
```
sudo lvdisplay
```
we should see this
```
  --- Logical volume ---
  LV Path                /dev/prodpoc_vg/store1
  LV Name                store1
  VG Name                prodpoc_vg
  LV UUID                Nqf88X-FLqL-XVv6-l4BN-tst5-T2cz-1hEVrc
  LV Write Access        read/write
  LV Creation host, time test-svr, 2021-04-21 07:56:20 +0000
  LV Status              available
  # open                 0
  LV Size                200.00 MiB
  Current LE             50
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:0
   
  --- Logical volume ---
  LV Path                /dev/prodpoc_vg/store2
  LV Name                store2
  VG Name                prodpoc_vg
  LV UUID                gfGOI5-FV7t-84gb-Nl3P-7uWo-hyCp-1E3J4J
  LV Write Access        read/write
  LV Creation host, time test-svr, 2021-04-21 07:56:29 +0000
  LV Status              available
  # open                 0
  LV Size                400.00 MiB
  Current LE             100
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:1
   
  --- Logical volume ---
  LV Path                /dev/prodpoc_vg/store3
  LV Name                store3
  VG Name                prodpoc_vg
  LV UUID                4vXGSa-egWg-zwhF-fCSc-vtZE-HOTb-ywEiTI
  LV Write Access        read/write
  LV Creation host, time test-svr, 2021-04-21 07:56:36 +0000
  LV Status              available
  # open                 0
  LV Size                1.95 GiB
  Current LE             500
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:2
```


### **Create Filesystem**
With our logical volumes in place, we can now create our filesystem
```
sudo mkfs.ext4 /dev/prodpoc_vg/store1
sudo mkfs.ext4 /dev/prodpoc_vg/store2
sudo mkfs.ext4 /dev/prodpoc_vg/store3
```
Create folders to mount the filesystem
```
mkdir ~/{st1,st2,st3}
```
Mount filesystem `/etc/fstab`
```
/dev/prodpoc_vg/store1   /home/ubuntu/st1   ext4  defaults 0 0
/dev/prodpoc_vg/store2   /home/ubuntu/st2   ext4  defaults 0 0
/dev/prodpoc_vg/store3   /home/ubuntu/st3   ext4  defaults 0 0
```
`sudo mount -a`. Type `df -a` and you should see this
```
Filesystem                     Size  Used Avail Use% Mounted on
udev                           966M     0  966M   0% /dev
tmpfs                          200M  732K  199M   1% /run
/dev/sda2                       13G  4.0G  8.2G  33% /
tmpfs                          997M     0  997M   0% /dev/shm
tmpfs                          5.0M     0  5.0M   0% /run/lock
tmpfs                          997M     0  997M   0% /sys/fs/cgroup
tmpfs                          200M     0  200M   0% /run/user/1000
/dev/mapper/prodpoc_vg-store1  190M  1.6M  175M   1% /home/ubuntu/st1
/dev/mapper/prodpoc_vg-store2  380M  2.3M  354M   1% /home/ubuntu/st2
/dev/mapper/prodpoc_vg-store3  1.9G  5.9M  1.8G   1% /home/ubuntu/st3
```


### **Extend Logical Volume**
With our filesystem in place, lets extend one of our logical volumes. We will add `3GB` to `/dev/prodpoc_vg/store2`
```
sudo lvextend -L +3000 /dev/prodpoc_vg/store2
```
you should see something like this
```
  Size of logical volume prodpoc_vg/store2 changed from 400.00 MiB (100 extents) to 3.32 GiB (850 extents).
  Logical volume prodpoc_vg/store2 successfully resized.
```
Now we resize it
```
sudo resize2fs /dev/prodpoc_vg/store2
```
you should see something like this
```
resize2fs 1.44.1 (24-Mar-2018)
Filesystem at /dev/prodpoc_vg/store2 is mounted on /home/ubuntu/st2; on-line resizing required
old_desc_blocks = 4, new_desc_blocks = 27
The filesystem on /dev/prodpoc_vg/store2 is now 3481600 (1k) blocks long.
```
Testing out with `df -h` we can see our changes
```
Filesystem                     Size  Used Avail Use% Mounted on
udev                           966M     0  966M   0% /dev
tmpfs                          200M  732K  199M   1% /run
/dev/sda2                       13G  4.0G  8.2G  33% /
tmpfs                          997M     0  997M   0% /dev/shm
tmpfs                          5.0M     0  5.0M   0% /run/lock
tmpfs                          997M     0  997M   0% /sys/fs/cgroup
tmpfs                          200M     0  200M   0% /run/user/1000
/dev/mapper/prodpoc_vg-store1  190M  1.6M  175M   1% /home/ubuntu/st1
/dev/mapper/prodpoc_vg-store2  3.3G  3.0M  3.1G   1% /home/ubuntu/st2
/dev/mapper/prodpoc_vg-store3  1.9G  5.9M  1.8G   1% /home/ubuntu/st3
```

### **Remove Logical Volume**
We can also remove the logical volumes if we no longer need them. In this case `/dev/prodpoc_vg/store1`
```
sudo lvremove /dev/prodpoc_vg/store1
```
answer yes
```
Do you really want to remove and DISCARD active logical volume prodpoc_vg/store1? [y/n]: y
  Logical volume "store1" successfully removed
```
*Remember to unmount first*

### **Command Summary Reference***
List of PV Commands
* **pvchange:** — Change attributes of a Physical Volume.
* **pvck:** — Check Physical Volume metadata.
* **pvcreate:** — Initialize a disk or partition for use by LVM.
* **pvdisplay:** — Display attributes of a Physical Volume.
* **pvmove:** — Move Physical Extents.
* **pvremove:** — Remove a Physical Volume.
* **pvresize:** — Resize a disk or partition in use by LVM2.
* **pvs:** — Report information about Physical Volumes.
* **pvscan:** — Scan all disks for Physical Volumes. 


List of VG commands
* **vgcfgbackup:** — Backup Volume Group descriptor area.
* **vgcfgrestore:** — Restore Volume Group descriptor area.
* **vgchange:** — Change attributes of a Volume Group.
* **vgck:** — Check Volume Group metadata.
* **vgconvert:** — Convert Volume Group metadata format.
* **vgcreate:** — Create a Volume Group.
* **vgdisplay:** — Display attributes of Volume Groups.
* **vgexport:** — Make volume Groups unknown to the system.
* **vgextend:** — Add Physical Volumes to a Volume Group.
* **vgimport:** — Make exported Volume Groups known to the system.
* **vgimportclone:** — Import and rename duplicated Volume Group (e.g. a hardware snapshot).
* **vgmerge:** — Merge two Volume Groups.
* **vgmknodes:** — Recreate Volume Group directory and Logical Volume special files
* **vgreduce:** — Reduce a Volume Group by removing one or more Physical Volumes.
* **vgremove:** — Remove a Volume Group.
* **vgrename:** — Rename a Volume Group.
* **vgs:** — Report information about Volume Groups.
* **vgscan:** — Scan all disks for Volume Groups and rebuild caches.
* **vgsplit:** — Split a Volume Group into two, moving any logical volumes from one Volume Group to another by moving entire Physical Volumes. 


List of LV commands
* **lvchange:** — Change attributes of a Logical Volume.
* **lvconvert:** — Convert a Logical Volume from linear to mirror or snapshot.
* **lvcreate:** — Create a Logical Volume in an existing Volume Group.
* **lvdisplay:** — Display the attributes of a Logical Volume.
* **lvextend:** — Extend the size of a Logical Volume.
* **lvreduce:** — Reduce the size of a Logical Volume.
* **lvremove:** — Remove a Logical Volume.
* **lvrename:** — Rename a Logical Volume.
* **lvresize:** — Resize a Logical Volume.
* **lvs:** — Report information about Logical Volumes.
* **lvscan:** — Scan (all disks) for Logical Volumes. 



### **Reference Links**
* [Resize LVM](https://carleton.ca/scs/2019/extend-lvm-disk-space/)
* [LVM Tutorial](https://linuxconfig.org/linux-lvm-logical-volume-manager)
* [LVM Examples](http://landoflinux.com/linux_lvm_command_examples.html)
* [LVM Cheat Sheet](http://www.datadisk.co.uk/html_docs/redhat/rh_lvm.htm)
* [Debian LVM Reference](https://wiki.debian.org/LVM)
