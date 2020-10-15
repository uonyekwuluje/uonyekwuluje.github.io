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
Our goal is to resize ```/dev/sda2        32G  1.6G   30G   6% /``` for **32Gb** to **100Gb**
