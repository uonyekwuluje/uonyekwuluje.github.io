---
layout: post
title:  "GitHub Actions Self Hosted Runner Setup"
categories: Automation CICD
---

GitHub Actions is an API for cause and effect on GitHub. Orchestrate and automate any workflow, based on any event. 
With GitHub Actions, workflows and steps are just code in a repository, so you can create, share, reuse your software 
development practices.
<br>
A core component of this process is the use of runners. Runners/Agents etc. are essentially the work horses of most
CICD systems and Github Actions is no exception. in this post, we will be focusing on installing and configuring 
self hosted runners.  

### **Requirements**
For the purpose of this post, setup or provision a VM with the following specs:
* Ubuntu 18.04/20.04 LTS.
* Admin Privilleges on the server
* Network and firewall configuration as needed. *In this post, we will be using CentOS 7 and CentOS8*

