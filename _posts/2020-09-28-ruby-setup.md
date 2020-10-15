---
layout: post
title:  "Ruby Setup"
categories: Ruby
---

This tutorial documents how to setup ruby in a linux system. I worked on a project a while back and it's core was centered
around Ruby. I got it working after some early struggles and decided to document my steps for future reference.


# Requirements
Ubuntu System Requirements:
```
sudo apt-get update
sudo apt-get install -y curl gnupg build-essential
```

Redhat/CentOS System Requirements:
```
sudo yum install -y curl gpg gcc gcc-c++ make
```


# Install Ruby RVM
Install RVM: *Adjust firewall as needed*
```
sudo gpg --keyserver hkp://pool.sks-keyservers.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3 7D2BAF1CF37B13E2069D6956105BD0E739499BDB
curl -sSL https://get.rvm.io | bash -s stable
sudo usermod -a -G rvm `whoami`
source /home/`whoami`/.rvm/scripts/rvm
```


# Install Ruby with RVM
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

# Install Ruby from Source
While this is dated, use cases exists for setting up ruby from source to ensure system wide access and stability.
```
RUBY_MAJOR_VERSION=2.7
RUBY_MINOR_VERSION=2.7.1
cd /tmp
wget https://cache.ruby-lang.org/pub/ruby/$RUBY_MAJOR_VERSION/ruby-$RUBY_MINOR_VERSION.tar.gz
tar xzf ruby-$RUBY_MINOR_VERSION.tar.gz
rm ruby-$RUBY_MINOR_VERSION.tar.gz
sudo mv ruby-${RUBY_MINOR_VERSION} /opt/
cd /opt/ruby-${RUBY_MINOR_VERSION}
sudo ./configure
sudo make
sudo make install
ruby -v
```




# Reference Links
* [RVM Installation Guide](https://rvm.io/rvm/install)
* [Ruby Development Tutorial](https://www.phusionpassenger.com/library/walkthroughs/deploy/ruby/ownserver/nginx/oss/install_language_runtime.html)
* [Ruby Troubleshooting Guide](https://bundler.io/guides/rubygems_tls_ssl_troubleshooting_guide.html?utm_source=ruby-ssl-check#troubleshooting-protocol-errors)
