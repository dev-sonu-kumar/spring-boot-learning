# Phase 7: Caching & Performance (Week 11)

## 📌 Overview

In this phase, you'll learn caching strategies, performance optimization, and monitoring techniques to build high-performing Spring Boot applications.

---

## 7.1 Caching with Spring Boot

### Add Cache Dependency

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>
```

### Enable Caching

```java
// Application.java
@SpringBootApplication
@EnableCaching
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

### @Cacheable Annotation

```java
@Service
public class UserService {

    private UserRepository userRepository;

    // Cache the result with key as the userId
    @Cacheable(value = "users", key = "#id")
    public User getUserById(Long id) {
        System.out.println("Fetching from database for ID: " + id);
        return userRepository.findById(id)
            .orElseThrow(() -> new RuntimeException("User not found"));
    }

    // Cache all users with a fixed key
    @Cacheable(value = "allUsers", key = "'all'")
    public List<User> getAllUsers() {
        System.out.println("Fetching all users from database");
        return userRepository.findAll();
    }

    // Custom cache key using SpEL
    @Cacheable(value = "usersByEmail", key = "#email.toLowerCase()")
    public User getUserByEmail(String email) {
        System.out.println("Fetching user by email: " + email);
        return userRepository.findByEmail(email)
            .orElseThrow(() -> new RuntimeException("User not found"));
    }

    // Conditional caching
    @Cacheable(value = "users", key = "#id", condition = "#id > 10")
    public User getUserWithCondition(Long id) {
        return userRepository.findById(id).orElse(null);
    }
}
```

### @CachePut - Update Cache

```java
@Service
public class UserService {

    @CachePut(value = "users", key = "#user.id")
    public User updateUser(User user) {
        System.out.println("Updating user: " + user.getId());
        return userRepository.save(user);
    }

    @CachePut(value = "users", key = "#id")
    public User partialUpdate(Long id, String email) {
        User user = userRepository.findById(id)
            .orElseThrow(() -> new RuntimeException("User not found"));
        user.setEmail(email);
        return userRepository.save(user);
    }
}
```

### @CacheEvict - Remove from Cache

```java
@Service
public class UserService {

    // Evict single cache entry
    @CacheEvict(value = "users", key = "#id")
    public void deleteUser(Long id) {
        System.out.println("Deleting user: " + id);
        userRepository.deleteById(id);
    }

    // Evict all entries in cache
    @CacheEvict(value = "users", allEntries = true)
    public void clearAllUserCache() {
        System.out.println("Clearing all user cache");
    }

    // Multiple cache evictions
    @Caching(evict = {
        @CacheEvict(value = "users", key = "#id"),
        @CacheEvict(value = "allUsers", key = "'all'")
    })
    public void deleteUserAndClearLists(Long id) {
        userRepository.deleteById(id);
    }
}
```

### Cache Configuration

```properties
# application.properties
spring.cache.type=simple

# Or specify cache manager
spring.cache.type=caffeine
spring.cache.caffeine.spec=maximumSize=500,expireAfterWrite=10m
```

```yaml
# application.yml
spring:
  cache:
    type: caffeine
    caffeine:
      spec: maximumSize=500,expireAfterWrite=10m
```

---

## 7.2 Redis Integration for Caching

### Add Redis Dependency

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>

<dependency>
    <groupId>io.lettuce</groupId>
    <artifactId>lettuce-core</artifactId>
</dependency>
```

### Configure Redis

```properties
# application.properties
spring.data.redis.host=localhost
spring.data.redis.port=6379
spring.data.redis.password=
spring.cache.type=redis
spring.cache.redis.time-to-live=600000
```

### Using Redis Cache

```java
@Service
public class ProductService {

    private ProductRepository productRepository;
    private RedisTemplate<String, Product> redisTemplate;

    public ProductService(ProductRepository productRepository,
                         RedisTemplate<String, Product> redisTemplate) {
        this.productRepository = productRepository;
        this.redisTemplate = redisTemplate;
    }

    @Cacheable(value = "products", key = "#id")
    public Product getProduct(Long id) {
        return productRepository.findById(id).orElse(null);
    }

    // Manual cache operations
    public Product getProductManually(Long id) {
        String key = "product:" + id;

        // Try to get from cache
        Product cached = redisTemplate.opsForValue().get(key);
        if (cached != null) {
            return cached;
        }

        // Get from database
        Product product = productRepository.findById(id).orElse(null);

        if (product != null) {
            // Store in cache for 10 minutes
            redisTemplate.opsForValue().set(key, product,
                Duration.ofMinutes(10));
        }

        return product;
    }
}

// Configure RedisTemplate
@Configuration
public class CacheConfig {

    @Bean
    public RedisTemplate<String, Product> redisTemplate(RedisConnectionFactory factory) {
        RedisTemplate<String, Product> template = new RedisTemplate<>();
        template.setConnectionFactory(factory);

        Jackson2JsonRedisSerializer<Product> jackson2JsonRedisSerializer =
            new Jackson2JsonRedisSerializer<>(Product.class);

        template.setDefaultSerializer(jackson2JsonRedisSerializer);
        template.setKeySerializer(new StringRedisSerializer());
        template.setValueSerializer(jackson2JsonRedisSerializer);

        return template;
    }
}
```

---

## 7.3 Performance Optimization

### Database Query Optimization

```java
// Use projections to fetch only needed fields
public interface ProductDTO {
    Long getId();
    String getName();
    Double getPrice();
}

public interface ProductRepository extends JpaRepository<Product, Long> {
    List<ProductDTO> findAllBy();
}

// Use batch operations
@Service
public class ProductService {

    private ProductRepository productRepository;

    public void saveBatch(List<Product> products) {
        // Save all at once (batching)
        productRepository.saveAll(products);
    }

    public void deleteBatch(List<Long> ids) {
        productRepository.deleteAllByIdInBatch(ids);
    }
}
```

### Connection Pooling

```properties
# application.properties
spring.datasource.hikari.maximum-pool-size=20
spring.datasource.hikari.minimum-idle=5
spring.datasource.hikari.connection-timeout=20000
spring.datasource.hikari.idle-timeout=300000
spring.datasource.hikari.max-lifetime=1200000
spring.datasource.hikari.auto-commit=true
```

### Lazy Loading vs Eager Loading

```java
// Avoid N+1 problem
@Entity
public class Author {

    @OneToMany(mappedBy = "author", fetch = FetchType.LAZY)
    private List<Book> books;
}

// Use JOIN FETCH to load in single query
public interface AuthorRepository extends JpaRepository<Author, Long> {

    @Query("SELECT DISTINCT a FROM Author a LEFT JOIN FETCH a.books")
    List<Author> findAllWithBooks();

    @EntityGraph(attributePaths = {"books"})
    List<Author> findAll();
}
```

---

## 7.4 Monitoring & Metrics with Actuator

### Add Actuator Dependency

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

### Configure Actuator

```properties
# application.properties
management.endpoints.web.exposure.include=health,info,metrics,prometheus
management.endpoint.health.show-details=always
management.metrics.export.prometheus.enabled=true
```

### Accessing Actuator Endpoints

```bash
# Health check
curl http://localhost:8080/actuator/health

# Application info
curl http://localhost:8080/actuator/info

# Metrics
curl http://localhost:8080/actuator/metrics

# Prometheus metrics
curl http://localhost:8080/actuator/prometheus
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
        User savedUser = userRepository.save(user);

        // Increment custom counter
        meterRegistry.counter("user.created").increment();

        return savedUser;
    }

    public void recordUserCount() {
        meterRegistry.gauge("user.count",
            userRepository::count);
    }

    public void recordOperationTime() {
        meterRegistry.timer("user.operation.time").record(() -> {
            // Long running operation
        });
    }
}
```

---

## 📚 Summary

In **Phase 7**, you learned:

- ✅ Caching with @Cacheable, @CachePut, @CacheEvict
- ✅ Redis integration for distributed caching
- ✅ Database query optimization
- ✅ Connection pooling configuration
- ✅ N+1 problem solutions
- ✅ Actuator for monitoring and metrics
- ✅ Custom metrics implementation

---

**Ready to handle async and messaging? Let's move to Phase 8! 🚀**
