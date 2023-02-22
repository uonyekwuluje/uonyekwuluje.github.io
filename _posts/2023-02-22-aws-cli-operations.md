---
layout: post
title:  "AWS CLI Operations"
categories: Systems Administration
author: Uchechukwu Onyekwuluje
---

Working with AWS means I spend a lot of time on the command line. Here are a few commands that come in handy

### List Regions
```
aws ec2 describe-regions --output table
```

### List Availability Zones in a given Region
Run command above and tool with the `RegionName`
```
aws ec2 describe-availability-zones --region us-east-1 --output table
```
