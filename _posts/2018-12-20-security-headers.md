---
layout: post
title: "Security Headers"
author: pratyush
categories: [ Security, Tutorial ]
#tags: [red, yellow]
image: assets/images/security-headers.webp
description: "How to secure website using proper http headers"
comments: true
#rating: 4.5
---

As the digital landscape evolves, the importance of website security cannot be overstated. Cyber threats and attacks have become more sophisticated, targeting websites with various malicious intents. However, adopting robust security measures can significantly reduce the risk of such attacks. One effective way to enhance the security of your website is by implementing HTTP security headers. In this blog post, we will explore the significance of various HTTP security headers, including X-Frame-Options, Content-Security-Policy, X-Content-Type-Options, Strict-Transport-Security, Referrer-Policy, and Permissions-Policy, and how they can protect your site from potential vulnerabilities.

### X-Frame-Options
The X-Frame-Options header is designed to defend against clickjacking attacks, where an attacker tricks users into clicking on a hidden element that executes unintended actions. By setting the X-Frame-Options header, web developers can control whether their site can be embedded within a frame or iframe on another domain. This prevents attackers from framing the website within their malicious pages, protecting your users from unknowingly engaging in harmful activities.

Example usage:
```
X-Frame-Options: DENY
```

### Content-Security-Policy (CSP)
Content-Security-Policy is a powerful security header that enables you to define the sources from which your website can load content, such as scripts, stylesheets, images, and more. By implementing CSP, you can mitigate the risks of cross-site scripting (XSS) attacks and data injection. With CSP, only approved sources are allowed to execute, making it significantly harder for attackers to inject and execute malicious scripts.

Example usage:
```
Content-Security-Policy: default-src 'self'; script-src 'self' www.google-analytics.com; style-src 'self' fonts.googleapis.com; img-src 'self' data:; object-src 'none'
```

### X-Content-Type-Options
X-Content-Type-Options is a simple yet effective header that prevents web browsers from interpreting files with MIME types other than what the server specifies. This guards against MIME-sniffing attacks, where attackers manipulate the content type of a file to exploit browser vulnerabilities. By setting this header, you ensure that browsers strictly adhere to the declared content type, reducing the risk of potential security loopholes.

Example usage:
```
X-Content-Type-Options: nosniff
```

### Strict-Transport-Security (HSTS)
Strict-Transport-Security is a crucial header for websites that enforce secure communication over HTTPS. By implementing HSTS, you instruct the user's browser to always access your website over an encrypted HTTPS connection, even if the user tries to access it via an insecure HTTP link. This helps thwart man-in-the-middle attacks and protects sensitive user data from interception.

Example usage:
```
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
```

### Referrer-Policy
The Referrer-Policy header allows you to control the amount of referrer information that is sent when a user clicks on a link to your site. Referrer information reveals the previous URL from which the user navigated to your site. With this header, you can choose to limit or even omit referrer information, which helps protect user privacy and prevents potentially sensitive data leakage.

Example usage:
```
Referrer-Policy: no-referrer-when-downgrade
```

### Permissions-Policy
Permissions-Policy is a relatively new header that allows you to specify the permissions required for various browser features and APIs, such as geolocation, camera access, microphone access, and more. By setting this header, you gain granular control over which resources can access these sensitive features, protecting your users from unauthorized access and potential privacy breaches.

Example usage:
```
Permissions-Policy: geolocation=(self "https://example.com"), camera=(), microphone=()
```

### Nginx config to enable all the above headers

```
server {
    listen 80;
    server_name your_domain.com;

    # Redirect all HTTP requests to HTTPS
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl http2;
    server_name your_domain.com;

    # SSL certificate and key path
    ssl_certificate /path/to/your_domain_com.crt;
    ssl_certificate_key /path/to/your_domain_com.key;

    # SSL configuration (adjust to your needs)
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers on;
    ssl_ciphers 'EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH';

    # Enable HSTS with a 1-year max-age and includeSubDomains
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload";

    # Prevent clickjacking attacks
    add_header X-Frame-Options "DENY";

    # Enable Content Security Policy
    add_header Content-Security-Policy "default-src 'self'; script-src 'self' www.google-analytics.com; style-src 'self' fonts.googleapis.com; img-src 'self' data:; object-src 'none'";

    # Prevent MIME-sniffing attacks
    add_header X-Content-Type-Options "nosniff";

    # Control the referrer information
    add_header Referrer-Policy "no-referrer-when-downgrade";

    # Set Permissions-Policy to control browser features and APIs
    add_header Permissions-Policy "geolocation=(self 'https://your_domain.com'), camera=(), microphone=()";

    # Example location block for serving your website files
    location / {
        root /path/to/your/website/files;
        index index.html;
    }
}
```

### Testing
In order to test HTTP Headers we will use Security Headers (https://securityheaders.com/). Security Headers is a part of Probely and was originally created by Scott Helme! It is a free and easy to use tool designed to help you better deploy and understand modern security features that are available for your website.

![Security Headers Test Report](/assets/images/security-headers-test-report.png)

### Additional Information

| Field                        | Description                                                                                             |
|------------------------------|---------------------------------------------------------------------------------------------------------|
| server                       | Server value has been changed. Typically you will see values like "Microsoft-IIS/8.0" or "nginx 1.7.2". |
| strict-transport-security    | HTTP Strict Transport Security is an excellent feature to support on your site and strengthens your implementation of TLS by getting the User Agent to enforce the use of HTTPS. |
| content-security-policy      | Content Security Policy is an effective measure to protect your site from XSS attacks. By whitelisting sources of approved content, you can prevent the browser from loading malicious assets. Analyse this policy in more detail. You can sign up for a free account on Report URI to collect reports about problems on your site. |
| x-frame-options              | X-Frame-Options tells the browser whether you want to allow your site to be framed or not. By preventing a browser from framing your site you can defend against attacks like clickjacking. |
| x-content-type-options       | X-Content-Type-Options stops a browser from trying to MIME-sniff the content type and forces it to stick with the declared content-type. The only valid value for this header is "X-Content-Type-Options: nosniff". |
| referrer-policy              | Referrer Policy is a new header that allows a site to control how much information the browser includes with navigations away from a document and should be set by all sites. |
| permissions-policy           | Permissions Policy is a new header that allows a site to control which features and APIs can be used in the browser. |
| cross-origin-embedder-policy | Cross-Origin Embedder Policy allows a site to prevent assets being loaded that do not grant permission to load them via CORS or CORP. |
| x-xss-protection             | X-XSS-Protection sets the configuration for the XSS Auditor built into older browsers. The recommended value was "X-XSS-Protection: 1; mode=block" but you should now look at Content Security Policy instead. |
| cross-origin-opener-policy   | Cross-Origin Opener Policy allows a site to opt-in to Cross-Origin Isolation in the browser. |
| cross-origin-resource-policy | Cross-Origin Resource Policy allows a resource owner to specify who can load the resource.              |