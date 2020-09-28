---
layout: post
title:  "Ruby Setup"
categories: Ruby
---

This tutorial documents how to setup ruby in a linux system.


**Requirements**<br>
Ubuntu System Requirements:
```
sudo apt-get update
sudo apt-get install -y curl gnupg build-essential
```

Redhat/CentOS System Requirements:
```
sudo yum install -y curl gpg gcc gcc-c++ make
```


**Install Ruby RVM**<br>
Install RVM: *Adjust firewall as needed*
```
sudo gpg --keyserver hkp://pool.sks-keyservers.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3 7D2BAF1CF37B13E2069D6956105BD0E739499BDB
curl -sSL https://get.rvm.io | bash -s stable
sudo usermod -a -G rvm `whoami`
source /home/`whoami`/.rvm/scripts/rvm
```


**Install Ruby**<br>
Install Ruby: 
```
# Update to your desired version
rvm install ruby-2.6.0
rvm --default use ruby-2.6.0
```


**List Ruby**<br>
List Ruby: 
```
rvm list

=* ruby-2.6.0 [ x86_64 ]

# => - current
# =* - current && default
#  * - default
```



# Reference Links
* [RVM Installation Guide](https://rvm.io/rvm/install)
* [Ruby Development Tutorial](https://www.phusionpassenger.com/library/walkthroughs/deploy/ruby/ownserver/nginx/oss/install_language_runtime.html)
* [Ruby Troubleshooting Guide](https://bundler.io/guides/rubygems_tls_ssl_troubleshooting_guide.html?utm_source=ruby-ssl-check#troubleshooting-protocol-errors)
