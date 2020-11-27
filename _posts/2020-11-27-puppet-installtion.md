---
layout: post
title:  "Puppet Server & Agent Installation"
categories: Puppet
---

Puppet is a configuration management tool used for configuring and managing servers. A functioning system comprises a Puppet Server and a Puppet Agent.
In this post, we are going to install Puppet 6.x on CentOS 8.

# **System Requirements**
Systems specification for our POC

   Component    |      Hostname     |  IP Address   |   Systems Specification      |  Operating System  
 -------------- | ----------------- | ------------- | ---------------------------- | ------------------- 
 puppetserver   | puppetserver.home | 192.168.1.156 | 2 CPU  4GB RAM  20GB Storage |  CentOS 8/RHEL 8   
 puppetagent1   | puppetagent1.home | 192.168.1.250 | 1 CPU  2GB RAM  10GB Storage |  CentOS 8/RHEL 8   

**NOTE:** *You can make changes as needed. The above is just a base systems spec.*

# **CentOS 8 Puppet Server Installation**
Run the following on your designated puppet server. 
```
sudo rpm -Uvh https://yum.puppet.com/puppet6-release-el-8.noarch.rpm
sudo dnf update -y
sudo dnf install -y puppetserver
```
Update Memory Config in ```/etc/sysconfig/puppetserver```. In my case, I am updating from 2g to 1g
```
JAVA_ARGS="-Xms1g -Xmx1g -Djruby.logger.class=com.puppetlabs.jruby_utils.jruby.Slf4jLogger"
```

### Update puppetserver configs
Edit ```/etc/puppetlabs/puppet/puppet.conf``` and update with entries below:
```
# Pupper Server Configuration
[master]
dns_alt_names = puppetserver.home,puppet,puppetserver

# Puppet Agent Configuration
[main]
certname = puppetserver.home
server = puppetserver.home
environment = production
runinterval = 1h
```

### Setup Puppet CA
```
sudo /opt/puppetlabs/bin/puppetserver ca setup
```
if all goes well, you should have this
```
Generation succeeded. Find your files in /etc/puppetlabs/puppet/ssl/ca
```

### Enable Puppet Service
```
sudo systemctl enable puppetserver
sudo systemctl start puppetserver
sudo systemctl status puppetserver
```
<br><br>


# **CentOS 8 Puppet Agent Installation**
Run the following on your CentOS 8 Agents
```
sudo rpm -Uvh https://yum.puppet.com/puppet6-release-el-8.noarch.rpm
sudo dnf update -y
sudo dnf install -y puppet-agent
```
### Update Agent Configs
Edit ```/etc/puppetlabs/puppet/puppet.conf``` with the code below:
```
[main]
certname = puppetagent1.home
server = puppetserver.home
environment = production
runinterval = 1h
```
Start puppet agent on the node and make it start automatically on system boot.
```
sudo /opt/puppetlabs/bin/puppet resource service puppet ensure=running enable=true
```
if this works well, you should see this:
```
Notice: /Service[puppet]/ensure: ensure changed 'stopped' to 'running'
service { 'puppet':
  ensure   => 'running',
  enable   => 'true',
  provider => 'systemd',
}
```
<br><br>
# **Sign Certificates**
To sign agent node certificate on master server, loginto the agent and type this command
```
sudo /opt/puppetlabs/bin/puppet agent --test
```
If it works, you should have this:
```
Info: csr_attributes file loading from /etc/puppetlabs/puppet/csr_attributes.yaml
Info: Creating a new SSL certificate request for puppetagent1.home
Info: Certificate Request fingerprint (SHA256): 15:42:90:E6:52:A4:04:B2:08:FF:32:35:B5:1C:F5:06:A0:29:55:8C:A7:97:4A:20:4F:4E:1D:32:83:0F:4C:77
Info: Certificate for puppetagent1.home has not been signed yet
Couldn't fetch certificate from CA server; you might still need to sign this agent's certificate (puppetagent1.home).
Exiting now because the waitforcert setting is set to 0.
```
<br>
Loginto the Server and Sign the Certificate
Before: ```sudo /opt/puppetlabs/bin/puppetserver ca list --all```
```
Requested Certificates:
    puppetagent1.home       (SHA256)  15:42:90:E6:52:A4:04:B2:08:FF:32:35:B5:1C:F5:06:A0:29:55:8C:A7:97:4A:20:4F:4E:1D:32:83:0F:4C:77
Signed Certificates:
    puppetserver.home       (SHA256)  40:F2:40:2B:71:B7:E4:02:80:57:DD:60:B9:BB:02:F7:1B:92:88:F7:73:E7:94:25:78:97:F9:E8:7F:5C:ED:C3	alt names: ["DNS:puppetserver.home", "DNS:puppet", "DNS:puppetserver", "DNS:puppetserver.home"]	authorization extensions: [pp_cli_auth: true]
```
Sign Certificate: ```sudo /opt/puppetlabs/bin/puppetserver ca sign --certname puppetagent1.home```. You should see this 
```
Successfully signed certificate request for puppetagent1.home
```
After: ```sudo /opt/puppetlabs/bin/puppetserver ca list --all```. You should see this
```
Signed Certificates:
    puppetserver.home       (SHA256)  40:F2:40:2B:71:B7:E4:02:80:57:DD:60:B9:BB:02:F7:1B:92:88:F7:73:E7:94:25:78:97:F9:E8:7F:5C:ED:C3	alt names: ["DNS:puppetserver.home", "DNS:puppet", "DNS:puppetserver", "DNS:puppetserver.home"]	authorization extensions: [pp_cli_auth: true]
    puppetagent1.home       (SHA256)  81:27:3E:77:68:C9:AC:99:B5:86:57:01:A0:D9:8A:62:0D:C3:4A:16:F7:F8:C9:34:27:6E:57:B8:C9:E9:EE:B2
```
<br>
Log back in the agent and verify
```sudo /opt/puppetlabs/bin/puppet agent --test```. You should see this:
```
Info: Using configured environment 'production'
Info: Retrieving pluginfacts
Info: Retrieving plugin
Info: Retrieving locales
Info: Caching catalog for puppetagent1.home
Info: Applying configuration version '1606500654'
Notice: Applied catalog in 0.01 seconds
```
<br>
That is it. You can continue with module and artifact development
