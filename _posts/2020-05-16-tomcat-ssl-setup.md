---
layout: post
title:  "Tomcat 9 SSL Setup"
categories: tomcat, ssl
---

The Apache Tomcat® software is an open source implementation of the Java Servlet, JavaServer Pages, Java Expression Language and Java WebSocket technologies. It has lots of features for administering your web application. In this post, we will cover the basics of setting up SSL/TLS to enable a secure setup. We will look at two implementations:
* JSSE implementation
* APR implementation


**Requirements**<br>
This post assumes the following:
* You have installed or have a working Tomcat Setup. Preferably Tomcat 9
* You have admin permissions and privilleges on the tomcat server
* You have your signed SSL Certificate
* You have your CA Certificate Bundle
* You have your SSL Certificate keys
* Your tomcat setup is in a linux server


**APR Implementation**<br>
Create a folder for your SSL Certificates:
```
sudo mkdir /etc/ssl/java
sudo chmod 700 /etc/ssl/java
```
Copy your SSL Certificates into this folder. Rename as needed:
```
vim /etc/ssl/java/server.com.crt
vim /etc/ssl/java/server.com.key
vim /etc/ssl/java/server.com.cabundle
```
Update permissions
```
chown -R tomcat:tomcat /etc/ssl/java/
```

Open your server.xml ```/opt/tomcat/conf/server.xml``` and add the following:
```
 <Connector protocol="org.apache.coyote.http11.Http11NioProtocol" port="8443"
               SSLEnabled="true" maxThreads="200" scheme="https" secure="true" 
               SSLCertificateFile="/etc/ssl/java/server.com.crt"
               SSLCertificateKeyFile="/etc/ssl/java/server.com.key"
               SSLVerifyClient="optional" SSLProtocol="TLSv1+TLSv1.1+TLSv1.2"/>

```
Save and restart tomcat service:
```
sudo systemctl stop tomcat
sudo systemctl start tomcat
```



<br><br><br>
**JSSE Implementation**<br>
Create a folder for your SSL Certificates:
```
sudo mkdir /etc/ssl/java
sudo chmod 700 /etc/ssl/java
```
Copy your SSL Certificates into this folder. Rename as needed:
```
vim /etc/ssl/java/server.com.crt
vim /etc/ssl/java/server.com.key
vim /etc/ssl/java/server.com.cabundle
```
Update permissions
```
chown -R tomcat:tomcat /etc/ssl/java/
```
Create a new keystore for this task:
```
keytool -genkey -alias yourdomain -keyalg RSA -keystore /etc/ssl/java/keystore.jks -storepass changethis -keysize 2048
```
Update permissions
```
chown -R tomcat:tomcat /etc/ssl/java/keystore.jks
```

<br>
Test to ensure you have the new entry in your keystore
```
keytool -list -keystore /etc/ssl/java/keystore.jks
```
Your should see something like:
```
Enter keystore password:  
Keystore type: PKCS12
Keystore provider: SUN

Your keystore contains 1 entry

yourdomain, May 15, 2020, PrivateKeyEntry, 
Certificate fingerprint (SHA-256): E8:06:39:EB:A8:B7:E7:09:D3:EA:6F:90:CA:7F:BE:7B:14:F2:3C:3B:EE:02:03:C0:88:77:20:58:C5:5E:6E:50
```
<br>
Export key, certificate and ca-bundle:
```
cat /etc/ssl/java/server.com.cabundle /etc/ssl/certs/ca-bundle.crt > /etc/ssl/java/server.com.allcabundle
openssl pkcs12 -export -in /etc/ssl/java/server.com.crt -inkey /etc/ssl/java/server.com.key -chain -CAfile /etc/ssl/java/server.com.allcabundle -name "server.com" -out /etc/ssl/java/server.com.p12

Enter Export Password:
Verifying - Enter Export Password:
```

<br>
Import the PKCS12 into your keystore:
```
keytool -importkeystore -deststorepass storePassword -destkeystore /etc/ssl/java/keystore.jks -srckeystore /etc/ssl/java/server.com.p12 -srcstoretype PKCS12

Importing keystore server.com.p12 to keystore.jks...
Enter source keystore password:  
Entry for alias server.com successfully imported.
Import command completed:  1 entries successfully imported, 0 entries failed or cancelled
```

<br>
Check your keystore to verify new entry:
```
keytool -list -keystore /etc/ssl/java/keystore.jks

Enter keystore password:  
Keystore type: PKCS12
Keystore provider: SUN

Your keystore contains 2 entries

yourdomain, May 15, 2020, PrivateKeyEntry, 
Certificate fingerprint (SHA-256): A5:06:2D:AF:DB:27:BD:E6:9D:D5:75:7B:FF:7D:E0:30:2F:AD:3F:0D:06:1E:6F:6B:B7:96:B8:A3:B9:72:5C:35
server.com, May 15, 2020, PrivateKeyEntry, 
Certificate fingerprint (SHA-256): B3:BC:48:0B:78:18:E9:5E:12:92:3B:CD:C7:8B:65:7C:62:4C:0E:2D:C4:EB:7E:C5:52:AE:08:41:B4:5C:BB:5D
```

<br>
Open your server.xml ```/opt/tomcat/conf/server.xml``` and add the following:
```
 <Connector protocol="org.apache.coyote.http11.Http11NioProtocol"
           port="8443" maxThreads="200" scheme="https" secure="true" SSLEnabled="true"
           keystoreFile="/etc/ssl/java/keystore.jks" keystorePass="storePassword" keyAlias="server.com"
           clientAuth="false" sslProtocol="TLS"/>
```
Save and restart tomcat service:
```
sudo systemctl stop tomcat
sudo systemctl start tomcat
```

<br>
### Reference
* [Apache Tomcat 9 SSL/TLS Configuration How-To](http://tomcat.apache.org/tomcat-9.0-doc/ssl-howto.html)
* [Coderwall](https://coderwall.com/p/3t4xka/import-private-key-and-certificate-into-java-keystore)<br>
*I fixed this error ```"Error unable to get issuer certificate getting chain."``` with comments in this post*
