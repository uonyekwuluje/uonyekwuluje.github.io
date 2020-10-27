---
layout: post
title:  "Chef Server & Workstation Installation"
categories: OpsCode Chef
---

Chef is a configuration management tool written in Ruby. It is used for configuring and managing servers. With installed plugins, chef can be used to manage any cloud environment. A functioning system comprises a Chef Server, a Chef Workstation and target clients.
The chef workstation is used for developing recipes and cookbooks. This is then uploaded to the chef server and the clients pull to update their configs.
In this post, we are going to install Chef 13 server, workstation on CentOS7 and test a sample cookbook on a target client.

# **Requirements**

| Component   | Systems Specification |
| ----------- | ----------- |
| chefserver  | 2 CPU  4GB RAM  20GB Storage      |
| chefworkstation   | 2 CPU  4GB RAM  20GB Storage        |


**NOTE:**
You can make changes as needed. The above is just a base systems spec.
