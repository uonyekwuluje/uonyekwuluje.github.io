---
layout: post
title:  "Find Operation"
categories: Systems Administration
author: Uchechukwu Onyekwuluje
tags: Administration
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
