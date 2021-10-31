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
sudo apt install lshw 
```

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
