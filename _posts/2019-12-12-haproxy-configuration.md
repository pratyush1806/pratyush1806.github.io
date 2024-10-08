---
layout: post
title: "HAProxy setup and configuration"
author: pratyush
categories: [ Tutorial ]
image: assets/images/haproxy.webp
description: "HAProxy: An Overview with Configuration"
comments: true
---

HAProxy (High Availability Proxy) is a powerful open-source solution for load balancing and high availability in networked applications. It is widely used to distribute traffic among multiple servers and ensure availability and reliability. HAProxy can handle millions of requests per second and supports both TCP (Layer 4) and HTTP (Layer 7) load balancing.

### Key Features of HAProxy
**Load Balancing:** HAProxy can distribute incoming traffic to multiple servers, allowing for improved performance and failover.

**High Availability:** HAProxy helps create a highly available system by routing traffic to healthy servers and bypassing those that are down.

**Protocol Support:** It supports both TCP (Layer 4) and HTTP (Layer 7) traffic balancing.

**SSL Termination:** HAProxy can handle SSL termination, which reduces the load on your backend servers.

**Health Checks:** It can monitor backend servers' health and stop sending traffic to unresponsive ones.
Access Control Lists (ACLs): HAProxy can route requests based on various criteria like URL paths, headers, or client IPs.

**Stats Interface:** HAProxy comes with a built-in statistics interface for real-time monitoring.

### Installation
```
# install
yum install haproxy

# to restart HAProxy on reboot
chkconfig haproxy on

# necessary firewall related changes to allow ports
firewall-cmd --permanent --zone=public --add-service=http
firewall-cmd --permanent --zone=public --add-service=https

firewall-cmd --reload

# start the service
systemctl start haproxy
```

### Basic HAProxy Configuration File
HAProxy uses a single configuration file, typically located at `/etc/haproxy/haproxy.cfg`. The configuration file consists of several sections including `global`, `defaults`, `frontend`, `backend`, and `listen`.

### Example Configuration File
```
global
	maxconn 50000
	stats socket /etc/haproxy/haproxy.sock level admin
	ssl-default-bind-options no-sslv3 no-tlsv10 no-tlsv11
	ssl-default-bind-ciphers EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH

defaults
	log global
	mode http
	timeout connect 5s
	timeout client 50s
	timeout server 50s

frontend http-in
	bind *:80

	redirect scheme https if !{ ssl_fc }  # Redirect all HTTP to HTTPS
	acl is_static path_beg -i /static /images
	use_backend static_server if is_static

	default_backend app_server

frontend https-in
	bind *:443 ssl crt /etc/ssl/private/certificate.pem alpn h2,http/1.1
	acl is_static path_beg -i /static /images
	use_backend static_server if is_static

	default_backend app_server

backend static_server
	balance roundrobin
	server static-server-1 192.168.1.10:80 check
	server static-server-2 192.168.1.11:80 check

backend app_server
	balance roundrobin
	server app-server-1 192.168.1.12:8001 check
	server app-server-2 191.168.1.13:8002 check

listen stats
	bind *:8080
	stats enable
	stats uri /haproxy?stats
	stats realm Haproxy\ Statistics
	stats auth admin:admin
	stats admin if TRUE
```

### Breakdown of the configuration
1. Global Section
	* `maxconn` is to set Maximum connected clients.
	* `stats socket` is configured for administrative purposes.
	* `ssl-default-bind-options no-sslv3 no-tlsv10 no-tlsv11`: Disables outdated SSL/TLS versions.
	* `ssl-default-bind-ciphers`: Defines a set of strong ciphers to use. You can adjust this for stronger security.
2. Defaults Section
	* Mode is set to HTTP mode, meaning HAProxy will process HTTP-specific traffic.
	* Logging is set to global logging with HTTP logs enabled.
	* Timeouts define limits on how long to wait for connections to the client and server.
3. Frontend Section
	* `bind`: Specifies which IP and port HAProxy listens to. In this case, it listens on port 80 for all IPs.
	* `acl`: Defines Access Control Lists (ACLs) to filter traffic based on conditions. Here, the `is_static` ACL checks if the request URL starts with `/static` or `/images`.
	* `use_backend`: This directive routes the request to different backends based on the ACLs.
		* Requests to `/static` or `/images` go to the `static_server` backend.
		* All other traffic goes to the `app_server`.
	* `redirect scheme`: Ensures that users accessing the site over http are redirected to https.
	* `ssl`: This keyword enables SSL termination/SSL offloading.
	* `crt`: This directive points to the combined `.pem` file that contains the certificate and private key.
	* `alpn h2,http/1.1`: Enables support for HTTP/2 and HTTP/1.1 protocols
4. Backend Sections
	The backend sections define where the traffic should be forwarded based on the conditions defined in the frontend. Each backend uses a load balancing algorithm (here, `roundrobin`) to distribute traffic between the listed servers.
	* `static_server`: Handles static content by forwarding traffic to `192.168.1.10` or `192.168.1.11`.
	* `app_server`: Handles application traffic, forwarding it to `192.168.1.12` or `192.168.1.13`.
	Each backend also includes a `check` directive to perform health checks on the servers, ensuring that only healthy servers are used.
5. Stats Section
	By navigating to http://<haproxy_ip>:8080/haproxy?stats, you'll be prompted for a username and password, after which you’ll be able to see detailed statistics and manage HAProxy through the interface.
	* `bind`: Specifies the IP and port (here `* :8080`) where the statistics page is available.
	* `stats enable`: Turns on the statistics page.
	* `stats uri`: Specifies the URL endpoint for accessing the stats page (`/haproxy?stats`).
	* `stats auth`: Requires basic authentication to access the stats page (in this case, username: `admin`, password: `password`).
	* `stats admin`: Grants administrative privileges on the stats page.

> In the above example I have considered CentOS 7 with HAProxy version 1.5.18