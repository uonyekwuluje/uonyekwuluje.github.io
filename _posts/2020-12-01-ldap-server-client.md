---
layout: post
title:  "OpenLDAP Server and Client Setup"
categories: ldap
---

OpenLDAP is am open-source implementation of the Lightweight Directory Access Protocol. It is used for consolidating services and resources in one directory services which will be further accessed and managed by the LDAP Client like email client, mail servers, web browsers. LDAP uses TCP/IP stack to access and manage the directory services.
<br>
In this post, I will be installing an LDAP Server and two clients. The primary use case here is to centralize and manage user accounts and logins in a linux environment.

### **Servers**
|Component    | Systems Specification    | Operating System 
|-------------|--------------------------|------------------
|ldapserver   | 2 CPU  4GB RAM  20GB HDD |  CentOS 7.x
|ldapclient01 | 1 CPU  2GB RAM  20GB HDD |  CentOS 7.x
|ldapclient02 | 1 CPU  2GB RAM  20GB HDD |  Ubuntu 18.04 LTS

**NOTE:**
You can make changes as needed. The above is just a base systems spec.
