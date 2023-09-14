---
layout: post
title: "Wildfly Clustering"
author: pratyush
categories: [ Tutorial ]
image: assets/images/wildfly-cluster.webp
description: "Setting up a 2 node Wildfly cluster for a Java Web Application"
comments: true
---

### Understanding Wildfly Session Clustering
Wildfly, formerly known as JBoss Application Server, is an open-source, lightweight, and high-performance application server that supports the Java EE (Enterprise Edition) platform. Wildfly session clustering enables the distribution and replication of user sessions across multiple instances of the application server, providing a robust solution for load balancing and fault tolerance.

### Step 1: Prepare the Java Code
1. Create a Maven Web Application with Archtype `maven-archetype-webapp`
2. Create a Servlet to save HTTP Sessions
3. Create a JSP to show session variables
4. Create a `web.xml` and use `<distributable/>` tag for session replication

### Step 2: Nginx Configuration
If you prefer Nginx as a reverse proxy, follow these steps
```
server {
    listen 80;
    server_name localhost;

    location / {
        proxy_pass http://wildfly-cluster;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}

upstream wildfly-cluster {
    server 127.0.0.1:8180;
    server 127.0.0.1:8280;
}
```

### Step 3: Start Wildfly in Standalone Cluster Mode
To start Wildfly in Standalone cluster mode, use the following command to run 2 nodes cluster in the same machine.
```
./standalone.sh -Djboss.node.name=node-01 -c standalone-ha.xml -b 0.0.0.0 -Djboss.socket.binding.port-offset=100
./standalone.sh -Djboss.node.name=node-02 -c standalone-ha.xml -b 0.0.0.0 -Djboss.socket.binding.port-offset=200
```

The `-c` flag specifies the configuration file (standalone-full-ha.xml for clustering).

The `-b` flag sets the server bind address to your server's IP.

The `-Djboss.node.name=node-01` is used to assign an identifier/name to the cluster member.

The `-Djboss.socket.binding.port-offset` is used to run multiple copies of server on the same machine.

### URLs to access wildfly
```
# node-01
http://localhost:8180

# node-02
http://localhost:8180

# management console 01
http://localhost:10090/management

# management console 02
http://localhost:10190/management
```

Link of the complete [source code](https://github.com/pratyush1806/cluster-demo).

With Wildfly's session clustering mechanism, you can ensure fault tolerance and load balancing across multiple instances of the application server. You can also perform application deployment without downtime and that too without effecting user sessions.