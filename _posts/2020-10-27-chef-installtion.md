---
layout: post
title:  "Chef Server & Workstation Installation"
categories: OpsCode Chef
---

Chef is a configuration management tool written in Ruby. It is used for configuring and managing servers. With installed plugins, chef can be used to manage any cloud environment. A functioning system comprises a Chef Server, a Chef Workstation and target clients.
The chef workstation is used for developing recipes and cookbooks. This is then uploaded to the chef server and the clients pull to update their configs.
In this post, we are going to install Chef 13 server, workstation on CentOS7 and test a sample cookbook on a target client.

# **System Requirements**
Systems specification for chef server and chef workstation
| Component         | Systems Specification          | Operating System  |
| ----------------- | ------------------------------ | ----------------- |
| chefserver        | 2 CPU  4GB RAM  20GB Storage   | CentOS 7/RHEL 7   |
| chefworkstation   | 2 CPU  4GB RAM  20GB Storage   | CentOS 7/RHEL 7   |

**NOTE:**
* You can make changes as needed. The above is just a base systems spec.
* I intentionally skipped the client for later.
* For Debian/Ubuntu, MAC, Windows etc. Download components for your operating system [here](https://downloads.chef.io/)

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
**Create User:**<br>
Run the command below to create a user in the chef server
```
sudo chef-server-ctl user-create chefadmin Chef Admin chefadmin@local.dev usesecurepassword -f /etc/chef/chef-admin.pem
```
*note: change entry as needed.*

**Create Organization:**<br>
Run the command below to create an organization and associate the admin user
```
sudo chef-server-ctl org-create devenv "Dev Environment" --association_user chefadmin -f /etc/chef/devenv-validator.pem
```

# **Create Chef Manage**
Run the commands below to create Chef Manage:
```
sudo chef-server-ctl install chef-manage 
sudo chef-server-ctl reconfigure 
sudo chef-manage-ctl reconfigure 
```
*note: enter yes for all defaults*

**Test Chef UI:**<br>
Open a browser and test
```
FQDN: https://192.168.1.141
username: chefadmin
password: usesecurepassword
```


# **Chef Workstation Installation**
Run the following on your designated chef workstation. This installs the required packages on chef workstation.
```
sudo yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
sudo yum update -y
sudo yum group install -y 'Development Tools'
sudo yum install -y libxml2 libxml2-devel libxslt libxslt-devel wget gcc \
libffi-devel openssl-devel make openssl-devel bzip2-devel vim screen nc jq
```
**Install Chef Workstation:**<br>
Run the command below in the chef work station
```
curl -sO https://packages.chef.io/files/stable/chef-workstation/20.7.96/el/7/chef-workstation-20.7.96-1.el7.x86_64.rpm
sudo rpm -ivh chef-workstation-20.7.96-1.el7.x86_64.rpm 
```

**Setup bash profile**<br>
```
echo 'eval "$(chef shell-init bash)"' >> ~/.bash_profile
source ~/.bash_profile
which ruby
```
Run this command to verify your client ```chef-client -v```. You should see something like this
```
Chef Infra Client: 16.2.73
```

**Configure Workstation**<br>
For the purpose of this post, I will use the default credentials created for the server. This will be the admin user pem file and validator pem file.
For production use, non privilleged accounts should be created with the right permission and scope.
Copy ```/etc/chef/chef-admin.pem``` and ```/etc/chef/devenv-validator.pem``` from master and update the chef workstation
```
mkdir ~/.chef
scp <user>@<chef server ip>:/etc/chef/chef-admin.pem .chef/
scp <user>@<chef server ip>:/etc/chef//etc/chef/devenv-validator.pem .chef/
```

**Generate Repo**<br>
Generate Chef Repo:
```
chef generate repo chef-repo
```
*note: yes for all defaults*

**Configure Knife**<br>
Create a new knife config ```vim ~/.chef/config.rb``` with these
```
current_dir              = File.dirname(__FILE__)
log_level                :info
log_location             STDOUT
node_name                'chefadmin'
client_key               "#{current_dir}/chef-admin.pem"
validation_client_name   'multienv-validator'
validation_key           "#{current_dir}/multienv-validator.pem"
chef_server_url          'https://chefserver.home/organizations/devenv'
syntax_check_cache_path  '/home/centos/.chef/syntax_check_cache'
cookbook_path            "/home/centos/chef-repo/cookbooks"
```

**Fetch SSL Certificates**<br>
Run this command to fetch ssl certificates ```knife ssl fetch```. You should see this:
```
WARNING: Certificates from chefserver.home will be fetched and placed in your trusted_cert
       directory (/home/centos/.chef/trusted_certs).
       
       Knife has no means to verify these are the correct certificates. You should
       verify the authenticity of these certificates after downloading.
Adding certificate for chefserver_home in /home/centos/.chef/trusted_certs/chefserver_home.crt
```
Check SSL certificate using this command ```knife ssl check```. You should see.
```
Connecting to host chefserver.home:443
Successfully verified certificates from `chefserver.home'
```

