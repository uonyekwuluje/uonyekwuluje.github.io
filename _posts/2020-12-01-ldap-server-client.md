---
layout: post
title:  "OpenLDAP Server and Client Setup"
categories: ldap
---

OpenLDAP is an open-source implementation of the Lightweight Directory Access Protocol. It is used for consolidating services and resources in one directory services which will be further accessed and managed by the LDAP Client like email client, mail servers, web browsers. LDAP uses TCP/IP stack to access and manage the directory services.
<br>
In this post, I will be installing an LDAP Server and two clients. The primary use case here is to centralize and manage user accounts and logins in a linux environment.

# **Server Stack**

  Component   |  Systems Specification   | Operating System  |      FQDN
 ------------ | ------------------------ | ----------------- | ------------------------------
 ldapserver   | 2 CPU  4GB RAM  20GB HDD |  CentOS 7.x       |  ldapserver.infralabs.com
 ldapclient01 | 1 CPU  2GB RAM  20GB HDD |  CentOS 7.x       |  ldapclient01.infralabs.com
 ldapclient02 | 1 CPU  2GB RAM  20GB HDD |  Ubuntu 18.04 LTS |  ldapclient02.infralabs.com

**NOTE:** *You can make changes as needed. The above is just a base systems spec.*

### **Install LDAP Server**
Install LDAP Server Components
```
sudo yum update -y
sudo yum group install -y 'Development Tools'
sudo yum -y install openldap compat-openldap openldap-clients openldap-servers \
openldap-devel nss-pam-ldapd openldap-servers-sql net-tools
```
Enable and start LDAP service
```
sudo systemctl enable slapd
sudo systemctl start slapd
```

### **Generate LDAP Root Password**
Run the command below. Substitute ```ldppassword``` with your desired password:
```
slappasswd -h {SSHA} -s ldppassword
```
You should see something like ```{SSHA}r+XyJvpQ+NPTBA4YP3+gcoR5fF8sh+G3```
