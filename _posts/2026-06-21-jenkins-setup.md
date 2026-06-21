---
layout: post
title:  "Jenkins Setup"
categories: Build Automation
author: Uchechukwu Onyekwuluje
---

Jenkins is one of the most popular CI/CD tool out there. It is used to automate a myriad of tasks ranging from software development tasks, testing code changes, packaging, deploying, infrastructure builds etc. The list goes on.

**Key Features:** Pipeline Automation: Define CI/CD processes as code. Extensibility: Supports over 1,400 plugins for various integrations (e.g., Git, Docker, Kubernetes). Distributed Builds: Run jobs across multiple machines for scalability. Easy Integration: Works with numerous tools and platforms (e.g., Maven, Gradle, GitHub). User-Friendly Interface: Simple web UI for job management and monitoring. Strong Community Support: Regular updates and extensive documentation.

### List Regions
```
aws ec2 describe-regions --output table
```

### List Availability Zones in a given Region
Run command above and tool with the `RegionName`
```
aws ec2 describe-availability-zones --region us-east-1 --output table
```
