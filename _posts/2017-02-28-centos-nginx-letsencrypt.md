---
layout: post
title:  "Free HTTPS with Centos 7, Nginx, and Let's Encrypt"
date:   2017-02-28 20:13:00 -0500
categories: https centos nginx
---
Enabling HTTPS is free and easy with [Let's Encrypt](https://letsencrypt.org/) from ISRG. Here's how to do it using Nginx and Centos 7.

First, install certbot with:

```
sudo yum install certbot
```

Next add a acme-challenge route to your Nginx server config block. Note the root path defined here is used by certbot in the next step.

```
server {
...
  location ~ /\.well-known/acme-challenge {
    root /usr/share/nginx/html;
  }
...
}
```

Then run certbot certificate only with webroot set. This saves example.com's SSL certificates into /etc/letsencrypt/live/example.com/. Change www.example.com to your domain name. Be sure to include your domain and www subdomain variants.

```
sudo certbot certonly --webroot -w /usr/share/nginx/html -d example.com -d www.example.com
```

Lastly, update the Nginx config to use the newly generated SSL certificate and force HTTPS only access on our site.

```
server {
  listen 80 default_server;
  listen [::]:80 default_server;
  server_name www.example.com;
  return 301 https://$server_name$request_uri;
}

server {
  listen 443 default ssl;
  server_name www.example.com;
  ssl_certificate /etc/letsencrypt/live/www.example.com/fullchain.pem;
  ssl_certificate_key /etc/letsencrypt/live/www.example.com/privkey.pem;

  root /var/www//www.example.com/current/public;

  location ^~ /assets/ {
    gzip_static on;
    expires max;
    add_header Cache-Control public;
  }

  location ~ /\.well-known/acme-challenge {
    root /usr/share/nginx/html;
  }

  try_files $uri/index.html $uri @puma;

  location @puma {
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $http_host;
    proxy_redirect off;
    proxy_pass http://puma;
  }

  error_page 500 502 503 504 /500.html;
  client_max_body_size 10M;
  keepalive_timeout 10;
}
```