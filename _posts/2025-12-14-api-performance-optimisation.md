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

1. Dependencies
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

2. Enable Caching
```
@SpringBootApplication
@EnableCaching
public class CachingApplication {
    public static void main(String[] args) {
        SpringApplication.run(CachingApplication.class, args);
    }
}
```

3. Redis Configuration
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

3. Service Layer (Server-Side Caching)
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

4. REST Controller (Client-Side Caching via HTTP Headers)
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

5. Model Class
```
public record Product(Long id, String name, Double price) {}
```

6. application.properties
```
spring.redis.host=localhost
spring.redis.port=6379
```

### 2. Use asynchronous processing: Offload long-running tasks via queues and background workers to keep endpoints responsive.

### 3. Enable pagination and data streaming: Process large data sets in manageable chunks or stream data as it’s available.
### 4. Optimize database queries: Use indexes, reduce N+1 query problems, and ensure optimal ORM configurations. 
### 5. Use connection pooling: Pool database connections for reuse instead of creating and destroying on each request.
### 6. Reduce network hops through smart routing: Use dynamic load balancers and distributed architectures for minimal latency.
### 7. Compress payloads: Enable compression (e.g., gzip) for API requests and responses to decrease transfer times.
### 8. Apply rate limiting and throttling: Prevent API abuse and accidental overload by limiting requests per client or IP.
### 9. Monitor and set performance metrics: Continuously track response times, errors, and throughput to identify and react to bottlenecks promptly.
### 10. Use of HTTP/2 or HTTP/3 protocols brings multiplexing, header compression, and better connection reuse, significantly improving latency and throughput for APIs serving many concurrent clients.
### 11. Use graceful degradation and circuit breakers: Integrate patterns that allow your API to remain responsive under partial failure, such as fallback values or disabling non-critical features temporarily.
### 12. Tune JVM and garbage collection settings for Java APIs: Specifically for Java environments, optimizing JVM memory management, garbage collection intervals, and thread pools can yield significant latency improvements.