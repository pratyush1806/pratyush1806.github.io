---
layout: post
title: "Perfomance tuning of Apache Server"
author: pratyush
categories: [ Tuning, Tutorial ]
image: assets/images/tuning-apache.webp
comments: true
---

Apache uses one of following MPM (Multi-Processing Module) for handling incoming requests and processes them. They are Prefork, Worker and Event.

### 1. Prefork
Prefork MPM launches multiple child processes. Each child process handle one connection at a time.

Prefork uses high memory in comparison to worker MPM. Prefork is the default MPM used by Apache2 server. Preform MPM always runs few minimum (MinSpareServers) defined processes as spare, so new requests do not need to wait for new process to start.

![Prefork](/assets/images/apache/prefork.jpg)

```
<IfModule mpm_prefork_module> 
    StartServers 5 
    MinSpareServers 5 
    MaxSpareServers 10 
    MaxRequestWorkers 250 
    MaxConnectionsPerChild 1000 
</IfModule>
```
### 2. Worker
Worker MPM generates multiple child processes similar to prefork. Each child process runs many threads. Each thread handles one connection at a time.

In short Worker MPM implements a hybrid multi-process multi-threaded server. Worker MPM uses low memory in comparison to Prefork MPM.

![Worker](/assets/images/apache/worker.jpg)

```
<IfModule mpm_worker_module> 
    StartServers 3 
    ServerLimit 16 
    MinSpareThreads 75 
    MaxSpareThreads 250 
    ThreadsPerChild 25 
    MaxRequestWorkers 400 
    MaxConnectionsPerChild 1000 
</IfModule>
```

### 3. Event
Event is similar to Worker MPM but it designed for managing high loads.

This MPM allows more requests to be served simultaneously by passing off some processing work to supporting threads. Using this MPM Apache tries to fix the ‘keep alive problem’ faced by other MPM. When a client completes the first request then the client can keep the connection open, and send further requests using the same socket, which reduces connection overload.

```
<IfModule mpm_event_module> 
    StartServers 3 
    ServerLimit 16 
    MinSpareThreads 75 
    MaxSpareThreads 250 
    ThreadsPerChild 25 
    MaxRequestWorkers 400 
    MaxConnectionsPerChild 1000 
</IfModule>
```