---
layout: post
title:  "Resizing Azure Linux VM OS Disk Partition"
categories: Systems-Administration
---

When creating Virtual Machines in Azure, you will observe that the root partition has a Default of 32GB for linux. This uses the XFS
file system. Orchestration tools like terraform have options of defining this size but it's up to you; the Administrator to resize
accordingly

# Base Default
When a new linux vm is created, this is what the file system looks like. This is for type RAW
```
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        958M     0  958M   0% /dev
tmpfs           968M     0  968M   0% /dev/shm
tmpfs           968M  9.0M  959M   1% /run
tmpfs           968M     0  968M   0% /sys/fs/cgroup
/dev/sda2        32G  1.6G   30G   6% /
/dev/sda1       497M  101M  397M  21% /boot
/dev/sdb1       3.9G  2.1G  1.7G  56% /mnt/resource
tmpfs           194M     0  194M   0% /run/user/1000

```
Our goal is to resize ```/dev/sda2        32G  1.6G   30G   6% /``` for **32Gb** to **200Gb**

# Stop the Virtual Machine
To stop the VM ```privrhelserver-0``` with Resource Group ```DEVLABS-NETWORK-RG```, type command below. *Substitute in your case and ensure you have the subscription defaults set*
```
az vm stop --name privrhelserver-0 --resource-group DEVLABS-NETWORK-RG
```
Verify the VM has stopped
```
az vm list -o table -d | grep privrhelserver-0
```
You should see this
```
privrhelserver-0  DEVLABS-NETWORK-RG  VM stopped                            eastus
```
Deallocate VM Resource
```
az vm deallocate --resource-group DEVLABS-NETWORK-RG --name privrhelserver-0
```


# Update Disk Size
**List Disk**<br>
List existing disks. *NOTE: You will find a list depending on the number of VMs you have*
```
az disk list --resource-group DEVLABS-NETWORK-RG --query '[*].{Name:name,Gb:diskSizeGb,Tier:accountType}' --output table
Name                                                     Gb
-------------------------------------------------------  ----
privrhelserver-0-dsk                                     35
```
**Update Disk Size**<br>
```
az disk update --resource-group DEVLABS-NETWORK-RG --name privrhelserver-0-dsk --size-gb 200

```
**Start VM**<br>
```
az vm start --name privrhelserver-0 --resource-group DEVLABS-NETWORK-RG
```


# Resize Disk Size
SSh into the VM and check the disk size. ```df -h```.
```
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        958M     0  958M   0% /dev
tmpfs           968M     0  968M   0% /dev/shm
tmpfs           968M  9.0M  959M   1% /run
tmpfs           968M     0  968M   0% /sys/fs/cgroup
/dev/sda2        32G  1.6G   30G   6% /
/dev/sda1       497M  101M  397M  21% /boot
/dev/sdb1       3.9G  2.1G  1.7G  56% /mnt/resource
tmpfs           194M     0  194M   0% /run/user/1000
```
or ```sudo lsblk```. You Should see
```
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
fd0      2:0    1    4K  0 disk 
sda      8:0    0  200G  0 disk 
├─sda1   8:1    0  500M  0 part /boot
└─sda2   8:2    0 31.5G  0 part /
sdb      8:16   0    4G  0 disk 
└─sdb1   8:17   0    4G  0 part /mnt/resource
```
The focus will be on ```sda```.<br>
Run fdisk on that partition
```
sudo fdisk /dev/sda
```
You should have this
```

The device presents a logical sector size that is smaller than
the physical sector size. Aligning to a physical sector (or optimal
I/O) size boundary is recommended, or performance may be impacted.
Welcome to fdisk (util-linux 2.23.2).

Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

```
Type ```p``` and you should see
```
Command (m for help): p

Disk /dev/sda: 214.7 GB, 214748364800 bytes, 419430400 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
Disk label type: dos
Disk identifier: 0x0009d9ef

   Device Boot      Start         End      Blocks   Id  System
/dev/sda1   *        2048     1026047      512000   83  Linux
/dev/sda2         1026048    67108863    33041408   83  Linux
```
Type ```d``` then ```2``` and you should see
```
Command (m for help): d
Partition number (1,2, default 2): 2
Partition 2 is deleted
```
Type ```n``` then ```p``` and defaults
```
Command (m for help): n
Partition type:
   p   primary (1 primary, 0 extended, 3 free)
   e   extended
Select (default p): p
Partition number (2-4, default 2): 2
First sector (1026048-419430399, default 1026048): 
Using default value 1026048
Last sector, +sectors or +size{K,M,G} (1026048-419430399, default 419430399): 
Using default value 419430399
Partition 2 of type Linux and of size 199.5 GiB is set
```
Type ```w``` and then reboot
```
sudo reboot
```


# Resize the VMS
Log into the VM and resize
```
sudo xfs_growfs /
```
You should see
```
meta-data=/dev/sda2              isize=512    agcount=4, agsize=2065088 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0 spinodes=0
data     =                       bsize=4096   blocks=8260352, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal               bsize=4096   blocks=4033, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 8260352 to 52300544
```
**Verify New Size**<br>
Run ```df -h``` and you should see
```
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        958M     0  958M   0% /dev
tmpfs           968M     0  968M   0% /dev/shm
tmpfs           968M  9.0M  959M   1% /run
tmpfs           968M     0  968M   0% /sys/fs/cgroup
/dev/sda2       200G  1.6G  198G   1% /
/dev/sda1       497M  101M  397M  21% /boot
/dev/sdb1       3.9G  2.1G  1.7G  56% /mnt/resource
tmpfs           194M     0  194M   0% /run/user/1000
```
or ```lsblk``` and you should see
```
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
fd0      2:0    1     4K  0 disk 
sda      8:0    0   200G  0 disk 
├─sda1   8:1    0   500M  0 part /boot
└─sda2   8:2    0 199.5G  0 part /
sdb      8:16   0     4G  0 disk 
└─sdb1   8:17   0     4G  0 part /mnt/resource
```
