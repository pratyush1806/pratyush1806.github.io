---
layout: post
title: "Distributed Transactions"
author: pratyush
categories: [ Tutorial ]
image: assets/images/distributed-transactions.webp
description: "Problem with Locking and the Solution of Distributed Locking"
comments: true
---

In concurrent and distributed systems, managing access to shared resources is a significant challenge. Traditional locking mechanisms, often used in single-threaded or single-process environments, fall short in distributed scenarios. This blog post delves into the problems with traditional locking and how distributed locking, particularly using Redis, addresses these issues.

### The Problem with Traditional Locking
Traditional locking mechanisms work well within a single process or on a single machine. They use constructs like mutexes, semaphores or monitors to ensure that only one thread or process can access a critical section of code at a time. However, when applied to distributed systems, these mechanisms encounter several challenges.

1. Lack of Global Visibility
Traditional locks are local to the process or machine. In a distributed system, different instances of an application might be running on multiple machines, and these instances are not aware of each other's locks.

2. Network Latency and Failures
In a distributed environment, communication between nodes occurs over a network, which introduces latency and potential failures. This can lead to scenarios where a lock appears to be held indefinitely due to network partitioning.

3. Distributed Coordination
Coordinating locks across multiple nodes requires a consensus mechanism to ensure consistency, which is complex and difficult to implement correctly.

4. Single Point of Failure
If the process or machine holding the lock fails, the lock might not be released, leading to deadlocks or resource starvation.

### The Solution: Distributed Locking
Distributed locking addresses these issues by providing a mechanism for managing locks across multiple nodes in a distributed system. Let's revisit the implementation of distributed locking using Redis in a Spring Boot application, focusing on a practical example of managing credit and debit transactions.

1. Project setup
Here's the `pom.xml` for the Spring Boot project with Redis for distributed locking

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.4.9</version>
		<relativePath /> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.pratyush</groupId>
	<artifactId>spring-boot-redis-distributed-locking</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>spring-boot-redis-distributed-locking</name>
	<description>Spring Boot project for Distributed Locking</description>
	<properties>
		<java.version>11</java.version>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<maven-jar-plugin.version>3.1.1</maven-jar-plugin.version>
	</properties>
	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
		<dependency>
			<groupId>org.projectlombok</groupId>
			<artifactId>lombok</artifactId>
			<optional>true</optional>
		</dependency>
		<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <scope>runtime</scope>
        </dependency>
	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>

</project>
```

2. Configuring Redis
Add the necessary Redis configuration to your `application.properties`

```
# Redis Configuration
spring.redis.host=localhost
spring.redis.port=6379

# Spring Data Source Details
spring.datasource.url=jdbc:h2:file:./testdb
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=
```

3. Distributed Lock Service
Create a service to manage distributed locks

```
import java.util.concurrent.TimeUnit;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.stereotype.Service;

@Service
public class DistributedLockService {
    private static final long LOCK_EXPIRY_TIME = 10000; // 10 seconds

    @Autowired
    private StringRedisTemplate redisTemplate;

    public boolean acquireLock(String key) {
        Boolean success = redisTemplate.opsForValue().setIfAbsent(key, "LOCKED", LOCK_EXPIRY_TIME, TimeUnit.MILLISECONDS);
        return success != null && success;
    }

    public void releaseLock(String key) {
        redisTemplate.delete(key);
    }
}
```

4. Account Entity and Repository
Define the Account entity and repository

```
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;

import javax.persistence.Id;

import lombok.Data;

@Data
@Entity
public class Account {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private int id;
    private double balance;
}
```

```
import org.springframework.data.repository.CrudRepository;
import org.springframework.stereotype.Repository;

import com.pratyush.redis.locking.entity.Account;

@Repository
public interface AccountRepository extends CrudRepository<Account, Integer> {
	
}
```

5. Transaction Service
Implement the transaction service with distributed locking

```
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import com.pratyush.redis.locking.entity.Account;
import com.pratyush.redis.locking.repository.AccountRepository;

@Service
public class TransactionService {
	@Autowired
    private AccountRepository accountRepository;

    @Autowired
    private DistributedLockService lockService;

    @Transactional
    public boolean credit(int accountId, double amount) {
    	String lockKey = "account:" + accountId;

        if (!lockService.acquireLock(lockKey)) {
            return false; // Failed to acquire lock
        }

        try {
            Account account = accountRepository.findById(accountId).orElseThrow(() -> new RuntimeException("Account not found"));
            account.setBalance(account.getBalance() + amount);
            accountRepository.save(account);
            return true;
        } finally {
            lockService.releaseLock(lockKey);
        }
    }
    
    @Transactional
    public boolean debit(int accountId, double amount) {
    	String lockKey = "account:" + accountId;

        if (!lockService.acquireLock(lockKey)) {
            return false; // Failed to acquire lock
        }

        try {
            Account account = accountRepository.findById(accountId).orElseThrow(() -> new RuntimeException("Account not found"));
            if (account.getBalance() < amount) {
                return false; // Insufficient funds
            }
            account.setBalance(account.getBalance() - amount);
            accountRepository.save(account);
            return true;
        } finally {
            lockService.releaseLock(lockKey);
        }
    }
}
```

6. Controller
Create a controller to handle HTTP requests for credit and debit operations

```
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

import com.pratyush.redis.locking.service.TransactionService;

@RestController
@RequestMapping("/api/account")
public class AccountController {
	@Autowired
    private TransactionService transactionService;
	
	@PostMapping("/credit")
    public String credit(@RequestParam int accountId, @RequestParam double amount) {
        boolean success = transactionService.credit(accountId, amount);
        return success ? "Credit successful" : "Credit failed";
    }

    @PostMapping("/debit")
    public String debit(@RequestParam int accountId, @RequestParam double amount) {
        boolean success = transactionService.debit(accountId, amount);
        return success ? "Debit successful" : "Debit failed";
    }
}
```

> By leveraging Redis and its atomic operations, we can implement a reliable and high-performance distributed lock mechanism in Spring Boot applications. This approach ensures data consistency and prevents race conditions, making it ideal for scenarios like managing financial transactions.