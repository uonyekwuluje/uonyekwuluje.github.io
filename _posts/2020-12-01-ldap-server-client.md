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

# **LDAP Org Stack**

  Domain       |      Domain DN      |   LDAP Server  
 ------------- | --------------------| ----------------- 
 infralabs.com | dc=infralabs,dc=com |  ldapserver.infralabs.com

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
**NOTE:** *update your firewall as needed*

### **Generate LDAP Root Password**
Run the command below. Substitute ```securepassword``` with your desired password:
```
slappasswd -h {SSHA} -s securepassword
```
You should see something like ```{SSHA}qkFfyzbp1cOAaM81XNyFTBy0pexa+f1h```

Next, we need to modify ```olcDatabase={2}hdb.ldif```. Create a file name ```db_ldapdomain.ldif``` with the following content

```
dn: olcDatabase={1}monitor,cn=config
changetype: modify
replace: olcAccess
olcAccess: {0}to * by dn.base="gidNumber=0+uidNumber=0,cn=peercred,cn=external, cn=auth" read by dn.base="cn=ldapadm,dc=infralabs,dc=com" read by * none

dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcSuffix
olcSuffix: dc=infralabs,dc=com

dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcRootDN
olcRootDN: cn=ldapadm,dc=infralabs,dc=com

dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcRootPW
olcRootPW: {SSHA}fp290ZG8IcHC/9U8f+q4ScOrYebz8uqv

dn: olcDatabase={2}hdb,cn=config
changetype: modify
add: olcAccess
olcAccess: {0}to attrs=userPassword,shadowLastChange by
  dn="cn=service,dc=test,dc=com" write by anonymous auth by self write by * none
olcAccess: {1}to dn.base="" by * read
olcAccess: {2}to * by dn="cn=service,dc=test,dc=com" write by * read
```

Apply Updates
```
sudo ldapmodify -Y EXTERNAL  -H ldapi:/// -f db_ldapdomain.ldif
```

Copy Database sample and set permissions
```
sudo cp /usr/share/openldap-servers/DB_CONFIG.example /var/lib/ldap/DB_CONFIG
sudo chown ldap:ldap -R /var/lib/ldap/
```

Next Add the cosine and nis LDAP schemas.
```
sudo ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/cosine.ldif
sudo ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/nis.ldif 
sudo ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/inetorgperson.ldif
```

Next we add our base domain org. Create a file ```base.ldif``` with the contents below:
```
dn: dc=infralabs,dc=com
dc: infralabs
objectClass: top
objectClass: domain

dn: cn=ldapadm ,dc=infralabs,dc=com
objectClass: organizationalRole
cn: ldapadm
description: LDAP Manager

dn: ou=People,dc=infralabs,dc=com
objectClass: organizationalUnit
ou: People

dn: ou=Group,dc=infralabs,dc=com
objectClass: organizationalUnit
ou: Group
```
Next add this our database
```
ldapadd -x -W -D "cn=ldapadm,dc=infralabs,dc=com" -f base.ldif
```
**NOTE:** *type your password on the prompt*




### **Add Users**
Create a file ```user.db.ldif``` and populate with user data:
```
dn: uid=pparker,ou=People,dc=infralabs,dc=com
objectClass: top
objectClass: account
objectClass: posixAccount
objectClass: shadowAccount
cn: pparker
uid: pparker
uidNumber: 9900
gidNumber: 1001
homeDirectory: /home/pparker
loginShell: /bin/bash
gecos: Peter Parker
userPassword: {SSHA}r+XyJvpQ+NPTBA4YP3+gcoR5fF8sh+G3
shadowLastChange: 17058
shadowMin: 0
shadowMax: 99999
shadowWarning: 7

dn: cn=pparker,ou=People,dc=infralabs,dc=com
objectClass: top
objectClass: posixGroup
gidNumber: 1001
```
Add to database
```
sudo ldapadd -x -W -D "cn=ldapadm,dc=infralabs,dc=com" -f user.db.ldif
```
**NOTE:** *type your password on the prompt*




<br>
### **Install LDAP Client CentOS 7**
Update and install packages on Client
```
sudo yum update -y
sudo yum group install -y 'Development Tools'
sudo yum install -y openldap-clients nss-pam-ldapd net-tools
```
Configure Auth
```
sudo authconfig --enableldap --enableldapauth --ldapserver=ldapserver.infralabs.com \
--ldapbasedn="dc=infralabs,dc=com" --enablemkhomedir --update
```
Enable Services
```
sudo systemctl enable nslcd
sudo systemctl start nslcd
```
Verify LDAP Connectivity
```
getent passwd pparker
```
You should see this
```
pparker:x:9900:1001:Peter Parker:/home/pparker:/bin/bash
```
