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
**Create the keystore**<br>
Create a new keystore for this task:
```
keytool -genkey -alias mydomain -keyalg RSA -keystore /etc/ssl/java/keystore.jks -storepass changethis -keysize 2048
```



