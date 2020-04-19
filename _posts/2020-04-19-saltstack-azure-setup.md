---
layout: post
title:  "Saltstack Microsoft Azure Setup and Configuration"
categories: Configuration Management, Azure
---

If you use Microsoft Azure as your cloud provider, you may find out that one of the features used the most is compute.
From an IaaS IaC point of view salt may not be the best tool at scale but it works if you are dealing with adhoc requests
for developers. This post is geared at showing you how this can be accomplished using salt.

**Requirements**<br>
This post assumes the following:
* You have a valid Azure Subscription
* You have admin permissions and privilleges
* You have a valid Azure Service Principal 
* You have a working saltmaster. Please refer to [Saltstack Setup](https://uonyekwuluje.github.io/2019-12-23/saltstack-setup/) for more information

**Microsoft Azure Repository**<br>
This is optional. If you need added packages for your setup which are azure specific, set this up:
```
sudo rpm -ivh https://packages.microsoft.com/config/rhel/7/packages-microsoft-prod.rpm
sudo yum update -y
sudo yum install -y blobfuse cifs-utils
```

**Create Certificates For SaltStack**<br>
You need to create a certificate for use with salt stack. The **.cer** needs to be uploaded to the Certificate Manager 
in azure.
```
openssl req -x509 -nodes -days 365 -newkey rsa:1024 -keyout /etc/salt/azure.pem -out /etc/salt/azure.pem
openssl x509 -inform pem -in /etc/salt/azure.pem -outform der -out /etc/salt/azure.cer
```


**Configure SaltCloud Provider**<br>
Create a new config for azure. ```/etc/salt/cloud.providers.d/azure.conf```
```
azure-config:
  driver: azurearm
  subscription_id: <YOUR SUBSCRIPTION ID>
  tenant: <YOUR TENANT ID>
  client_id: <YOUR CLIENT ID>
  secret: <YOUR CLIENT SECRET> 
  cloud_environment: AZURE_PUBLIC_CLOUD
  certificate_path: /etc/salt/azure.pem
  location: <YOUR REGION>
  minion:
    master: <SALTMASTER PRIVATE IP>

  ssh_interface: private_ips
  management_host: management.core.windows.net

  cleanup_disks: True
  cleanup_vhds: True
  cleanup_data_disks: True
  cleanup_interfaces: True
  #custom_data: 'This is custom data'
  expire_publisher_cache: 604800  # 7 days
  expire_offer_cache: 518400  # 6 days
  expire_sku_cache: 432000  # 5 days
  expire_version_cache: 345600  # 4 days
  expire_group_cache: 14400  # 4 hours
  expire_interface_cache: 3600  # 1 hour
  expire_network_cache: 3600  # 1 hour
```


**Configure SaltCloud VM Profile**<br>
Create a new config profile for azure virtual machines. ```/etc/salt/cloud.profiles.d/azure.conf```
```
azure-rhel7:
  provider: azure-config
  image: 'RedHat|RHEL|7-LVM|latest'
  size: Standard_D1_v2
  location: 'East US'
  ssh_username: azureuser
  ssh_password: <choose password of your choice>
  resource_group: <Valid Resource Group>
  network: <Valid Network>
  subnet: <Valid subnet>
  allocate_public_ip: False
  bootstrap_interface: private

  volumes:
  - disk_size_gb: 50
    caching: ReadWrite
  - disk_size_gb: 100
    caching: ReadWrite
    managed_disk:
      storage_account_type: Standard_LRS
```
Please note the following:
* You will need a valid subnet name
* You will need a valid resource group name
* You will need a valid network name


**Test Your Config**<br>
When You are through, restart the saltmaster service and test your config:
```
sudo systemctl restart salt-master.service
```
Test Config with:
```
sudo salt-cloud -u
```
you should see something like:
```
    ----------
    Files updated:
        - /etc/salt/cloud.deploy.d/bootstrap-salt.sh
        - /usr/lib/python2.7/site-packages/salt/cloud/deploy/bootstrap-salt.sh
```

You can also try the following to be sure api works
```
sudo salt-cloud --list-images azure-config
sudo salt-cloud --list-sizes azure-config
sudo salt-cloud --list-locations azure-config
```


**Create a Virtual Machine**<br>
Type the following commands to bootstrap a virtual machine using our profile for RHEL 7:
```
sudo salt-cloud -p azure-rhel7 testvm1 -l debug
sudo salt-cloud -p azure-rhel7 testvm2 -l debug
```

**Test Ping Virtual Machines**<br>
Type the following command to perform a test ping:
```
sudo salt '*' test.ping
```
you should see this:
```
testvm1:
    True
testvm2:
    True
```

**Delete Azure Virtual Machines**<br>
Type the following commands to delete the virtual machines:
```
sudo salt-cloud -a destroy testvm1 testvm2 -y
```

#### **Reference Links**
* Saltstack Cloud [salt-cloud](https://docs.saltstack.com/en/latest/topics/cloud/)
* Salt Coud Azure [salt azure](https://docs.saltstack.com/en/latest/topics/cloud/azure.html)

