---
layout: post
title: "RabbitMQ Clustering"
author: pratyush
categories: [ Tutorial ]
image: assets/images/rabbitmq-clustering.webp
description: "RabbitMQ Clustering with Docker Compose"
comments: true
---

RabbitMQ is a robust messaging broker that facilitates the exchange of messages between producers and consumers, ensuring that messages are safely stored until they are delivered. However, when you scale your applications, a single RabbitMQ node might not be enough to handle the load or ensure high availability. This is where RabbitMQ clustering comes in.

In this blog post, we'll dive deep into the concept of RabbitMQ clustering, why it's important, and how you can set up a RabbitMQ cluster using Docker Compose. We'll also explain how to configure queue mirroring for high availability and discuss the Docker Compose setup in detail.

### What is RabbitMQ Clustering?
RabbitMQ clustering is a method of linking multiple RabbitMQ nodes to work together as a single, logical broker. When nodes are clustered, they share metadata, queues, and exchanges, allowing the cluster to function as a unified system. Clustering provides several benefits:

- **High Availability:** If one node fails, other nodes in the cluster can continue processing messages.
- **Load Balancing:** Workloads can be distributed across multiple nodes to prevent any single node from becoming a bottleneck.
- **Scalability:** You can add more nodes to the cluster to handle increased traffic.

### RabbitMQ Clustering Architecture
In a RabbitMQ cluster, each node has its own copy of metadata and is aware of other nodes in the cluster. The cluster can be set up in two modes:

- **Disc Nodes:** These nodes store all their data on disk. They are resilient to node restarts, as the data persists.
- **RAM Nodes:** These nodes store most of their data in memory for faster access. However, they rely on disc nodes for persistence.
In most cases, at least one disc node is required in a cluster to ensure data durability.

### Queue Mirroring for High Availability
To ensure that queues are available even if a node fails, RabbitMQ supports queue mirroring. In a mirrored queue setup, each queue has a master node and one or more mirrors (replicas). The master is responsible for coordinating the queue operations, while the mirrors replicate the state of the queue.

If the master node fails, one of the mirrors is automatically promoted to become the new master. This feature is crucial for systems that require high availability.

### Setting Up RabbitMQ Clustering with Docker Compose

#### Step 1: Docker Compose Setup
Here’s the complete `docker-compose.yml` file:
```
version: '3'

services:
  rabbit1:
    image: rabbitmq:3-management
    hostname: rabbit1
    container_name: rabbit1
    ports:
      - "15672:15672"
      - "5672:5672"
    volumes:
      - "./rabbitmq.conf:/etc/rabbitmq/rabbitmq.conf"
      - "./.erlang.cookie:/var/lib/rabbitmq/.erlang.cookie"
#    environment:
#      - RABBITMQ_SERVER_ADDITIONAL_ERL_ARGS=-rabbitmq_cluster_policy "ha-all" '{"ha-mode":"all"}'
    networks:
      - rabbitmq_network

  rabbit2:
    image: rabbitmq:3-management
    hostname: rabbit2
    container_name: rabbit2
    ports:
      - "15673:15672"
      - "5673:5672"
    volumes:
      - "./rabbitmq.conf:/etc/rabbitmq/rabbitmq.conf"
      - "./.erlang.cookie:/var/lib/rabbitmq/.erlang.cookie"
#    environment:
#      - RABBITMQ_SERVER_ADDITIONAL_ERL_ARGS=-rabbitmq_cluster_policy "ha-all" '{"ha-mode":"all"}'
    networks:
      - rabbitmq_network

  rabbit3:
    image: rabbitmq:3-management
    hostname: rabbit3
    container_name: rabbit3
    ports:
      - "15674:15672"
      - "5674:5672"
    volumes:
      - "./rabbitmq.conf:/etc/rabbitmq/rabbitmq.conf"
      - "./.erlang.cookie:/var/lib/rabbitmq/.erlang.cookie"
#    environment:
#      - RABBITMQ_SERVER_ADDITIONAL_ERL_ARGS=-rabbitmq_cluster_policy "ha-all" '{"ha-mode":"all"}'
    networks:
      - rabbitmq_network

networks:
  rabbitmq_network:
    driver: bridge
```

#### Step 2: RabbitMQ Configuration
Here’s the `rabbitmq.conf` file
```
cluster_formation.peer_discovery_backend  = classic_config
cluster_formation.classic_config.nodes.1  = rabbit@rabbit1
cluster_formation.classic_config.nodes.2  = rabbit@rabbit2
cluster_formation.classic_config.nodes.3  = rabbit@rabbit3

# Enable queue mirroring to all nodes
queue_master_locator = min-masters
cluster_partition_handling = autoheal

# Logs
log.file.level = info
```

#### Step 3: RabbitMQ Configuration
Here’s the `.erlang.cookie` file
```
RABBITMQ_ERLANG_COOKIE
```

#### Step 4: Start the cluster
With everything configured, you can start the RabbitMQ cluster by running:
```
# start the cluster
docker-compose up -d

# check container logs
docker logs rabbit1
docker logs rabbit2
docker logs rabbit3

# shutdown the cluster
docker-compose down
```

#### Step 5: Verify the Cluster
You can verify the RabbitMQ cluster by accessing the Management UI for each node:

[http://localhost:15672](http://localhost:15672) (for rabbit1)

[http://localhost:15673](http://localhost:15673) (for rabbit2)

[http://localhost:15674](http://localhost:15674) (for rabbit3)

Log in using the default credentials `(guest/guest)` and navigate to the "Cluster" tab. You should see all three nodes listed, indicating that the cluster is up and running.

### Queue Mirroring Policy
The queue mirroring policy we configured ensures that all queues are mirrored across all nodes in the cluster. This is crucial for high availability, as it allows any node to take over queue operations if the master node fails.

> Cluster policy is not working currently, it will be covered in the future.

### Conclusion
RabbitMQ clustering is an essential feature for building resilient, high-availability messaging systems. By using Docker Compose, we can easily set up a multi-node RabbitMQ cluster with minimal configuration. This setup ensures that your queues are mirrored across nodes, providing fault tolerance and load balancing.
