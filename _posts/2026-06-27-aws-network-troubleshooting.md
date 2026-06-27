---
layout: post
title:  "AWS Network Troubleshooting"
categories: Cloud
author: Uchechukwu Onyekwuluje
---

This is a curated list of troubleshooting tips in AWS Cloud Environment. AWS Networking can become complex as a given infrastructure grows.
Dealing with networking issues almost always comes down to understanding the building blocks and basics. 

### Public Access from Private Resource
*Consider this case.* AWS ec2 instance. you can telnet private ip of sftp family server but not the public ip address. When security group has 0.0.0.0/0, it works. When security group has VPC CIDR of ec2 instance, it does not work

```
curl ifconfig.me
curl https://checkip.amazonaws.com
```
If this returns a public IP, then your traffic to the SFTP server's public endpoint, the public server or public facing resource is leaving the VPC through a NAT Gateway/Internet Gateway and re-entering AWS. The SFTP server, public ec2 instance or resource sees that public IP, not the EC2's private IP. Adjust the security group accordingly
