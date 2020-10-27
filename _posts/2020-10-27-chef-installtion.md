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
* You can make changes as needed. The above is just a base systems spec.
* I intentionally skipped the client for later.

# **Chef Server Installation**
Run the following on your designated chef server. This installs the required packages on the chef server.
```
sudo yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
sudo yum update -y
sudo yum group install -y 'Development Tools'
sudo yum install -y libxml2 libxml2-devel libxslt libxslt-devel wget gcc \
libffi-devel openssl-devel make openssl-devel bzip2-devel vim screen nc jq
```
**Install Chef:**<br>
```
curl -sO https://packages.chef.io/files/stable/chef-server/13.2.0/el/7/chef-server-core-13.2.0-1.el7.x86_64.rpm
sudo rpm -ivh chef-server-core-13.2.0-1.el7.x86_64.rpm
rm -Rf chef-server-core-13.2.0-1.el7.x86_64.rpm
```
**Configure Chef:**<br>
```
sudo chef-server-ctl reconfigure
```
*note: enter yes*

**Start & Test Chef:**<br>
```
sudo chef-server-ctl start
sudo chef-server-ctl test
```
*note: you will see long output text on your screen. that is fine. lookout for any error messages*

**Check Status:** Run this command ```sudo chef-server-ctl status``` to check the status. You should have this
```
run: bookshelf: (pid 19598) 565s; run: log: (pid 12880) 686s
run: nginx: (pid 19498) 568s; run: log: (pid 13565) 631s
run: oc_bifrost: (pid 19445) 569s; run: log: (pid 12511) 715s
run: oc_id: (pid 19474) 568s; run: log: (pid 12574) 707s
run: opscode-erchef: (pid 19612) 565s; run: log: (pid 13035) 679s
run: opscode-expander: (pid 19587) 566s; run: log: (pid 12734) 690s
run: opscode-solr4: (pid 19573) 566s; run: log: (pid 12649) 698s
run: postgresql: (pid 19437) 569s; run: log: (pid 12019) 728s
run: rabbitmq: (pid 20635) 531s; run: log: (pid 13845) 627s
run: redis_lb: (pid 19395) 571s; run: log: (pid 19394) 571s
``` 
