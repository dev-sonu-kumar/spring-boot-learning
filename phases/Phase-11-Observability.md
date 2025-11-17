# Phase 11: Observability & Monitoring (Week 17)

## 📌 Overview

In this phase, you'll master logging, monitoring, metrics collection, and distributed tracing for production-ready Spring Boot applications.

---

## 11.1 Logging Configuration

### SLF4J and Logback

```xml
<!-- Included by default in spring-boot-starter -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-logging</artifactId>
</dependency>
```

### Logback Configuration

Create `src/main/resources/logback-spring.xml`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <property name="LOG_PATTERN"
              value="%d{yyyy-MM-dd HH:mm:ss} - %logger{36} - %msg%n"/>
    <property name="LOG_FILE" value="logs/application.log"/>

    <!-- Console Appender -->
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>${LOG_PATTERN}</pattern>
        </encoder>
    </appender>

    <!-- File Appender -->
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${LOG_FILE}</file>
        <encoder>
            <pattern>${LOG_PATTERN}</pattern>
        </encoder>
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <fileNamePattern>logs/application-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <maxFileSize>10MB</maxFileSize>
            <maxHistory>30</maxHistory>
        </rollingPolicy>
    </appender>

    <!-- Spring Profiles -->
    <springProfile name="dev">
        <logger name="com.example.demo" level="DEBUG"/>
        <root level="INFO">
            <appender-ref ref="CONSOLE"/>
        </root>
    </springProfile>

    <springProfile name="prod">
        <logger name="com.example.demo" level="INFO"/>
        <root level="WARN">
            <appender-ref ref="FILE"/>
        </root>
    </springProfile>
</configuration>
```

### Structured Logging

```java
@RestController
@RequestMapping("/api/users")
public class UserController {

    private static final Logger logger = LoggerFactory.getLogger(UserController.class);
    private UserService userService;

    public UserController(UserService userService) {
        this.userService = userService;
    }

    @PostMapping
    public ResponseEntity<User> createUser(@RequestBody User user) {
        logger.info("Creating user with email: {}", user.getEmail());

        try {
            User created = userService.createUser(user);
            logger.debug("User created successfully with ID: {}", created.getId());
            return ResponseEntity.status(HttpStatus.CREATED).body(created);
        } catch (Exception e) {
            logger.error("Error creating user with email: {}", user.getEmail(), e);
            throw e;
        }
    }

    @GetMapping("/{id}")
    public ResponseEntity<User> getUser(@PathVariable Long id) {
        logger.debug("Fetching user with ID: {}", id);

        User user = userService.getUser(id)
            .orElseThrow(() -> {
                logger.warn("User not found with ID: {}", id);
                return new RuntimeException("User not found");
            });

        return ResponseEntity.ok(user);
    }
}
```

---

## 11.2 Metrics with Micrometer

### Add Dependencies

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>

<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
```

### Configure Metrics

```properties
# application.properties
management.endpoints.web.exposure.include=health,info,metrics,prometheus
management.endpoint.health.show-details=always
management.metrics.export.prometheus.enabled=true
management.metrics.tags.application=${spring.application.name}
```

### Custom Metrics

```java
@Service
public class UserService {

    private UserRepository userRepository;
    private MeterRegistry meterRegistry;

    public UserService(UserRepository userRepository, MeterRegistry meterRegistry) {
        this.userRepository = userRepository;
        this.meterRegistry = meterRegistry;
    }

    public User createUser(User user) {
        // Counter - count total users created
        meterRegistry.counter("user.created.total").increment();

        User saved = userRepository.save(user);

        // Gauge - track current user count
        meterRegistry.gauge("user.count", userRepository::count);

        return saved;
    }

    public List<User> getAllUsers() {
        // Timer - measure operation time
        return meterRegistry.timer("user.fetch.time").recordCallable(() ->
            userRepository.findAll()
        );
    }

    public void deleteUser(Long id) {
        meterRegistry.counter("user.deleted.total").increment();
        userRepository.deleteById(id);
    }

    public void recordUserRegistrationTime(long millis) {
        meterRegistry.timer("user.registration.time").record(millis, TimeUnit.MILLISECONDS);
    }
}
```

### Prometheus Integration

```java
@Configuration
public class PrometheusConfig {

    @Bean
    public MeterBinder userMetrics(UserRepository userRepository) {
        return (registry) -> {
            Gauge.builder("user.total", userRepository::count)
                .description("Total number of users")
                .register(registry);
        };
    }
}
```

Access metrics: `http://localhost:8080/actuator/prometheus`

---

## 11.3 Distributed Tracing with Sleuth and Zipkin

### Add Dependencies

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-sleuth</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-sleuth-zipkin</artifactId>
</dependency>
```

### Configure Sleuth

```properties
# application.properties
spring.application.name=user-service
spring.sleuth.tracing.sampler.probability=1.0
spring.zipkin.base-url=http://localhost:9411
management.zipkin.tracing.endpoint=http://localhost:9411/api/v2/spans
```

### Automatic Trace Logging

```java
// Sleuth automatically adds trace ID to logs
@Service
public class OrderService {

    private static final Logger logger = LoggerFactory.getLogger(OrderService.class);
    private UserClient userClient;
    private OrderRepository orderRepository;

    public Order createOrder(OrderRequest request) {
        // Trace ID automatically added to logs
        logger.info("Creating order for user: {}", request.getUserId());

        User user = userClient.getUser(request.getUserId());
        Order order = new Order();
        order.setUser(user);
        order.setTotal(request.getTotal());

        return orderRepository.save(order);
    }
}
```

### Docker Compose for Zipkin

```yaml
version: "3"
services:
  zipkin:
    image: openzipkin/zipkin:latest
    ports:
      - "9411:9411"
    environment:
      - STORAGE_TYPE=mem
```

Access Zipkin UI: `http://localhost:9411`

---

## 11.4 Health Checks

### Custom Health Indicator

```java
@Component
public class DatabaseHealthIndicator implements HealthIndicator {

    private DataSource dataSource;

    public DatabaseHealthIndicator(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    @Override
    public Health health() {
        try (Connection connection = dataSource.getConnection()) {
            if (connection.isValid(2)) {
                return Health.up()
                    .withDetail("database", "MySQL")
                    .withDetail("version", "8.0")
                    .build();
            }
        } catch (SQLException e) {
            return Health.down()
                .withDetail("error", e.getMessage())
                .build();
        }
        return Health.unknown().build();
    }
}

@Component
public class ExternalServiceHealthIndicator implements HealthIndicator {

    private RestTemplate restTemplate;

    public ExternalServiceHealthIndicator(RestTemplate restTemplate) {
        this.restTemplate = restTemplate;
    }

    @Override
    public Health health() {
        try {
            ResponseEntity<String> response = restTemplate.getForEntity(
                "http://external-api/health", String.class
            );
            if (response.getStatusCode() == HttpStatus.OK) {
                return Health.up()
                    .withDetail("service", "External API")
                    .build();
            }
        } catch (Exception e) {
            return Health.down()
                .withDetail("error", e.getMessage())
                .build();
        }
        return Health.unknown().build();
    }
}
```

Access health endpoint: `http://localhost:8080/actuator/health`

---

## 11.5 Application Insights (Azure)

```xml
<dependency>
    <groupId>com.microsoft.azure</groupId>
    <artifactId>applicationinsights-spring-boot-starter</artifactId>
    <version>2.6.4</version>
</dependency>
```

```properties
# application.properties
spring.application.insights.instrumentation-key=<your-key>
spring.application.insights.enabled=true
```

---

## 📚 Summary

In **Phase 11**, you learned:

- ✅ SLF4J and Logback configuration
- ✅ Structured logging
- ✅ Metrics collection with Micrometer
- ✅ Prometheus integration
- ✅ Custom metrics
- ✅ Distributed tracing with Sleuth and Zipkin
- ✅ Custom health indicators
- ✅ Azure Application Insights

---

**Ready to apply all concepts? Let's move to Phase 12! 🚀**
