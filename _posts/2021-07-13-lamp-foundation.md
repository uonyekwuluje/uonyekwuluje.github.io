---
layout: post
title:  "Linux Apache2 MysQL/MariaDB PHP Foundation"
categories: LAMP 
---

The LAMP Stack stands for Linux Apache MySQL PHP. This stack forms the basis of most websites and solutions built today. They
integrate seamlessly and are very easy to setup and configure. This is more of a quick reference for quick setup. 

#### **Linux**
This can be any distribution of your choice. I'll focus on Ubuntu. My reason for Ubuntu is that it has good updates and
package integration. You will seldom find yourself hunting down packages if APT does not have them. This post assumes
you have a clean ubuntu **18.04 LTS** or **20.04 LTS** setup

#### **Apache2**
Type this commands to install `Apache2`. `sudo apt install -y apache2`. Ensure apache2 services are up
```
sudo systemctl start apache2.service
sudo systemctl enable apache2.service
```

#### **MariaDB**
Type this commands to install `MariaDB`. `sudo apt install -y mariadb-server mariadb-client`. Ensure mariaDB services are up
```
sudo systemctl start mariadb.service
sudo systemctl enable mariadb.service
```
When this is configured, you can update and configure as needed with this command. `sudo mysql_secure_installation`. When complete,
test mysql login. At least this should work `sudo mysql`. You can proceed to create a test database and user
```
CREATE DATABASE testdb;
CREATE USER 'testuser'@'localhost' IDENTIFIED BY 'securepassword';
GRANT ALL ON testdb.* TO 'testuser'@'localhost' IDENTIFIED BY 'securepassword' WITH GRANT OPTION;
FLUSH PRIVILEGES;
EXIT;
```
Test your connection with `mysql -u <user> -p<password>`


#### **PHP**
Type this commands to install PHP PPA Repo. 
```
sudo apt-get install software-properties-common
sudo add-apt-repository ppa:ondrej/php -y
sudo apt update
```
after this, proceed to install PHP:
```
sudo apt install -y php7.4 php7.4-common php7.4-mbstring php7.4-xmlrpc php7.4-gd php7.4-xml php7.4-intl php7.4-mysql php7.4-cli php7.4 php7.4-ldap php7.4-zip php7.4-curl libapache2-mod-php7.4 php7.4-json php7.4-imagic unzip imagemagick 
```
You can also update `php.ini`. Example in this use case will be `sudo vim /etc/php/7.4/apache2/php.ini`
```
file_uploads = On
allow_url_fopen = On
memory_limit = 256M
upload_max_filesize = 100M
date.timezone = America/New_York
```
***NOTE:*** *While PHP8 works, you will run into some breaking changes. May not cause issues in your code base*
Create a test config
```
sudo bash -c 'cat <<EOF> /var/www/html/info.php
<?php
  phpinfo();
?>
EOF'
```
Restart apache2 `sudo systemctl restart apache2` and test `http://localhost/info.php`

#### **LAMP Variants**
Other variants of this stack exist. You can have
* **LEMP** (Linux, NGINX, MySQL/MariaDB, PHP/Perl/Python)
* **LAPP** (Linux, Apache, PostgreSQL, PHP)
* **LEAP** (Linux, Eucalyptus, AppScale, Python)
* **LLMP** (Linux, Lighttpd, MySQL/MariaDB, PHP/Perl/Python)
