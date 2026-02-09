---
layout: post
title: "API Performance Optimisation Best Practices"
author: pratyush
categories: [ Tutorial ]
image: assets/images/api-performance-optimisation.webp
description: "API Performance Optimisation Practices"
comments: true
---

### 1. Implement effective caching: Utilize server-side and client-side caching (e.g., Redis) and leverage HTTP cache headers.
This Spring Boot code demonstrates effective caching by combining server-side caching using Redis and client-side caching using HTTP cache headers.

At the server side, Spring’s caching abstraction is enabled using `@EnableCaching`. The `@Cacheable` annotation in the service layer stores the result of a method call in Redis. When a request for the same data is made again, Spring first checks Redis and returns the cached value instead of executing the method (e.g., avoiding a database call). A `TTL` (time-to-live) is configured so cached data automatically expires after a defined duration.

At the client side, the REST controller adds HTTP cache headers such as `Cache-Control` and `ETag` to the response. Cache-Control allows browsers or CDNs to cache the response for a specified time, while `ETag` enables conditional requests. If the data has not changed, the server can respond with `304 Not Modified`, reducing bandwidth and improving performance.

1.Dependencies

```
<dependencies>
    <!-- Spring Web -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <!-- Spring Cache -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-cache</artifactId>
    </dependency>

    <!-- Redis -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-redis</artifactId>
    </dependency>

    <!-- Jackson (for Redis serialization) -->
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
    </dependency>
</dependencies>
```

2.Enable Caching

```
@SpringBootApplication
@EnableCaching
public class CachingApplication {
    public static void main(String[] args) {
        SpringApplication.run(CachingApplication.class, args);
    }
}
```

3.Redis Configuration

```
@Configuration
public class RedisConfig {

    @Bean
    public RedisCacheConfiguration cacheConfiguration() {
        return RedisCacheConfiguration.defaultCacheConfig()
                .entryTtl(Duration.ofMinutes(10)) // cache TTL
                .disableCachingNullValues();
    }
}
```

4.Service Layer (Server-Side Caching)
* First call will hit the DB. 
* Next call will be served from Redis.

```
@Service
public class ProductService {

    @Cacheable(value = "products", key = "#id")
    public Product getProductById(Long id) {
        simulateSlowCall();
        return new Product(id, "Product-" + id, 100.0);
    }

    private void simulateSlowCall() {
        try {
            Thread.sleep(2000); // simulate DB call
        } catch (InterruptedException ignored) {}
    }
}
```

5.REST Controller (Client-Side Caching via HTTP Headers)
* Cache-Control: public, max-age=60
* Browser/CDN caches response for 60 seconds
* ETag enables conditional requests (304 Not Modified)

```
@RestController
@RequestMapping("/products")
public class ProductController {

    private final ProductService productService;

    public ProductController(ProductService productService) {
        this.productService = productService;
    }

    @GetMapping("/{id}")
    public ResponseEntity<Product> getProduct(@PathVariable Long id) {
        Product product = productService.getProductById(id);

        String eTag = Integer.toHexString(product.hashCode());

        return ResponseEntity.ok()
                .cacheControl(CacheControl.maxAge(60, TimeUnit.SECONDS).cachePublic())
                .eTag(eTag)
                .body(product);
    }
}
```

6.Model Class (Java 17)

```
public record Product(Long id, String name, Double price) {}
```

7.application.properties

```
spring.redis.host=localhost
spring.redis.port=6379
```

### 2. Use asynchronous processing: Offload long-running tasks via queues and background workers to keep endpoints responsive.
To keep REST APIs responsive while handling long-running tasks, we use RabbitMQ as a message queue. Instead of processing heavy work inside the HTTP request thread, the application publishes a message to a RabbitMQ queue and returns the response immediately. A background consumer then processes the task asynchronously.

**Working**
1. The REST controller receives a request and creates a task message.
2. The message is published to a durable RabbitMQ queue using RabbitTemplate.
3. The API responds immediately with 202 Accepted.
4. A RabbitMQ consumer (@RabbitListener) listens to the queue and processes messages in the background.
5. RabbitMQ handles buffering, retries, and scalability, ensuring tasks are not lost.

This approach decouples API responsiveness from task execution, supports horizontal scaling, survives application restarts, and is suitable for production workloads such as file processing, notifications, and integrations.

![RabbitMQ Architecture](/assets/images/spring-boot-rabbit-mq.webp)

1.Dependencies

```
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-amqp</artifactId>
    </dependency>

    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <scope>provided</scope>
    </dependency>
</dependencies>
```

2.RabbitMQ configuration

```
@Configuration
public class RabbitMQConfig {

    public static final String QUEUE_NAME = "long-running-task-queue";

    @Bean
    public Queue taskQueue() {
        return new Queue(QUEUE_NAME, true);
    }

    @Bean
    public MessageConverter messageConverter() {
        return new Jackson2JsonMessageConverter();
    }
}
```

3.Message payload

```
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class TaskMessage {
    private String taskId;
}
```

4.Producer (publishes tasks to RabbitMQ)

```
@Service
@RequiredArgsConstructor
public class TaskProducer {

    private final RabbitTemplate rabbitTemplate;

    public void send(TaskMessage message) {
        rabbitTemplate.convertAndSend(RabbitMQConfig.QUEUE_NAME, message);
    }
}
```

5.Consumer (background worker)

```
@Service
public class TaskConsumer {

    @RabbitListener(queues = RabbitMQConfig.QUEUE_NAME)
    public void process(TaskMessage message) throws InterruptedException {
        System.out.println("Processing task: " + message.getTaskId());
        Thread.sleep(5000);
        System.out.println("Completed task: " + message.getTaskId());
    }
}
```

6.REST Controller (fast and responsive endpoint)

```
@RestController
@RequestMapping("/tasks")
@RequiredArgsConstructor
public class TaskController {

    private final TaskProducer producer;

    @PostMapping("/start")
    public ResponseEntity<String> startTask() {
        String taskId = UUID.randomUUID().toString();
        producer.send(TaskMessage.builder().taskId(taskId).build());
        return ResponseEntity.accepted().body("Task queued: " + taskId);
    }
}
```

7.application.properties

```
spring.rabbitmq.host=localhost
spring.rabbitmq.port=5672
spring.rabbitmq.username=guest
spring.rabbitmq.password=guest
```

### 3. Enable pagination: Process large data sets in manageable chunks.
Spring Boot, together with Spring Data JPA, provides built-in support for pagination through the Pageable abstraction. Pagination helps retrieve large datasets in manageable chunks, improving application performance, response times, and user experience.

Using pagination, the application avoids loading the entire result set into memory and instead fetches only a subset of records per request, typically implemented at the database level using LIMIT and OFFSET (for MySQL and compatible databases).

**Spring Data JPA exposes pagination via:**\
`Pageable` – Defines page number, page size, and sorting\
`Page<T>` – Represents paginated data along with metadata

#### Benefits of Using Pagination
**1. Performance Optimization**\
Prevents loading large datasets into memory and reduces database load.\
**2. Scalability**\
Enables APIs to scale efficiently as data grows.\
**3. Improved User Experience**\
Supports page-by-page navigation and faster UI rendering.\
**4. Clean API Design**\
Pagination metadata (total pages, total elements) is automatically provided.\
**5. Database-Agnostic**\
JPA translates pagination to database-specific SQL.

1.Entity

```
@Entity
@Table(name = "users")
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;
    private String email;

    // getters & setters
}
```

2.Repository

```
public interface UserRepository extends JpaRepository<User, Long> {
}
```

3.Service

```
@Service
public class UserService {

    @Autowired
    private UserRepository repository;

    public Page<User> getUsers(int page, int size) {
        Pageable pageable = PageRequest.of(page, size, Sort.by("id"));
        return repository.findAll(pageable);
    }
}
```

4.Controller

```
@RestController
@RequestMapping("/api/users")
public class UserController {

    @Autowired
    private UserService service;

    @GetMapping
    public Page<User> getUsers(
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "10") int size
    ) {
        return service.getUsers(page, size);
    }
}
```

5.What Hibernate generates (MySQL)

```
select user0_.id as id1_1_0_, 
       user0_.name as name2_1_0_, 
       user0_.email as email3_1_0_ 
from user user0_ 
order by user0_.id asc limit 10 offset 0

select count(id) as col_0_0_1_ from user user1_
```

> `Page<T>` always triggers an additional COUNT(*) query which can sometimes be expensive on large tables.\
> Ensure indexes exist on columns used for sorting and filtering.\
> Never allow unbounded page sizes from clients. Always enforce a maximum page size (e.g., 50/100).


<!-- ### 4. Optimize database queries: Use indexes, reduce N+1 query problems, and ensure optimal ORM configurations. 
### 5. Use connection pooling: Pool database connections for reuse instead of creating and destroying on each request.
### 6. Reduce network hops through smart routing: Use dynamic load balancers and distributed architectures for minimal latency.
### 7. Compress payloads: Enable compression (e.g., gzip) for API requests and responses to decrease transfer times.
### 8. Apply rate limiting and throttling: Prevent API abuse and accidental overload by limiting requests per client or IP.
### 9. Monitor and set performance metrics: Continuously track response times, errors, and throughput to identify and react to bottlenecks promptly.
### 10. Use of HTTP/2 or HTTP/3 protocols brings multiplexing, header compression, and better connection reuse, significantly improving latency and throughput for APIs serving many concurrent clients.
### 11. Use graceful degradation and circuit breakers: Integrate patterns that allow your API to remain responsive under partial failure, such as fallback values or disabling non-critical features temporarily.
### 12. Tune JVM and garbage collection settings for Java APIs: Specifically for Java environments, optimizing JVM memory management, garbage collection intervals, and thread pools can yield significant latency improvements. -->