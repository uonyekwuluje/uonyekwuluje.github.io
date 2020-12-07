---
layout: post
title:  "HashiCorp Vault Setup"
categories: Security Automation, Secrets
---

Managing secrets, tokens, credentials, sensitive data etc., is one of the hallmarks of IT operations. They are lots of tools out there but we will focus on HashiCorp Vault. [HashiCorp Vault](https://www.vaultproject.io/) is a secrets management tool that helps to provide secure, automated access to sensitive data. In this post, we will install, configure, and access Vault in a DEV envronment to illustrate Vault’s features and API. We will use the latest version of Vault, which is 1.6.0 at the time of this writing.

### **Server Requirements**
We will be installing Vault in a CentOS 8 Server. Ensure you have the required permissions and you also have your firewalls configured accordingly.

### **Update Server**
Update Vault server and install required packages:
```
sudo dnf update -y
sudo dnf group install -y 'Development Tools'
sudo dnf install -y wget
```

### **Prepare Systems Directories**
Prepare directories and folders for our installation:
```
sudo mkdir -p /opt/vault/{logs,bin,data}
sudo mkdir /etc/vault
```

Create Vault user and set folder ownership:
```
sudo useradd -r -d /home/vault -s /bin/nologin vault
sudo chown -R vault:vault /opt/vault
```

### **Install and Configure Vault**
Download and install Vault:
```
VAULT_VERSION=1.6.0
wget https://releases.hashicorp.com/vault/${VAULT_VERSION}/vault_${VAULT_VERSION}_linux_amd64.zip
sudo unzip vault_${VAULT_VERSION}_linux_amd64.zip -d /opt/vault/bin
```
Test Vault with this command ```/opt/vault/bin/vault --version```. You should see this
```
Vault v1.6.0 (7ce0bd9691998e0443bc77e98b1e2a4ab1e965d4)
```
