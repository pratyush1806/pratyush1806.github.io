---
layout: post
title: "CORS Policy"
author: pratyush
categories: [ Tutorial ]
image: assets/images/cors-policy.webp
description: "Understanding CORS Policy"
comments: true
---

### Introduction to CORS
Cross-Origin Resource Sharing (CORS) is a critical security mechanism implemented by web browsers to control how web pages in one domain can make requests to another domain. As web applications become increasingly complex and distributed, understanding CORS is essential for developers to create secure and functional web services.

### What is CORS?
CORS is a browser security feature that restricts web pages from making requests to a different domain than the one serving the web page. This mechanism prevents malicious websites from making unauthorized requests to your API or server, protecting users from potential cross-site scripting (XSS) and other security vulnerabilities.

### Why is CORS Necessary?
By default, web browsers follow the Same-Origin Policy, which prevents web pages from making requests to a different domain. While this policy enhances security, it also limits the functionality of modern web applications that often need to interact with multiple services and APIs.
CORS provides a controlled way to relax these restrictions, allowing servers to specify:

* Which origins can access their resources
* What HTTP methods are permitted
* What headers can be included in cross-origin requests

![CORS Policy](https://docs.aws.amazon.com/images/sdk-for-javascript/v2/developer-guide/images/cors-overview.png)

### The Role of Preflight Requests in CORS
A preflight request is a special type of HTTP `OPTIONS` request that browsers automatically send before making a complex cross-origin HTTP request. It acts as a preliminary "permission check" to determine whether the actual request is safe to send.

#### How Preflight Requests Work
1. **Initial Check:** The browser sends an `OPTIONS` request to the target server with two key headers `Access-Control-Request-Method` and `Access-Control-Request-Headers`.

2. **Server Response:** The server responds with CORS headers indicating:
    1. Allowed origins
    2. Permitted HTTP methods
    3. Allowed headers
    4. Caching duration for preflight results

```
# Preflight Request
OPTIONS /api/data HTTP/1.1

Host: api.example.com
Origin: https://webapp.com
Access-Control-Request-Method: PUT
Access-Control-Request-Headers: Content-Type, Authorization

# Server Response
HTTP/1.1 204 No Content
Access-Control-Allow-Origin: https://webapp.com
Access-Control-Allow-Methods: PUT, GET, POST
Access-Control-Allow-Headers: Content-Type, Authorization
Access-Control-Max-Age: 86400
```

### CORS Headers: A Detailed Breakdown
1. Access-Control-Allow-Origin
    * Specifies which origins are permitted to access the resource
    * Can be a specific domain or * for all origins (not recommended for production)
```
# Allow a specific domain to access the resource
Access-Control-Allow-Origin: yourdomain.com
# Or for all origins
Access-Control-Allow-Origin: *
```

2. Access-Control-Allow-Methods
    * Defines the HTTP methods allowed when accessing the resource
    * Crucial for controlling the types of requests permitted
```
# Defines which methods are allowed to access
Access-Control-Allow-Methods: GET, POST, PUT, DELETE, OPTIONS
```

3. Access-Control-Allow-Headers
    * Specifies which HTTP headers can be used in the actual request
    * Important for custom headers or specific authentication mechanisms
```
Access-Control-Allow-Headers: Content-Type, Authorization, X-Requested-With
```

4. Access-Control-Allow-Credentials
    * Indicates whether the response to the request can be exposed when the credentials flag is true
    * Required when sending cookies or authentication headers in cross-origin requests
```
# Required to pass authentication headers for secured resources
Access-Control-Allow-Credentials: true
```

5. Access-Control-Expose-Headers
    * Lists headers that the browser is allowed to access from the response
```
Access-Control-Expose-Headers: X-Custom-Header, Content-Length
```

6. Access-Control-Max-Age
    * Specifies how long the results of a preflight request can be cached
    * Reduces the number of preflight requests for the same resource
```
Access-Control-Max-Age: 86400
```

### Full Nginx Configuration for CORS setup
```
server {
    listen 80;
    server_name example.com;

    # Enable CORS for all locations
    location / {
        # Preflight requests handling
        if ($request_method = 'OPTIONS') {
            add_header 'Access-Control-Allow-Origin' '*';
            add_header 'Access-Control-Allow-Methods' 'GET, PUT, POST, DELETE, OPTIONS';
            add_header 'Access-Control-Allow-Headers' 'Content-Type, Authorization, X-Requested-With';
            add_header 'Access-Control-Max-Age' 86400;
            return 204;
        }

        # Regular CORS headers
        add_header 'Access-Control-Allow-Origin' '*';
        add_header 'Access-Control-Allow-Methods' 'GET, PUT, POST, DELETE, OPTIONS';
        add_header 'Access-Control-Allow-Headers' 'Content-Type, Authorization, X-Requested-With';
    }
}
```

### Best Practices
1. Always specify exact origins instead of using `*`
2. Use the least permissive CORS configuration possible
3. Implement proper authentication and authorization
4. Use HTTPS to encrypt cross-origin communications
5. Regularly audit and update CORS policies