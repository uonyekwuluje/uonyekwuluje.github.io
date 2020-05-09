---
layout: post
title:  "Tomcat9 Centos Installation"
categories: Java, Tomcat
---

Tomcat is an open-source implementation of the Java Servlet, JavaServer Pages, Java Expression Language, and Java WebSocket technologies.
This tutorial covers the steps required to install Tomcat 9.0 on CentOS 8. Refer to [Apache Tomcat](https://tomcat.apache.org/index.html)
for more information 

**Requirements**<br>
This post assumes the following:
* You have a CentOS Linux System
* You have admin permissions and privilleges
* The server is accessible via HTTP/HTTPS

**Package Setup**<br>
This is optional if you already have your base package setup. If you don't run the commands below:
```
sudo yum group install -y 'Development Tools'
sudo yum install -y libxml2 libxml2-devel libxslt libxslt-devel wget gcc \
libffi-devel openssl-devel make openssl-devel bzip2-devel java-11-openjdk-devel java-11-openjdk
```

**Create Tomcat Service Account**<br>
You need a service account.
```
sudo groupadd --system tomcat
sudo useradd -s /bin/false -g tomcat -d /opt/tomcat tomcat
```

**Download and Install Tomcat**<br>
Download and install Tomcat
```
cd /tmp
sudo mkdir /opt/tomcat
TVERSION="9"
TOMCAT_VERSION="9.0.34"
curl -O https://downloads.apache.org/tomcat/tomcat-${TVERSION}/v${TOMCAT_VERSION}/bin/apache-tomcat-${TOMCAT_VERSION}.tar.gz
sudo tar xzvf apache-tomcat-${TVERSION}*tar.gz -C /opt/tomcat --strip-components=1

cd /opt/tomcat
sudo chgrp -R tomcat /opt/tomcat
sudo chmod -R g+r conf
sudo chmod g+x conf
sudo chown -R tomcat webapps/ work/ temp/ logs/
```

**Create Tomcat Service**<br>
Create Systemd Service for tomcat
```
sudo vim /etc/systemd/system/tomcat.service
```
Make the entry below. ***NOTE: Update your Java Version as needed***
```
[Unit]
Description=Apache Tomcat Web Application Container
After=network.target

[Service]
Type=forking

Environment=JAVA_HOME=/usr/lib/jvm/jre-11-openjdk
Environment=CATALINA_PID=/opt/tomcat/temp/tomcat.pid
Environment=CATALINA_HOME=/opt/tomcat
Environment=CATALINA_BASE=/opt/tomcat
Environment='CATALINA_OPTS=-Xms512M -Xmx1024M -server -XX:+UseParallelGC'
Environment='JAVA_OPTS=-Djava.awt.headless=true -Djava.security.egd=file:/dev/./urandom'

ExecStart=/opt/tomcat/bin/startup.sh
ExecStop=/opt/tomcat/bin/shutdown.sh

User=tomcat
Group=tomcat
UMask=0007
RestartSec=10
Restart=always

[Install]
WantedBy=multi-user.target
```
Enable and start tomcat
```
sudo systemctl daemon-reload
sudo systemctl enable tomcat
sudo systemctl start tomcat
```

**Update Firewall**
Update firewall rukes and reload
```
sudo firewall-cmd --zone=public --permanent --add-port=8080/tcp
sudo firewall-cmd --reload
```

**Configure The Management Interface**<br> 
Edit ```/opt/tomcat/conf/tomcat-users.xml``` with the update below:
``` 
<tomcat-users>
<!--
    Comments
-->
   <role rolename="admin-gui"/>
   <role rolename="manager-gui"/>
   <user username="admin" password="admin_password" roles="admin-gui,manager-gui"/>
</tomcat-users>
```

To enable web management interface outside localhost update the following config files:<br>
```
sudo vim /opt/tomcat/webapps/manager/META-INF/context.xml
```
Comment of removing the block below: ***Note: I have commneded this already***
```
<Context antiResourceLocking="false" privileged="true" >
<!--
  <Valve className="org.apache.catalina.valves.RemoteAddrValve"
         allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1" />
-->
</Context>
```
Next
```
sudo vim /opt/tomcat/webapps/host-manager/META-INF/context.xml
```
Comment of removing the block below: ***Note: I have commneded this already***
```
  <!--<Valve className="org.apache.catalina.valves.RemoteAddrValve"
     allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1" />-->
```

Restart Tomcat:
```
sudo systemctl restart tomcat
```

**Test Tomcat**<br>
Open your browser and type the following
```
http://<Your IP Address>:8080
```

**Caution and Notes**<br>
The server status in the admin page should be your best friend. This is primarily for monitoring and resource utilization
```
