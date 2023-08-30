---
layout: post
title: "Redis High Availability"
author: pratyush
categories: [ Tuning, Tutorial ]
image: assets/images/redis-high-availability.webp
description: "Redis High Availability using Master-Slave Setup"
comments: true
---

When it comes to handling large datasets and ensuring high availability, Redis emerges as a prominent player in the world of in-memory data stores. One of the key features that make Redis a robust solution is its ability to set up a Master-Slave replication.

### Understanding Redis Master-Slave Replication
Redis Master-Slave replication is a data synchronization mechanism that enables data to be copied from a Redis server (Master) to one or more other servers (Slaves). The Master server acts as the primary node, handling write operations, while the Slave servers replicate the data from the Master and serve read requests. This setup enhances data redundancy, fault tolerance, and read scalability, making it an invaluable addition to a Redis deployment.

### The Benefits of Redis Master-Slave Setup

1. **Data Redundancy and High Availability:** With a Master-Slave configuration, your data is duplicated across multiple servers. In the event of a failure or outage of the Master node, one of the Slaves can be promoted as the new Master, ensuring continuous data availability and minimal disruption to your application.
2. **Improved Read Performance:** Since Slaves can handle read-only queries, they offload the read traffic from the Master. This leads to improved read performance and response times, allowing your application to efficiently scale and accommodate a higher number of concurrent users.
3. **Load Balancing:** By distributing read operations across multiple Slave nodes, you achieve load balancing, reducing the chances of bottlenecks during peak traffic times. This results in a smoother user experience and better overall performance.
4. **Backup and Disaster Recovery:** In case of data corruption or accidental deletion, Slave nodes can act as a reliable backup. They can be promoted to the Master role, ensuring seamless recovery and reducing the risk of data loss.
5. **Reduction in Master Overhead:** Writing data to the Master node can be resource-intensive. By offloading read operations to Slaves, the Master server's workload is reduced, allowing it to focus on processing write requests efficiently.
6. **Geographical Distribution and Latency Reduction:** Master-Slave replication supports asynchronous replication, making it possible to have Slave nodes in different geographical locations. This enables you to reduce latency for users accessing your application from distant regions.


### The Redis configuration file, usually named redis.conf, contains various settings that govern the behavior and performance of the Redis server. Below is a template of the Redis configuration file with explanations for each section:
```
# Redis Configuration File (version 5.0 and above)

# Server
bind 127.0.0.1             # IP address on which the Redis server will listen.
port 6379                  # Port number for Redis server to listen.

# General
daemonize yes                             # Run Redis server as a background daemon.
pidfile /var/run/redis/redis-server.pid   # Path to the PID file for the Redis process.
logfile /var/log/redis/redis-server.log   # Path to the log file for Redis server.

# Persistence
save 900 1                 # Save the database to disk if at least 1 key changes within 900 seconds (15 minutes).
save 300 10                # Save the database to disk if at least 10 keys change within 300 seconds (5 minutes).
save 60 10000              # Save the database to disk if at least 10000 keys change within 60 seconds (1 minute).

stop-writes-on-bgsave-error yes     # Stop accepting writes if a background save operation fails.
rdbcompression yes                  # Use compression when saving the RDB database to disk.
rdbchecksum yes                     # Add checksum to RDB file for error checking.
dir /var/lib/redis                  # Directory where RDB snapshots and AOF logs will be saved.

# Replication
# replicaof <master-ip> <master-port>    # Set this on Slave nodes to specify the Master IP and port for replication.
# masterauth <master-password>           # Authentication password to connect to the Master.
# replica-serve-stale-data yes           # Allow Slaves to serve data when they are out of sync with the Master.
# replica-read-only yes                  # Set Slave nodes to read-only mode.

# Security
requirepass <password>     # Set a password for Redis connections (optional but recommended).
rename-command CONFIG ""   # Disable the CONFIG command to enhance security.

# Memory Management
maxmemory 2gb              # Maximum memory Redis can use. You can set it according to your server's capacity.
maxmemory-policy volatile-lru  # Eviction policy when maxmemory is reached (other options: allkeys-lru, noeviction).

# Networking
tcp-backlog 511            # Set the TCP listen backlog for incoming connections.
timeout 0                  # Close the connection after a client is idle for specified seconds (0 disables).

# Advanced
activerehashing yes        # Enable background rehashing when Redis performs operations on large hash tables.
repl-backlog-size 1mb      # Size of the replication backlog buffer.
repl-backlog-ttl 3600      # Time-to-live (TTL) of the replication backlog buffer.

# Append Only File (AOF)
appendonly yes                      # Enable Append Only File (AOF) persistence.
appendfilename "appendonly.aof"     # Name of the AOF file.
appendfsync everysec                # Frequency of AOF fsync: every second (everysec) or always (always).

# Lua scripting
lua-time-limit 5000        # Maximum time in milliseconds that a Lua script can run.

# Security
protected-mode yes         # Enable protected mode to prevent Redis from accepting connections from external IPs.
```

### Master Server Configuration
> No change is required from the above config file.

### Slave Server Configuration
For the Slave server, you need to modify the configuration to make it a replica of the Master. Here are the changes to be made:

1. Uncomment the ```replicaof``` directive and specify the IP address and port of the Master server to which the Slave should replicate.
2. Set the ```replica-serve-stale-data``` directive to ```yes``` to allow the Slave to serve data when it is temporarily disconnected from the Master.
3. Set the ```replica-read-only directive``` to ```yes``` to make the Slave read-only.

```
replicaof 10.0.0.1 6379
masterauth <master-password>
replica-serve-stale-data yes
replica-read-only yes 
```

### Testing the configuration (Master)
> The below output requires the setup of Slave server and a service restart.

```
127.0.0.1:6379> info replication
```

```
# Replication
role:master
connected_slaves:1
slave0:ip=10.0.0.2,port=6379,state=online,offset=1737,lag=1
master_repl_offset:1737
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:2
repl_backlog_histlen:1736
```

```
127.0.0.1:6379> set test 'this key was defined on the master server'
```

### Testing replication (Slave)
```
127.0.0.1:6379> info replication
```

```
# Replication
role:slave
master_host:10.0.0.1
master_port:6379
master_link_status:up
master_last_io_seconds_ago:5
master_sync_in_progress:0
slave_repl_offset:1387
slave_priority:100
slave_read_only:1
connected_slaves:0
master_repl_offset:0
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0
```

```
127.0.0.1:6379> get test
```

```
"this key was defined on the master server"
```