---
layout: post
title: "SSL Grade A+ using Nginx"
author: pratyush
categories: [ Tutorial ]
image: assets/images/nginx-ssl-grade-a.webp
description: "Achieve A+ in SSL Labs using Nginx"
featured: true
hidden: true
---

How to achieve A+ rating on SSL Labs test on a server running Nginx with Openssl version 1.0.2k.

### Enable SSL Termination and HTTP2
```
listen 443 ssl http2;
```
### Key Exchange
To increase the Key Exchange score a new key needs to be generated with custom DH parameters. Running the command `openssl dhparam -out /etc/nginx/dhparam.pem 4096` will create a file to be used in `ssl_dhparam` directive.

### SSL Protocols
SSLv3 and TLSv1.1 should be disabled.
> The TLSv1.3 parameter works only when OpenSSL 1.1.1 or higher is used.

### HTTP Strict Transport Security (HSTS)
HTTP Strict Transport Security (HSTS) is a policy mechanism that helps to protect websites against man-in-the-middle attacks such as protocol downgrade attacks and cookie hijacking. It allows web servers to declare that web browsers should interact with it using only HTTPS connections.

### Here’s the complete configuration
```
server {
        listen                          80;
        server_name                     localhost;
        rewrite ^(.*)$                  https://$host$request_uri? permanent;
}
server {
        listen                          443 ssl http2;
        server_name                     localhost;

        ssl                             on;
        ssl_certificate                 /etc/nginx/conf.d/ssl/server.crt;
        ssl_certificate_key             /etc/nginx/conf.d/ssl/private.key;
        ssl_dhparam                     /etc/nginx/conf.d/ssl/dhparam.pem;

        server_tokens                   off;
        ssl_protocols                   TLSv1.2;
        ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;
        ssl_prefer_server_ciphers       on;

        ssl_session_timeout             1d;
        ssl_session_cache               shared:MozSSL:10m;
        ssl_session_tickets             off;

        add_header Strict-Transport-Security "max-age=63072000" always;
        gzip                            on;

        location / {
                root   /usr/share/nginx/html;
                index  index.html index.htm;
        }
}
```