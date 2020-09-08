---
layout: post
title:  "Sonarqube Setup"
categories: logging
---

Sonarqube is tool used for static code analysis. In more plain terms, it ensures code quality by checking for bugs, vulnerabilities and known issues. In this post, we will go through the base configuration of this tool.

#### **Requirements**
* One server. RHEL 7. 2vCPU and 6GiB Ram
* One PostgreSql Server. Version 10.


#### **Installation**

**Base Requirements:**<br>
Before installing and setting up sonarqube, we need to update the operating system and install some basic packages.
```
sudo yum update -y
sudo yum install -y java-11-openjdk-devel java-11-openjdk
sudo yum install -y wget unzip 
```

**Selinux:**<br>
We will be disabling selinux this post. In production you need to configure this for security:
```
sudo setenforce 0
sudo setenforce Permissive
```

**Systems Settings:**<br>
Update *sysctl /etc/sysctl.conf* with these enteries:
```
vm.max_map_count=524288
fs.file-max=131072
```

Update */etc/security/limits.conf* with these enteries:
```
sonarqube   -   nofile   131072
sonarqube   -   nproc    8192
```


*You can update this based on your systems requirements*

**Create Sonar User:**<br>
Create the sonar user account:
```
useradd sonar
mkdir -p /var/sonarqube/data
mkdir -p /var/sonarqube/temp
chown -R sonar:sonar /var/sonarqube
```

**Install & Configure PostgreSQL:**<br>
Download and install PostgreSql 10:
```
sudo yum install -y https://download.postgresql.org/pub/repos/yum/10/redhat/rhel-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
sudo yum install -y postgresql10-server postgresql10-contrib 
sudo /usr/pgsql-10/bin/postgresql-10-setup initdb

```

**Update PostgreSql Config:**<br>
Update PostgreSql config
```
sudo vim /var/lib/pgsql/10/data/pg_hba.conf
```
from
```
# TYPE  DATABASE        USER            ADDRESS                 METHOD

# "local" is for Unix domain socket connections only
local   all             all                                     peer
# IPv4 local connections:
host    all             all             127.0.0.1/32            ident
# IPv6 local connections:
host    all             all             ::1/128                 ident
# Allow replication connections from localhost, by a user with the
# replication privilege.
local   replication     all                                     peer
host    replication     all             127.0.0.1/32            ident
host    replication     all             ::1/128                 ident
```
to
```
# TYPE  DATABASE        USER            ADDRESS                 METHOD

# "local" is for Unix domain socket connections only
local   all             all                                     trust
# IPv4 local connections:
host    all             all             127.0.0.1/32            md5
# IPv6 local connections:
host    all             all             ::1/128                 md5
# Allow replication connections from localhost, by a user with the
# replication privilege.
local   replication     all                                     trust
host    replication     all             127.0.0.1/32            md5
host    replication     all             ::1/128                 md5
```
Enable and restart PostgreSql
```
sudo systemctl start postgresql-10
sudo systemctl enable postgresql-10
```

**Configure PostgreSql for SonarQube:**<br>
```
su - postgres
createuser sonar
psql

ALTER USER sonar WITH ENCRYPTED password 'P)8dEr7d)(*?Q';
CREATE DATABASE sonar OWNER sonar;
\q
exit
```

**Download, Install and Setup SonarQube:**<br>
```
cd /tmp
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-8.4.2.36762.zip
unzip sonarqube-8.4.2.36762.zip
mv sonarqube-8.4.2.36762 /opt/sonarqube
chown -R sonar:sonar /opt/sonarqube
```

Update sonar config
```
vim /opt/sonarqube/conf/sonar.properties

sonar.jdbc.username=sonar
sonar.jdbc.password=P)8dEr7d)(*?Q
sonar.jdbc.url=jdbc:postgresql://localhost/sonar
sonar.web.host=0.0.0.0
sonar.web.port=9000
sonar.web.context=/sonarqube
sonar.web.javaOpts=-server -Xms1024m -Xmx1024m -XX:+HeapDumpOnOutOfMemoryError
sonar.search.javaOpts=-server -Xms1024m -Xmx1024m -XX:+HeapDumpOnOutOfMemoryError
sonar.path.data=/var/sonarqube/data
sonar.path.temp=/var/sonarqube/temp
sonar.web.accessLogs.enable=true 
```

**Create SonarQube Service:**<br>
```
vim /etc/systemd/system/sonarqube.service

[Unit]
Description=SonarQube service
After=syslog.target network.target

[Service]
Type=forking
ExecStart=/opt/sonarqube/bin/linux-x86-64/sonar.sh start
ExecStop=/opt/sonarqube/bin/linux-x86-64/sonar.sh stop
LimitNOFILE=131072
LimitNPROC=8192
User=sonar
Group=sonar
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

Create and start Sonarqube service:
```
systemctl enable sonarqube.service
systemctl start sonarqube.service
systemctl status sonarqube.service
```


#### **Test**
Open your browser and check this address:
```
http://<ip address>:9000/sonarqube
```


#### **Logs & Diagnostics**
If you run into any issues, check these logs:
```
tail -f /opt/sonarqube/logs/sonar.log
tail -f /opt/sonarqube/logs/web.log
tail -f /opt/sonarqube/logs/es.log
tail -f /opt/sonarqube/logs/ce.log
```



#### **Reference Links**
* SonarQube Site [Sonarqube](https://www.sonarqube.org/)
