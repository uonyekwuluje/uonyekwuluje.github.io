---
layout: post
title:  "Find Host IP Addreess"
categories: Systems-Administration
---

When performing various automation tasks, there is usually a need to work with IP Addresses. 
To find the private IP address on a linux server, the following commands work.

## Private IP 
```
hostname -I | awk '{print $1}'
ip route get 1.2.3.4 | awk '{print $7}'
ip addr show eno1 | grep "inet\b" | awk '{print $2}' | cut -d/ -f1
```

## Parsing and processing logs with cut
This can be used with any log file. In this case, I am using cut to filter apache2 logs
```
# Get only IPS
tail -n 1000 /var/log/apache2/access.log | cut -f 1 -d ' ' | sort | uniq -c | sort -nr | more

# Get IPs and others
cat /var/log/apache2/access.log | cut -f 1,3,4,6 -d ' ' | sort | uniq -c | sort -nr 
```
