---
layout: post
title: "Achieving High Availability using Keepalived"
author: pratyush
categories: [ Tutorial ]
#tags: [red, yellow]
image: assets/images/keepalived-architecture.webp
description: "Achieving High Availability using Keepalived"
comments: true
#rating: 4.5
---

In today's fast-paced and competitive digital landscape, high availability and fault tolerance have become critical requirements for businesses and organizations. They need their online services to be continuously accessible, even in the face of hardware failures or other disruptions. This is where Keepalived comes into the picture. Keepalived is an open-source software that ensures high availability by providing failover capabilities for network services like Nginx load balancers. In this blog post, we will explore the use cases of Keepalived and demonstrate how to configure it for achieving high availability for an Nginx load balancer.

## What is Keepalived and its Use Cases?
Keepalived is a simple yet powerful software that helps maintain high availability and load balancing in a network environment. It works by assigning a virtual IP address (VIP) to multiple servers, which together form a highly available group. The virtual IP acts as a floating IP, moving between the servers based on their health and availability.

## The primary use cases of Keepalived include:
1. Load Balancing: Keepalived works in conjunction with load balancers like Nginx to distribute incoming network traffic across multiple backend servers. This ensures even distribution of the workload, prevents overloading a single server, and improves overall system performance.
2. High Availability: Keepalived provides failover capabilities to ensure that network services remain available even if one or more servers in the group fail. When a server goes down, the virtual IP is automatically migrated to another available server, minimizing downtime.
3. Redundancy: Keepalived allows you to set up redundant servers, which continuously monitor each other's health. If one server fails, the remaining servers take over the responsibilities to maintain seamless service delivery.
4. Load Balancer Scalability: By utilizing Keepalived, you can easily scale your load balancer infrastructure by adding or removing backend servers without interrupting the service.

## Below is a sample Keepalived configuration for achieving high availability for an Nginx load balancer setup. This example assumes you have two servers (Server A and Server B) in the group and one virtual IP address.

## 1. Install keepalived and start it
```
yum install keepalived -y
systemctl enable keepalived
systemctl start keepalived
```

## 2. Update kernel to support two IP addresses
```
echo net.ipv4.ip_nonlocal_bind=1 >> /etc/sysctl.conf
sysctl -p                   # persist the change without rebooting the machine
```

## 3. Configure Keepalived on Server A
```
# /etc/keepalived/keepalived.conf

vrrp_script chk_nginx {
	script "pidof nginx"    # Check for pid of Nginx
	interval 2
}

vrrp_instance VI_1 {        # Defines an individual instance of the VRRP protocol running on an interface
	interface eth0          # Network interface id
	state MASTER
	virtual_router_id 51    
	priority 100            # Set a higher priority for the master server
	
    virtual_ipaddress {
		10.0.0.101          # Your virtual IP address
	}
	
    track_script {
		chk_nginx
	}
}
```

## 4. Configure Keepalived on Server B
```
# /etc/keepalived/keepalived.conf

vrrp_script chk_nginx {
	script "pidof nginx"    # Check for pid of Nginx
	interval 2
}

vrrp_instance VI_1 {        # Defines an individual instance of the VRRP protocol running on an interface
	interface eth0          # Network interface id
	state BACKUP
	virtual_router_id 51
	priority 90             # Set a lower priority for the backup server
	
    virtual_ipaddress {
		10.0.0.101          # Your virtual IP address
	}
	
    track_script {
		chk_nginx
	}
}
```

## 5. Verify VRRP
### Perform a ping to the VRRP VIP

> ping 192.168.1.2

### On the master server check IP address list to see whether the VIP is active on the master host

> ip addr list

## Once you’ve confirmed that Keepalived has started on both servers and Server A is the active master, you can test out failover functionality by “flipping” the VIP to the other server. By stopping Keepalived on Server A, the active master stops sending out advertisements and Server B takes over the VIP.