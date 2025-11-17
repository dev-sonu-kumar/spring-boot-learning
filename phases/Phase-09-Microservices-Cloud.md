# Phase 9: Microservices & Cloud (Week 13-14)

## 📌 Overview

In this phase, you'll learn microservices architecture, service discovery, API gateways, circuit breakers, and cloud deployment strategies.

---

## 9.1 Microservices Architecture

### Microservices Principles

```
✅ Single Responsibility - One service, one business capability
✅ Loose Coupling - Services independent
✅ Autonomous - Own database, technology stack
✅ Observable - Logging, tracing, monitoring
✅ Scalable - Scale services independently
✅ Resilient - Handle failures gracefully
```

### Service Registration & Discovery (Eureka)

```xml
<!-- Eureka Server Dependency -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>

<!-- Eureka Client Dependency -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

#### Eureka Server

```java
@SpringBootApplication
@EnableEurekaServer
public class EurekaServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }
}
```

```properties
# application.properties
server.port=8761
spring.application.name=eureka-server
eureka.client.register-with-eureka=false
eureka.client.fetch-registry=false
```

#### Eureka Client (Microservice)

```java
@SpringBootApplication
@EnableDiscoveryClient
public class UserServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(UserServiceApplication.class, args);
    }
}
```

```properties
# application.properties
server.port=8081
spring.application.name=user-service
eureka.client.service-url.defaultZone=http://localhost:8761/eureka/
```

---

## 9.2 API Gateway

### Spring Cloud Gateway Configuration

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
```

```java
// GatewayConfig.java
@Configuration
public class GatewayConfig {

    @Bean
    public RouteLocator customRoutes(RouteLocatorBuilder builder) {
        return builder.routes()
            .route("user-service", r -> r
                .path("/users/**")
                .filters(f -> f
                    .stripPrefix(1)
                    .addRequestHeader("X-Service", "User-Service")
                )
                .uri("lb://user-service")
            )
            .route("order-service", r -> r
                .path("/orders/**")
                .filters(f -> f
                    .stripPrefix(1)
                    .addRequestHeader("X-Service", "Order-Service")
                )
                .uri("lb://order-service")
            )
            .route("product-service", r -> r
                .path("/products/**")
                .filters(f -> f
                    .stripPrefix(1)
                    .addRequestHeader("X-Service", "Product-Service")
                )
                .uri("lb://product-service")
            )
            .build();
    }
}
```

```properties
# application.properties
server.port=8080
spring.application.name=api-gateway
spring.cloud.gateway.discovery.locator.enabled=true
spring.cloud.gateway.discovery.locator.lower-case-service-id=true
```

---

## 9.3 Circuit Breaker Pattern (Resilience4j)

```xml
<dependency>
    <groupId>io.github.resilience4j</groupId>
    <artifactId>resilience4j-spring-boot3</artifactId>
    <version>2.0.0</version>
</dependency>

<dependency>
    <groupId>io.github.resilience4j</groupId>
    <artifactId>resilience4j-circuitbreaker</artifactId>
</dependency>

<dependency>
    <groupId>io.github.resilience4j</groupId>
    <artifactId>resilience4j-timelimiter</artifactId>
</dependency>
```

### Circuit Breaker Implementation

```java
@Service
public class OrderService {

    private RestTemplate restTemplate;

    // Circuit breaker for product service calls
    @CircuitBreaker(name = "productService", fallbackMethod = "productServiceFallback")
    @Retry(name = "productService")
    @TimeLimiter(name = "productService")
    public CompletableFuture<Product> getProduct(Long productId) {
        return CompletableFuture.supplyAsync(() ->
            restTemplate.getForObject(
                "http://product-service/api/products/" + productId,
                Product.class
            )
        );
    }

    public CompletableFuture<Product> productServiceFallback(Long productId, Exception ex) {
        System.err.println("Product service failed, using fallback: " + ex.getMessage());
        Product fallback = new Product();
        fallback.setId(productId);
        fallback.setName("Default Product");
        fallback.setPrice(0.0);
        return CompletableFuture.completedFuture(fallback);
    }
}
```

```properties
# application.properties
resilience4j.circuitbreaker.instances.productService.failure-rate-threshold=50
resilience4j.circuitbreaker.instances.productService.slow-call-rate-threshold=50
resilience4j.circuitbreaker.instances.productService.wait-duration-in-open-state=10000
resilience4j.circuitbreaker.instances.productService.slow-call-duration-threshold=2000
resilience4j.circuitbreaker.instances.productService.minimum-number-of-calls=10

resilience4j.retry.instances.productService.max-attempts=3
resilience4j.retry.instances.productService.wait-duration=1000

resilience4j.timelimiter.instances.productService.timeout-duration=3000
```

---

## 9.4 Inter-Service Communication

### RestTemplate

```java
@Configuration
public class RestTemplateConfig {

    @Bean
    public RestTemplate restTemplate(RestTemplateBuilder builder) {
        return builder
            .setConnectTimeout(Duration.ofSeconds(2))
            .setReadTimeout(Duration.ofSeconds(2))
            .build();
    }
}

@Service
public class UserService {

    private RestTemplate restTemplate;

    public User getUserWithOrders(Long userId) {
        User user = restTemplate.getForObject(
            "http://localhost:8081/users/" + userId,
            User.class
        );

        List<Order> orders = Arrays.asList(restTemplate.getForObject(
            "http://localhost:8082/orders/user/" + userId,
            Order[].class
        ));

        user.setOrders(orders);
        return user;
    }
}
```

### WebClient (Reactive HTTP Client)

```java
@Configuration
public class WebClientConfig {

    @Bean
    public WebClient webClient() {
        return WebClient.builder()
            .baseUrl("http://localhost:8080")
            .build();
    }
}

@Service
public class ReactiveUserService {

    private WebClient webClient;

    public Mono<User> getUser(Long userId) {
        return webClient.get()
            .uri("/users/{id}", userId)
            .retrieve()
            .bodyToMono(User.class)
            .timeout(Duration.ofSeconds(5));
    }

    public Flux<Order> getUserOrders(Long userId) {
        return webClient.get()
            .uri("/orders/user/{userId}", userId)
            .retrieve()
            .bodyToFlux(Order.class);
    }
}
```

### Feign Client

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

```java
@FeignClient(name = "product-service", url = "http://localhost:8083")
public interface ProductClient {

    @GetMapping("/products/{id}")
    Product getProduct(@PathVariable Long id);

    @PostMapping("/products")
    Product createProduct(@RequestBody Product product);
}

@Service
public class OrderService {

    private ProductClient productClient;

    public Order createOrder(OrderRequest request) {
        Product product = productClient.getProduct(request.getProductId());
        // Create order
        return new Order();
    }
}
```

---

## 9.5 Docker Containerization

### Dockerfile

```dockerfile
FROM openjdk:17-jdk-slim

WORKDIR /app

COPY target/user-service-1.0.0.jar app.jar

ENV JAVA_OPTS="-Xmx256m -Xms128m"

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "app.jar"]
```

### docker-compose.yml

```yaml
version: "3.8"

services:
  eureka-server:
    image: eureka-server:latest
    ports:
      - "8761:8761"
    environment:
      - JAVA_OPTS=-Xmx256m -Xms128m

  user-service:
    image: user-service:latest
    ports:
      - "8081:8080"
    depends_on:
      - eureka-server
      - mysql
    environment:
      - EUREKA_CLIENT_SERVICEURL_DEFAULTZONE=http://eureka-server:8761/eureka/
      - SPRING_DATASOURCE_URL=jdbc:mysql://mysql:3306/userdb
      - SPRING_DATASOURCE_USERNAME=root
      - SPRING_DATASOURCE_PASSWORD=password

  mysql:
    image: mysql:8.0
    ports:
      - "3306:3306"
    environment:
      - MYSQL_ROOT_PASSWORD=password
      - MYSQL_DATABASE=userdb
    volumes:
      - mysql_data:/var/lib/mysql

  api-gateway:
    image: api-gateway:latest
    ports:
      - "8080:8080"
    depends_on:
      - eureka-server
    environment:
      - EUREKA_CLIENT_SERVICEURL_DEFAULTZONE=http://eureka-server:8761/eureka/

volumes:
  mysql_data:
```

---

## 📚 Summary

In **Phase 9**, you learned:

- ✅ Microservices architecture principles
- ✅ Service discovery with Eureka
- ✅ API Gateway with Spring Cloud Gateway
- ✅ Circuit breaker pattern with Resilience4j
- ✅ RestTemplate and WebClient
- ✅ Feign client for service communication
- ✅ Docker containerization and docker-compose

---

**Ready to handle reactive programming? Let's move to Phase 10! 🚀**
