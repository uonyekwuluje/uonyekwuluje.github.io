---
layout: post
title:  "Linux Hardware"
categories: linux
---
Some aspects of systems administration in linux involves identifying and working with hardware.
Knowing your motherboard, CPU, RAM, system specs is nice. Below are some tools for this. This is 
specifically geared at ubuntu systems.

## Install Packages
Install package
```
sudo apt install lshw hwinfo pydf
```

**pydf**<br>
An improved version of `df`. Type `pydf`. You should see
```
Filesystem    Size  Used Avail Use%                              Mounted on
/dev/pve/root  58G 3559M   52G  6.0 [##........................] /         
/dev/sda2     511M  328k  511M  0.1 [..........................] /boot/efi 
/dev/fuse     128M   16k  128M  0.0 [..........................] /etc/pve  
/dev/sdb1     916G 3516M  866G  0.4 [..........................] /mnt/data 
```

**hwinfo**<br>
HWinfo is a general propbing Hardware probing tool. Type `hwinfo --short`. Should have
```
cpu:                                                            
                       Intel(R) Core(TM) i5-6500 CPU @ 3.20GHz, 3200 MHz
                       Intel(R) Core(TM) i5-6500 CPU @ 3.20GHz, 900 MHz
                       Intel(R) Core(TM) i5-6500 CPU @ 3.20GHz, 3200 MHz
                       Intel(R) Core(TM) i5-6500 CPU @ 3.20GHz, 3200 MHz
graphics card:
                       Intel HD Graphics 530
sound:
                       Intel 100 Series/C230 Series Chipset Family HD Audio Controller
storage:
                       Floppy disk controller
                       Intel Q170/Q150/B150/H170/H110/Z170/CM236 Chipset SATA Controller [AHCI Mode]
network:
  eno1                 Intel Ethernet Connection (2) I219-LM
network interface:
  vmbr0                Ethernet network interface
  eno1                 Ethernet network interface
  lo                   Loopback network interface
disk:
  /dev/sdb             TOSHIBA DT01ACA1
  /dev/sda             KingFast
partition:
  /dev/sdb1            Partition
  /dev/sda1            Partition
  /dev/sda2            Partition
  /dev/sda3            Partition
cdrom:
  /dev/sr0             hp HLDS DVDRW  GUD1N
usb controller:
                       Intel 100 Series/C230 Series Chipset Family USB 3.0 xHCI Controller
bios:
                       BIOS
bridge:
                       Intel 100 Series/C230 Series Chipset Family PCI Express Root Port #6
                       Intel Q170 Chipset LPC/eSPI Controller
                       Intel Xeon E3-1200 v5/E3-1500 v5/6th Gen Core Processor Host Bridge/DRAM Registers
                       Intel 100 Series/C230 Series Chipset Family PCI Express Root Port #9
                       Intel 100 Series/C230 Series Chipset Family PCI Express Root Port #7
hub:
                       Linux Foundation 2.0 root hub
                       Linux Foundation 3.0 root hub
memory:
                       Main Memory
unknown:
                       FPU
                       DMA controller
                       PIC
                       Keyboard controller
                       PS/2 Controller
                       Intel 100 Series/C230 Series Chipset Family Power Management Controller
                       Intel 100 Series/C230 Series Chipset Family Thermal Subsystem
                       Intel 100 Series/C230 Series Chipset Family SMBus
  /dev/ttyS0           16550A
```
For more infor, see help for this command


**lshw**<br>
To list hardware, type `lshw -short`. Should have
```
H/W path           Device      Class          Description
=========================================================
                               system         HP EliteDesk 800 G2 SFF (L1G76AV)
/0                             bus            8054
/0/0                           memory         128KiB L1 cache
/0/1                           memory         128KiB L1 cache
/0/2                           memory         1MiB L2 cache
/0/3                           memory         6MiB L3 cache
/0/4                           processor      Intel(R) Core(TM) i5-6500 CPU @ 3.20GHz
/0/5                           memory         40GiB System Memory
/0/5/0                         memory         8GiB DIMM DDR4 Synchronous Unbuffered (Unregistered) 2133 MHz (0.5 ns)
/0/5/1                         memory         16GiB DIMM DDR4 Synchronous Unbuffered (Unregistered) 2133 MHz (0.5 ns)
/0/5/2                         memory         [empty]
/0/5/3                         memory         16GiB DIMM DDR4 Synchronous Unbuffered (Unregistered) 2133 MHz (0.5 ns)
/0/b                           memory         64KiB BIOS
/0/100                         bridge         Xeon E3-1200 v5/E3-1500 v5/6th Gen Core Processor Host Bridge/DRAM Regis
/0/100/2                       display        HD Graphics 530
/0/100/14                      bus            100 Series/C230 Series Chipset Family USB 3.0 xHCI Controller
/0/100/14/0        usb1        bus            xHCI Host Controller
/0/100/14/1        usb2        bus            xHCI Host Controller
/0/100/14.2                    generic        100 Series/C230 Series Chipset Family Thermal Subsystem
/0/100/17          scsi0       storage        Q170/Q150/B150/H170/H110/Z170/CM236 Chipset SATA Controller [AHCI Mode]
/0/100/17/0        /dev/sda    disk           256GB KingFast
/0/100/17/0/1      /dev/sda1   volume         1006KiB BIOS Boot partition
/0/100/17/0/2                  volume         511MiB Windows FAT volume
/0/100/17/0/3      /dev/sda3   volume         237GiB LVM Physical Volume
/0/100/17/1        /dev/cdrom  disk           DVDRW  GUD1N
/0/100/17/0.0.0    /dev/sdb    disk           1TB TOSHIBA DT01ACA1
/0/100/17/0.0.0/1  /dev/sdb1   volume         931GiB EXT4 volume
/0/100/1c                      bridge         100 Series/C230 Series Chipset Family PCI Express Root Port #6
/0/100/1c.6                    bridge         100 Series/C230 Series Chipset Family PCI Express Root Port #7
/0/100/1d                      bridge         100 Series/C230 Series Chipset Family PCI Express Root Port #9
/0/100/1f                      bridge         Q170 Chipset LPC/eSPI Controller
/0/100/1f.2                    memory         Memory controller
/0/100/1f.3                    multimedia     100 Series/C230 Series Chipset Family HD Audio Controller
/0/100/1f.4                    bus            100 Series/C230 Series Chipset Family SMBus
/0/100/1f.6        eno1        network        Ethernet Connection (2) I219-LM
/0/6                           system         PnP device PNP0c02
/0/7                           system         PnP device PNP0c02
/0/8                           system         PnP device PNP0c02
/0/9                           system         PnP device PNP0c02
/0/a                           system         PnP device PNP0b00
/0/c                           generic        PnP device INT3f0d
/0/d                           generic        PnP device HPQ8001
/0/e                           input          PnP device PNP0f13
/0/f                           communication  PnP device PNP0501
/0/10                          system         PnP device PNP0c02
/0/11                          system         PnP device PNP0c02
/1                             power          High Efficiency
/2                 vmbr0       network        Ethernet interface
```
For more details, try the commanmd help


**lscpu**<br>
To get CPU details, type `lscpu`. Should have 
```
Architecture:                    x86_64
CPU op-mode(s):                  32-bit, 64-bit
Byte Order:                      Little Endian
Address sizes:                   36 bits physical, 48 bits virtual
CPU(s):                          4
On-line CPU(s) list:             0-3
Thread(s) per core:              2
Core(s) per socket:              2
Socket(s):                       1
NUMA node(s):                    1
Vendor ID:                       GenuineIntel
CPU family:                      6
Model:                           42
Model name:                      Intel(R) Core(TM) i3-2120 CPU @ 3.30GHz
Stepping:                        7
CPU MHz:                         3220.394
CPU max MHz:                     3300.0000
CPU min MHz:                     1600.0000
BogoMIPS:                        6585.10
Virtualization:                  VT-x
L1d cache:                       64 KiB
L1i cache:                       64 KiB
L2 cache:                        512 KiB
L3 cache:                        3 MiB
NUMA node0 CPU(s):               0-3
Vulnerability Itlb multihit:     KVM: Mitigation: VMX disabled
```

**lsmem**<br>
To get Memory details, type `lsmem`. Should have
```
RANGE                                  SIZE  STATE REMOVABLE  BLOCK
0x0000000000000000-0x00000000dfffffff  3.5G online       yes   0-27
0x0000000100000000-0x000000039fffffff 10.5G online       yes 32-115

Memory block size:       128M
Total online memory:      14G
Total offline memory:      0B
```

**Other Commands**<br>
A list of other commands are
* `demidecode`
* `free -m`
* `cat /proc/version`
* `cat /proc/cpuinfo`
* `cat /proc/meminfo`
