---
layout: post
title:  "Chef Server & Workstation Installation"
categories: OpsCode Chef
---

Chef is a configuration management tool written in Ruby. It is used for configuring and managing servers. With installed plugins, chef can be used to manage any cloud environment. A functioning system comprises a Chef Server, a Chef Workstation and target clients.
The chef workstation is used for developing recipes and cookbooks. This is then uploaded to the chef server and the clients pull to update their configs.
In this post, we are going to install Chef 13 server, workstation on CentOS7 and bootstrap a target client.

# **System Requirements**
Systems specification for chef server and chef workstation

 Component         | Systems Specification          | Operating System  
 ---------------- | ---------------------------- | ---------------- 
 chefserver        | 2 CPU  4GB RAM  20GB Storage   | CentOS 7/RHEL 7   
 chefworkstation   | 2 CPU  4GB RAM  20GB Storage   | CentOS 7/RHEL 7   

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

**Create Chef Manage**<br>
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

# Bootstrap Existing Node
To bootstrap an existing node. Loginto the chef workstation and run the command below:
```
knife bootstrap 192.168.1.142 --node-name chefclient --connection-user centos --sudo --ssh-identity-file ~/.ssh/id_rsa
```
you should see this.
```
Connecting to 192.168.1.142 using ssh
The authenticity of host '192.168.1.142 ()' can't be established.
fingerprint is SHA256:etC4Kf1belep3aK4oZ7xk6udPIg3+lKc/rU7FN6vneU.

Are you sure you want to continue connecting
? (Y/N) Y
Connecting to 192.168.1.142 using ssh
Creating new client for chefclient
Creating new node for chefclient
Bootstrapping 192.168.1.142
 [192.168.1.142] -----> Installing Chef Omnibus (stable/16)
downloading https://omnitruck.chef.io/chef/install.sh
  to file /tmp/install.sh.1762/install.sh
 [192.168.1.142] trying curl...
 [192.168.1.142] el 7 x86_64
Getting information for chef stable 16 for el...
downloading https://omnitruck.chef.io/stable/chef/metadata?v=16&p=el&pv=7&m=x86_64
  to file /tmp/install.sh.1768/metadata.txt
 [192.168.1.142] trying curl...
 [192.168.1.142] sha1	4a39c750d80858fcc4e85f0a2374a311c4f7cec7
sha256	cbe863f28a8d9b061aea6e56d66f18f46ccc4bf50a54761ecaec9a5ab79952e1
url	https://packages.chef.io/files/stable/chef/16.6.14/el/7/chef-16.6.14-1.el7.x86_64.rpm
version	16.6.14
 [192.168.1.142] 
 [192.168.1.142] downloaded metadata file looks valid...
 [192.168.1.142] downloading https://packages.chef.io/files/stable/chef/16.6.14/el/7/chef-16.6.14-1.el7.x86_64.rpm
  to file /tmp/install.sh.1768/chef-16.6.14-1.el7.x86_64.rpm
 [192.168.1.142] trying curl...
 [192.168.1.142] Comparing checksum with sha256sum...
 [192.168.1.142] Installing chef 16
installing with rpm...
 [192.168.1.142] warning: /tmp/install.sh.1768/chef-16.6.14-1.el7.x86_64.rpm: Header V4 DSA/SHA1 Signature, key ID 83ef826a: NOKEY
 [192.168.1.142] Preparing...                          
 [192.168.1.142] ########################################
 [192.168.1.142] Updating / installing...
chef-16.6.14-1.el7                    
 [192.168.1.142] #
 [192.168.1.142] #
 [192.168.1.142] #
 [192.168.1.142] #
 [192.168.1.142] Thank you for installing Chef Infra Client! For help getting started visit https://learn.chef.io
 [192.168.1.142] Starting the first Chef Infra Client Client run...
 [192.168.1.142] +---------------------------------------------+
✔ 2 product licenses accepted.
+---------------------------------------------+
 [192.168.1.142] Starting Chef Infra Client, version 16.6.14
Patents: https://www.chef.io/patents
 [192.168.1.142] resolving cookbooks for run list: []
 [192.168.1.142] Synchronizing Cookbooks:
 [192.168.1.142] Installing Cookbook Gems:
Compiling Cookbooks...
[2020-10-27T13:22:41-04:00] WARN: Node chefclient has an empty run list.
 [192.168.1.142] Converging 0 resources
 [192.168.1.142] 
Running handlers:
Running handlers complete
Chef Infra Client finished, 0/0 resources updated in 02 seconds
```

**Test new node**<br>
Test new node with this command ```knife node list```. You should see 
```chefclient```

# Reference Links
* [Chef Downloads](https://downloads.chef.io/)
* [Chef Setup](https://docs.chef.io/)
