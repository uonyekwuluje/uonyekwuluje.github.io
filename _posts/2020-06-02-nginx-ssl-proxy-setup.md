---
layout: post
title:  "Nginx Proxy Setup"
categories: nginx, ssl
---

This tutorial covers how to setup nginx as a Reverse Proxy to back end applications on a given host. We will also cover Basic Auth
and SSL setup. I put this together because I ran into situations were I needed a secure frontend to my applications and open source
tools. Usually these tools lack basic Auth in their free tier offerings. They are available for paid subscriptions but that covers
a different use case. If you are happy and satisfied with your POC, you will be better off with the paid offerings. So, lets get to
it.

**Requirements**<br>
This post assumes the following:
* You have installed nginx in your CentOS 7/CentOS 8 server. Besides location, the same should apply to other linux distributions
* You have admin permissions and privilleges
* You have your signed SSL Certificate
* You have your SSL Certificate keys

**Package Update**<br>
Ensure the following packages have been installed:
```
sudo yum install -y nginx httpd-tools
```

**Generate Basic Auth Credentials**<br>
Open your terminal and type the following:
```
sudo htpasswd -c /etc/nginx/.htpasswd webadmin 
```
This will prompt for a password for the user ```webadmin```. Enter something secure.
To append to the file, you can type the following:
```
sudo htpasswd /etc/nginx/.htpasswd tony
```
This will prompt for a password for user Tony. When you have made your entry; it should be added to the file.

If you ```cat /etc/nginx/.htpasswd``` the file you should see this:
```
webadmin:$apr1$s5U9Opz/$kv5fiTw6KvHJYt95oPTbH/
tony:$apr1$dlOH/ld5$OOvwDDQ89bkX2nxJvqxBn/
```



**Update Nginx Config**<br>
Edit your nginx config. It should be like ```/etc/nginx/nginx.conf```:
```
user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

# Load dynamic modules. See /usr/share/doc/nginx/README.dynamic.
include /usr/share/nginx/modules/*.conf;

events {
    worker_connections 1024;
}

http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 2048;

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;

    include /etc/nginx/conf.d/*.conf;

    server {
        listen       80 default_server;
        listen       [::]:80 default_server;
        server_name  _;
            return 301 https://$host$request_uri;
    }


   server {
        listen       443 ssl http2 default_server;
        listen       [::]:443 ssl http2 default_server;
        server_name  _;
        #root         /usr/share/nginx/html;
        auth_basic              "Restricted Access!";
        auth_basic_user_file    /etc/nginx/.htpasswd;

        ssl_certificate "/etc/ssl/certs/server.crt";
        ssl_certificate_key "/etc/ssl/certs/server.key";
        ssl_session_cache shared:SSL:1m;
        ssl_session_timeout  10m;
        ssl_protocols TLSv1.1 TLSv1.2;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        gzip_types *;
        proxy_set_header   Host              $host;
        proxy_set_header   X-Real-IP         $remote_addr;
        proxy_set_header   X-Forwarded-For   $proxy_add_x_forwarded_for;
        proxy_set_header   X-Forwarded-Host   $host:443;
        proxy_set_header   X-Forwarded-Server $host;
        proxy_set_header   X-Forwarded-Port 443;
        proxy_set_header   X-Forwarded-Proto $scheme;

        location / {
           proxy_pass http://localhost:9090;
        }

        location /alertmanager/ {
           proxy_pass http://localhost:9093/;
        }

        location /grafana/ {
           proxy_pass http://localhost:3000/;
        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }
}

```
The configuration above accomplishes the following:
* Proxys applications on local port 9090, 9093 and 3000 to web request paths for ```/``` , ```/alertmanager/``` and ```/grafana/```
* Provides basic Auth 
* Routes all ```HTTP``` requests to ```HTTPS``` 


Test your config ```sudo nginx -t -c /etc/nginx/nginx.conf```. You should see this
```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```
Restart nginx after making the update ```sudo systemctl restart nginx```
<br>
That's it. Your POC should now be secure and ready for use
