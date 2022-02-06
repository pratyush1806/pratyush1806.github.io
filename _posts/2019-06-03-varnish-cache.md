---
layout: post
title: "What is Varnish Cache"
author: pratyush
categories: [ Tutorial ]
image: assets/images/varnish-cache.webp
comments: true
---

Varnish Cache is a web application accelerator. It can be installed it in front of any web server like Apache and Nginx to speeds up delivery by a factor upto 1000x.

### What is Varnish Cache
Varnish is a reverse HTTP proxy, sometimes referred to as an HTTP accelerator or a web accelerator. A reverse proxy is a proxy server that appears to clients as an ordinary server. Varnish stores (caches) files or fragments of files in memory that are used to reduce the response time and network bandwidth consumption on future, equivalent requests. Varnish is designed for modern hardware, modern operating systems and modern work loads. Varnish is more than a reverse HTTP proxy that caches content to speed up your server. Depending on the installation, Varnish can also be used as:

1. web application firewall, 
2. DDoS attack defender, 
3. hotlinking protector,
4. load balancer,
5. integration point,
6. single sign-on gateway,
7. authentication and authorization policy mechanism, 
8. quick fix for unstable backends, and
9. HTTP router.

<p><iframe width="560" height="315" src="https://www.youtube.com/embed/fGD14ChpcL4" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></p>

#### Varnish Architecture
Varnish handles all incoming requests before they land to your backend server. If the request is not cached, Varnish will forward the request to the web server’s backend and cache the result. The cached requests are then stored in the memory. Any subsequent request is then served from memory till TTL (Time To Live) expires. Varnish does not cache POST request, Ajax request or any request having Logged-In cookies.

![Varnish Architecture](/assets/images/varnish-architecture.jpg)

> Varnish Makes caching decision based on http headers and VCL rules.

### Varnish Flow Diagram

![Varnish Architecture](/assets/images/varnish-flow.png)

### 1. vcl_recv()
 1. First subroutine executed after varnish parsed the client request.
 2. Normalize client input.
 3. Backend selection.
 4. User agent detection.
 5. Manipulating client request.
 6. Deciding cache policy

### 2. vcl_backend_response()
 1. This subroutine is executed after the response headers are successfully received from backend.
 2. This block decides whether to cache the response based on cache headers.
 3. This section also sets the TTL for the cache content.

### 3. vcl_deliver()
 1. Last subroutine which triggers before the content is delivered to the user.
 2. Commonly used to add or remove debug headers.

#### Sample Request
```
Request URL: https://www.xyz.com/
Request Method: GET
Status Code: 200 
Remote Address: 142.250.194.100:443
Referrer Policy: strict-origin-when-cross-origin
```
#### Sample Response (with x-cache headers)
```
accept-language: bytes
cache-control: max-age=1200, public, must-revalidate
content-encoding: gzip
content-length: 36678
content-type: text/html; charset=utf-8
date: Fri, 04 Feb 2019 17:08:57 GMT
server: nginx
vary: X-Device, Accept-Encoding
x-cache: HIT
x-cache-hits: 1
```

### Minimal Configuration File
```
# Setting VCL Version
vcl 4.0;

backend default {
	.host = "127.0.0.1";
	.port = "8080";
}

sub vcl_recv {
	# Remove cookies you don't need
	# Remove request parameters you don't need
	# Device detection code
	# Hostname detection code for multiple domains
	# By-Pass requests based on URL
}

# This function is used when a request is sent by our backend (Apache/Nginx server)
sub vcl_backend_response {
	# Clean response header, remove unwanted set-cookies
	# Do not cache error pages like 301, 404, 500 etc

	# Setting a cache time of 20 minutes and a grace period of 1 hour
	# Grace period will deliver stale response if backend server is down
	set beresp.ttl = 20m;
	set beresp.grace = 1h;
}

sub vcl_deliver {
	# Adding cache hits/miss headers
	if (obj.hits > 0) {
		set resp.http.X-Cache = "HIT";
		set resp.http.X-Cache-Hits = obj.hits;
	} else {
		set resp.http.x-Cache = "MISS";
	}
	unset resp.http.X-Powered-By;
	unset resp.http.Age;
}
```