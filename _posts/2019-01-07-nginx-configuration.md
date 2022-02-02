---
layout: post
title: "Nginx performance tuning and Security configuration"
author: pratyush
categories: [ Tuning, Tutorial, Security ]
image: assets/images/nginx-configuration.jpg
#beforetoc: "Markdown editor is a very powerful thing. In this article I'm going to show you what you can actually do with it, some tricks and tips while editing your post."
#toc: true
comments: true
---

Nginx (pronounced "engine X") is a web server that can also be used as a reverse proxy, load balancer, mail proxy and HTTP cache.

### Nginx Configuration Step by Step
The below is a complete configuration file with almost all used cases. Below are the use cases covered
1. Redireting of non www or http traffic to https and www.
2. SSL termination/Offloading at the load balancer level.
3. Security settings include
    1. Removing Nginx server signature.
    2. Security headers like X-Frame-Options, X-Content-Type-Options, XSS protection etc.
    3. Disabling of SSL v3 and enabling TLS v1.2 to mitigate POODLE vulnerability.
    4. Mitigate weak Diffie-Hellman keys.
4. File upload limits and timeouts to handle slow server.
5. Enabling gzip compression so that content travels in a compressed form over the network.
6. Handling of the DNS caching issue in case backend servers are using DNS names instead of IP addresses.
7. Allow/Deny certain IP for accessing the site
8. Sample redirection rules
9. Caching of static content
10. Settings of reverse proxy to backend server with marking servers as down if required.

```
server {
        # Redirecting 'non-www', 'http' traffic to 'www' and https'
        listen                  80;
        server_name             amazon.com;
        rewrite ^(.*)$ https://www.amazon.com$1 permanent;
}

server {
        listen                          443 ssl; # SSL Termination
        server_name                     www.amazon.com;

        # Disabling Nginx Version
        server_tokens                   off;

        # Additional Headers for Security
        add_header                      X-UA-Compatible "IE=Edge,chrome=1";
        add_header                      Set-Cookie "HttpOnly;Secure";
        add_header                      X-Frame-Options SAMEORIGIN;
        add_header                      X-Content-Type-Options nosniff;
        add_header                      X-XSS-Protection "1; mode=block";
        add_header                      Referrer-Policy "no-referrer-when-downgrade";
        add_header                      Feature-Policy "midi 'none'; microphone 'none'; camera 'none'; magnetometer 'none'; gyroscope 'none'";
#        add_header Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline' 'unsafe-eval' https://ssl.google-analytics.com https://assets.zendesk.com https://connect.facebook.net; img-src 'self' https://ssl.google-analytics.com https://s-static.ak.facebook.com https://assets.zendesk.com; style-src 'self' 'unsafe-inline' https://fonts.googleapis.com https://assets.zendesk.com; font-src 'self' https://themes.googleusercontent.com; frame-src https://assets.zendesk.com https://www.facebook.com https://s-static.ak.facebook.com https://tautt.zendesk.com; object-src 'none'";

        # Path of SSL Certificates
        ssl                             on;
        ssl_certificate                 /etc/nginx/conf.d/ssl/server.crt;
        ssl_certificate_key             /etc/nginx/conf.d/ssl/private.key;

        # Allowed Protocols and Cipher Suits
        ssl_protocols                   TLSv1 TLSv1.1 TLSv1.2;
        ssl_ciphers                     "EECDH+ECDSA+AESGCM EECDH+aRSA+AESGCM EECDH+ECDSA+SHA384 EECDH+ECDSA+SHA256 EECDH+aRSA+SHA384 EECDH+aRSA+SHA256 EECDH+aRSA+RC4 EECDH EDH+aRSA RC4 !aNULL !eNULL !LOW !3DES !MD5 !EXP !PSK !SRP !DSS !RC4";
        ssl_prefer_server_ciphers       on;
        ssl_dhparam                     /etc/nginx/conf.d/ssl/dhparam.pem;

        # File upload limits and time-outs
        client_max_body_size            6M;
        proxy_read_timeout              900;

        # Enabling Gzip for traffic compression
        gzip                            on;
        gzip_types                      text/plain application/xhtml+xml text/css application/xml application/xml+rss text/javascript application/javascript application/x-javascript;

        # Disables passing a request to the next server, incase of timeout(Optional, if required)
        proxy_next_upstream             off;
        
        # Adding resolvers(Google and custom) in case DNS Caching is an issue(Optional, if required)
#       resolver 8.8.8.8 8.8.4.4 220.226.204.100 valid=30s;
#       resolver_timeout 2s;

        # Adding Access Control Headers(Optional, if required)
        add_header                      Access-Control-Allow-Origin *;
        add_header                      Access-Control-Allow-Headers 'Content-Type';

        # Root location if Apache is in same machine as Nginx(Optional, if required)
        root                    /var/www/html/site;
        index                   index.php index.html index.htm default.html default.htm;

        # Allowing access to specific IPs only(Optional, if required)
#       allow                           210.212.195.98;                                                          
#       deny                            all;

        # Rewrite rule example(Optional, if required)
#       rewrite "^/aboutus" /about-us permanent;

        # Setting expiry of static contents
        location ~* \.(js|css|png|jpg|jpeg|gif|swf|xml|json|ttf)$ {
                expires 14d;
                access_log off;
                # Below statement not required if content is in Nginx server(Optional, if required)
                proxy_pass http://mycluster-content;
        }

        # Sending API requests to backend
        location ~*(\/LOCATION_URL) {
                proxy_pass http://mycluster-api;

                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header X-Forwarded-Proto $scheme;
        }
}

# Path of Upstream Servers
upstream mycluster-content {
        server 192.168.1.1:8080 weight=1;
        server 192.168.1.2:8080 weight=1;
        server 192.168.1.3:8080 down;
}

# Path of Upstream Servers
upstream mycluster-api {
        server 192.168.1.4:8080 weight=1;
        server 192.168.1.5:8080 weight=1;
        server 192.168.1.6:8080 down;       
}
```
