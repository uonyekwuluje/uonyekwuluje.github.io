---
layout: post
title:  "Find Operation"
categories: Systems Administration
author: Uchechukwu Onyekwuluje
---

As a Linux Systems Administrator or a regular linux user, you usually encounter situations that call for searching
the file system to locate files or logs greater than, less than or within a given range. Log management is one of
the many responsibilities here. This is a collection of single one liners for specific find tasks

### File Size Units
These are units of file sizes
```
G => for gibibytes
M => for megabytes
K => for kibibytes
b => for bytes
```
General syntax
```
find <directory/path> -type f -size +N<Unit Type>
```

## Examples and use cases
Find files greater than `3G`
```
find ~/ -type f -size +3G
```
Find files greater than `3G`, `300M` and display size summary
```
find ~/dataDrive/ -type f -size +3G -exec ls -lh {} \; | awk '{ print $9 "|| Size : " $5 }'
find ~/dataDrive/ -type f -size +300M -exec ls -lh {} \; | awk '{ print $9 "|| Size : " $5 }'
```
Find files less than `3G`. Change size as needed, `3M`, `450M` etc.
```
find ~/ -type f -size -3G
```
Find files between `1G` and `3G` in size
```
find ~/dataDrive/iso_files/ -type f -size +1G -size -3G
```
Find files between `1G` and `3G` in size and list them
```
find ~/dataDrive/iso_files/ -type f -size +1G -size -3G -exec ls -lh {} +
```

# Reference Blogs
* [Ostechnix](https://ostechnix.com/find-files-bigger-smaller-x-size-linux/)
* [This Pointer](https://thispointer.com/linux-find-files-larger-than-given-size/)
