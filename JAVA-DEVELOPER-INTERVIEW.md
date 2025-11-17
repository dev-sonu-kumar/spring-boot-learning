# Java Developer Interview Preparation (7+ Years Experience)

## Overview

This document contains frequently asked interview questions for experienced Java developers with 7+ years of experience. Questions are organized by category and difficulty level.

---

## Table of Contents

1. [Core Java](#core-java)
2. [Advanced Java Concepts](#advanced-java-concepts)
3. [Spring & Spring Boot](#spring--spring-boot)
4. [Database & SQL](#database--sql)
5. [Microservices & Architecture](#microservices--architecture)
6. [Performance & Optimization](#performance--optimization)
7. [Concurrency & Multithreading](#concurrency--multithreading)
8. [Design Patterns](#design-patterns)
9. [System Design](#system-design)
10. [Real-World Scenarios](#real-world-scenarios)

---

## Core Java

### 1. Explain the difference between `==` and `.equals()` method

**Answer:**

- `==` compares object references (memory addresses)
- `.equals()` compares object content/values
- String comparison should use `.equals()` not `==`
- For custom objects, override `.equals()` method

```java
String s1 = new String("Hello");
String s2 = new String("Hello");
s1 == s2;        // false (different references)
s1.equals(s2);   // true (same content)
```

### 2. What is immutability? Why are Strings immutable in Java?

**Answer:**

- Immutable objects cannot be modified after creation
- String is immutable in Java for:
  - **Security**: String pool, password handling
  - **Performance**: Caching, reuse
  - **Thread Safety**: No synchronization needed
  - **HashMap Keys**: Can safely use as keys

### 3. What is the difference between ArrayList and LinkedList?

**Answer:**

| Feature            | ArrayList      | LinkedList          |
| ------------------ | -------------- | ------------------- |
| Data Structure     | Dynamic Array  | Doubly Linked List  |
| Access             | O(1)           | O(n)                |
| Insertion/Deletion | O(n)           | O(1)                |
| Memory             | Less overhead  | More overhead       |
| Use Case           | Frequent reads | Frequent insertions |

### 4. Explain HashMap vs. Hashtable vs. ConcurrentHashMap

**Answer:**

- **HashMap**: Not synchronized, faster, not thread-safe
- **Hashtable**: Synchronized, slower, thread-safe but legacy
- **ConcurrentHashMap**: Thread-safe with fine-grained locking (bucket level), modern choice for concurrent access

### 5. What is a Stream API? Give examples of common operations

**Answer:**

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

// Filter
numbers.stream()
  .filter(n -> n > 2)
  .forEach(System.out::println); // 3, 4, 5

// Map
numbers.stream()
  .map(n -> n * 2)
  .collect(Collectors.toList()); // [2, 4, 6, 8, 10]

// Reduce
int sum = numbers.stream()
  .reduce(0, Integer::sum); // 15

// GroupBy
Map<Boolean, List<Integer>> grouped = numbers.stream()
  .collect(Collectors.partitioningBy(n -> n > 2));
```

### 6. What is Optional? When and how to use it?

**Answer:**

```java
Optional<String> opt = Optional.of("Hello");

// Get with default
String result = opt.orElse("Default");

// Throw exception if empty
String value = opt.orElseThrow(() -> new RuntimeException("Empty!"));

// Map/FlatMap
Optional<Integer> length = opt.map(String::length);

// Filter
opt.filter(s -> s.startsWith("H"))
   .ifPresent(System.out::println);
```

### 7. Explain Exception Handling - Checked vs Unchecked Exceptions

**Answer:**

- **Checked Exceptions**: Must be caught/declared (`IOException`, `SQLException`)
- **Unchecked Exceptions**: Runtime exceptions (`NullPointerException`, `ArrayIndexOutOfBoundsException`)
- Use checked for recoverable errors
- Use unchecked for programming errors

### 8. What is the difference between final, finally, and finalize()?

**Answer:**

- **final**: Modifier for classes, methods, variables (immutable/non-overrideable)
- **finally**: Block that executes after try-catch
- **finalize()**: Method called by garbage collector before object destruction (deprecated in Java 9+)

---

## Advanced Java Concepts

### 9. What are Generics? Why use them?

**Answer:**

- Type-safe collections and methods
- Eliminates casting
- Catches errors at compile time
- Cannot use with primitive types

```java
List<String> list = new ArrayList<>();
list.add("Hello");
String str = list.get(0); // No casting needed

// Generic method
public <T> T getFirst(List<T> list) {
  return list.isEmpty() ? null : list.get(0);
}

// Bounded types
public <T extends Number> void process(T number) {
  // T must be Number or subclass
}
```

### 10. Explain Reflection API and when to use it

**Answer:**

```java
// Get class information at runtime
Class<?> clazz = Class.forName("com.example.User");

// Get methods
Method[] methods = clazz.getDeclaredMethods();

// Get fields
Field field = clazz.getDeclaredField("name");

// Create instance
Object instance = clazz.getDeclaredConstructor().newInstance();

// Invoke method
Method method = clazz.getMethod("getName");
Object result = method.invoke(instance);
```

**Use Cases**: Frameworks (Spring, Hibernate), Testing, Dependency Injection

### 11. What is the difference between abstract class and interface?

**Answer:**

| Aspect               | Abstract Class      | Interface                          |
| -------------------- | ------------------- | ---------------------------------- |
| Methods              | Concrete + Abstract | Abstract (Java 8+ default, static) |
| Variables            | Any modifier        | public static final                |
| Constructor          | Yes                 | No                                 |
| Access               | Any                 | public only                        |
| Multiple Inheritance | No                  | Yes                                |
| State                | Can have state      | Stateless (mostly)                 |

### 12. What are lambda expressions? How do they relate to functional interfaces?

**Answer:**

```java
// Traditional anonymous class
Runnable r = new Runnable() {
  public void run() { System.out.println("Hello"); }
};

// Lambda expression
Runnable r = () -> System.out.println("Hello");

// With parameters
Function<Integer, Integer> square = x -> x * x;
BiFunction<Integer, Integer, Integer> add = (a, b) -> a + b;

// Functional interface example
@FunctionalInterface
interface Calculator {
  int calculate(int a, int b);
}
```

### 13. What is the difference between Serial, Parallel, and Parallel Stream GC?

**Answer:**

- **Serial GC**: Single thread, full pause time (young + old generation)
- **Parallel GC**: Multiple threads, reduced pause time, better throughput
- **Parallel Stream GC**: Divides work across CPU cores for computation

### 14. Explain varargs (Variable Arguments)

**Answer:**

```java
// Varargs parameter
public void printNumbers(int... numbers) {
  for (int num : numbers) {
    System.out.println(num);
  }
}

// Usage
printNumbers(1, 2, 3, 4);
printNumbers(1);
printNumbers(); // Empty is allowed

// Important: Must be last parameter
public void mix(String name, int... values) {
  // OK
}
```

---

## Spring & Spring Boot

### 15. What is Dependency Injection and how does Spring implement it?

**Answer:**

- IoC Container manages object creation and dependencies
- Types: Constructor Injection, Setter Injection, Field Injection
- Constructor Injection is preferred (immutability, testability)

```java
@Component
public class UserService {
  private UserRepository repo;

  // Constructor Injection (Preferred)
  public UserService(UserRepository repo) {
    this.repo = repo;
  }
}

@Configuration
public class Config {
  @Bean
  public UserRepository userRepository() {
    return new UserRepository();
  }
}
```

### 16. Explain Spring Bean lifecycle

**Answer:**

1. Instantiation
2. Populate Properties
3. Set Bean Name (BeanNameAware)
4. Set Container (BeanFactoryAware)
5. Pre-Initialization (BeanPostProcessor)
6. InitializingBean
7. Custom init-method
8. Post-Initialization (BeanPostProcessor)
9. Bean Ready for Use
10. Destroy (DisposableBean)
11. Custom destroy-method

```java
@Component
public class MyBean implements InitializingBean, DisposableBean {

  @PostConstruct
  public void init() { }

  @Override
  public void afterPropertiesSet() { }

  @PreDestroy
  public void cleanup() { }

  @Override
  public void destroy() { }
}
```

### 17. What is the difference between @Bean and @Component?

**Answer:**

- **@Component**: Auto-detected, class-level, for Spring-managed classes
- **@Bean**: Explicit, method-level, for external library objects or conditional creation

```java
// @Component - automatic
@Component
public class UserService { }

// @Bean - manual configuration
@Configuration
public class Config {
  @Bean
  public JdbcTemplate jdbcTemplate(DataSource ds) {
    return new JdbcTemplate(ds);
  }
}
```

### 18. Explain Spring AOP (Aspect-Oriented Programming)

**Answer:**

```java
// Create aspect
@Aspect
@Component
public class LoggingAspect {

  // Pointcut
  @Pointcut("execution(* com.example.service.*.*(..))")
  public void serviceMethods() { }

  // Before advice
  @Before("serviceMethods()")
  public void logBefore(JoinPoint jp) {
    System.out.println("Executing: " + jp.getSignature());
  }

  // After advice
  @After("serviceMethods()")
  public void logAfter(JoinPoint jp) {
    System.out.println("Completed: " + jp.getSignature());
  }

  // Around advice
  @Around("serviceMethods()")
  public Object logAround(ProceedingJoinPoint pjp) throws Throwable {
    long start = System.currentTimeMillis();
    Object result = pjp.proceed();
    long time = System.currentTimeMillis() - start;
    System.out.println("Time taken: " + time + "ms");
    return result;
  }
}
```

### 19. What is Spring Transaction Management?

**Answer:**

```java
@Service
public class UserService {

  @Transactional
  public void saveUser(User user) {
    // Save user
    // If error occurs, transaction rolls back
  }

  // Custom transaction config
  @Transactional(
    propagation = Propagation.REQUIRED,
    isolation = Isolation.READ_COMMITTED,
    rollbackFor = Exception.class,
    timeout = 30
  )
  public void complexOperation() { }
}
```

**Propagation Types**: REQUIRED, REQUIRES_NEW, NESTED, SUPPORTS, NOT_SUPPORTED, NEVER

**Isolation Levels**: READ_UNCOMMITTED, READ_COMMITTED, REPEATABLE_READ, SERIALIZABLE

### 20. Explain Spring Security

**Answer:**

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

  @Bean
  public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http
      .authorizeRequests()
        .antMatchers("/public/**").permitAll()
        .antMatchers("/admin/**").hasRole("ADMIN")
        .anyRequest().authenticated()
      .and()
        .formLogin()
        .and()
        .logout();
    return http.build();
  }

  @Bean
  public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
  }
}
```

### 21. What is Spring Boot and how is it different from Spring?

**Answer:**

- **Spring**: Framework with extensive configuration
- **Spring Boot**: Opinionated, auto-configured, production-ready, embedded server

**Benefits**:

- Convention over configuration
- Embedded Tomcat/Jetty
- Auto-configuration
- Starter dependencies
- Actuator for monitoring

### 22. Explain Spring Data JPA

**Answer:**

```java
public interface UserRepository extends JpaRepository<User, Long> {

  // Derived query methods
  User findByEmail(String email);
  List<User> findByAgeGreaterThan(int age);
  List<User> findByFirstNameAndLastName(String firstName, String lastName);

  // Custom @Query
  @Query("SELECT u FROM User u WHERE u.age > ?1")
  List<User> findAdultUsers(int minAge);

  // Named parameters
  @Query("SELECT u FROM User u WHERE u.email = :email")
  User findByEmailCustom(@Param("email") String email);

  // Native SQL
  @Query(value = "SELECT * FROM users WHERE age > ?1", nativeQuery = true)
  List<User> findAdultNative(int age);
}

@Service
public class UserService {
  @Autowired
  private UserRepository repo;

  public User createUser(User user) {
    return repo.save(user);
  }
}
```

---

## Database & SQL

### 23. What are SQL Joins? Explain INNER, LEFT, RIGHT, FULL OUTER

**Answer:**

```sql
-- INNER JOIN: Common records only
SELECT u.name, o.order_id
FROM users u
INNER JOIN orders o ON u.id = o.user_id;

-- LEFT JOIN: All from left + matching from right
SELECT u.name, o.order_id
FROM users u
LEFT JOIN orders o ON u.id = o.user_id;

-- RIGHT JOIN: All from right + matching from left
SELECT u.name, o.order_id
FROM users u
RIGHT JOIN orders o ON u.id = o.user_id;

-- FULL OUTER JOIN: All records from both
SELECT u.name, o.order_id
FROM users u
FULL OUTER JOIN orders o ON u.id = o.user_id;

-- CROSS JOIN: Cartesian product
SELECT u.name, o.order_id
FROM users u
CROSS JOIN orders o;
```

### 24. What is the difference between WHERE and HAVING clause?

**Answer:**

- **WHERE**: Filters rows before grouping
- **HAVING**: Filters groups after grouping

```sql
-- WHERE filters records
SELECT category, COUNT(*) as count
FROM products
WHERE price > 100
GROUP BY category;

-- HAVING filters groups
SELECT category, COUNT(*) as count
FROM products
GROUP BY category
HAVING COUNT(*) > 5;

-- Both together
SELECT category, COUNT(*) as count
FROM products
WHERE price > 100
GROUP BY category
HAVING COUNT(*) > 5;
```

### 25. Explain Normalization (1NF, 2NF, 3NF, BCNF)

**Answer:**

- **1NF**: Atomic values, no repeating groups
- **2NF**: 1NF + No partial dependencies
- **3NF**: 2NF + No transitive dependencies
- **BCNF**: Every determinant is a candidate key

### 26. What is Denormalization? When to use it?

**Answer:**

- Combining normalized tables to improve query performance
- Use when: Complex joins are slow, reporting requirements, read-heavy operations
- Cost: Data redundancy, update anomalies

### 27. What are Indexes? Types of Indexes

**Answer:**
**Types**:

- **Primary Key Index**: Unique, non-null
- **Unique Index**: Unique values
- **Composite Index**: Multiple columns
- **Full-text Index**: Text search

```sql
-- Create indexes
CREATE INDEX idx_email ON users(email);
CREATE UNIQUE INDEX idx_username ON users(username);
CREATE INDEX idx_name_age ON users(first_name, last_name);

-- Query execution plan
EXPLAIN SELECT * FROM users WHERE email = 'test@example.com';
```

### 28. What is N+1 Query Problem?

**Answer:**

```java
// Bad: N+1 problem
@Entity
public class User {
  @OneToMany
  private List<Order> orders;
}

// Causes N+1 queries
List<User> users = userRepository.findAll(); // 1 query
for (User user : users) {
  System.out.println(user.getOrders()); // N additional queries
}

// Solution 1: Eager loading with JOIN FETCH
@Query("SELECT DISTINCT u FROM User u JOIN FETCH u.orders")
List<User> findAllWithOrders();

// Solution 2: @EntityGraph
@EntityGraph(attributePaths = "orders")
List<User> findAll();

// Solution 3: Projection/DTO
public interface UserDTO {
  Long getId();
  String getName();
}
```

---

## Microservices & Architecture

### 29. What are Microservices? Advantages and Disadvantages

**Answer:**
**Advantages**:

- Independent deployment and scaling
- Technology flexibility
- Fault isolation
- Team autonomy

**Disadvantages**:

- Distributed system complexity
- Network latency and failures
- Data consistency challenges
- Operational overhead

### 30. What is Service Discovery?

**Answer:**

- Mechanism to locate service instances dynamically
- **Client-side**: Client queries registry
- **Server-side**: Load balancer queries registry

**Tools**: Eureka, Consul, Zookeeper

```java
// Eureka Server
@SpringBootApplication
@EnableEurekaServer
public class EurekaServerApp { }

// Eureka Client
@SpringBootApplication
@EnableEurekaClient
public class UserServiceApp {
  // Automatically registers with Eureka
}

// Client communication
@Bean
public RestTemplate restTemplate() {
  return new RestTemplate();
}
```

### 31. What is API Gateway?

**Answer:**

- Single entry point for client requests
- Route requests to appropriate services
- Handle cross-cutting concerns

**Responsibilities**:

- Authentication/Authorization
- Rate limiting
- Request/Response transformation
- Load balancing

### 32. Explain Circuit Breaker Pattern

**Answer:**

```java
// Using Resilience4j
@Service
public class OrderService {

  @CircuitBreaker(name = "paymentService", fallbackMethod = "fallback")
  @Retry(name = "paymentService", fallbackMethod = "fallback")
  @Timeout(name = "paymentService")
  public Payment processPayment(Order order) {
    // Call payment service
    return paymentService.pay(order);
  }

  public Payment fallback(Order order, Exception e) {
    log.error("Payment service failed", e);
    // Return default or cached response
    return new Payment();
  }
}
```

**States**: CLOSED → OPEN → HALF_OPEN → CLOSED

### 33. What is eventually consistent data in distributed systems?

**Answer:**

- Data across services may be temporarily inconsistent
- But eventually becomes consistent
- Better availability and partition tolerance
- Tradeoff from CAP theorem

**Strategies**:

- Event sourcing
- Saga pattern
- Compensation transactions

### 34. Explain the Saga Pattern

**Answer:**

```java
// Choreography-based Saga
@Service
public class OrderService {
  @Transactional
  public void createOrder(Order order) {
    orderRepository.save(order);
    orderCreatedEvent.publish(order);
  }
}

@Service
public class PaymentService {
  @EventListener
  public void onOrderCreated(OrderCreatedEvent event) {
    processPayment(event.getOrder());
    paymentProcessedEvent.publish(event);
  }
}

// Orchestration-based Saga
@Service
public class SagaOrchestrator {
  public void createOrder(Order order) {
    // Step 1: Create order
    orderService.create(order);

    // Step 2: Process payment
    if (!paymentService.process(order)) {
      orderService.cancel(order);
      return;
    }

    // Step 3: Update inventory
    if (!inventoryService.reserve(order)) {
      paymentService.refund(order);
      orderService.cancel(order);
    }
  }
}
```

### 35. What is Event Sourcing?

**Answer:**

- Store all state changes as immutable events
- Reconstruct state by replaying events
- Event store is single source of truth

```java
@Entity
public class OrderEvent {
  private String eventType; // OrderCreated, PaymentProcessed, etc
  private String aggregate;
  private String data;
  private LocalDateTime timestamp;
}

public class Order {
  private List<OrderEvent> events = new ArrayList<>();

  public void applyEvent(OrderEvent event) {
    switch (event.getEventType()) {
      case "OrderCreated":
        this.status = "CREATED";
        break;
      case "PaymentProcessed":
        this.status = "PAID";
        break;
    }
    events.add(event);
  }

  public void reconstruct() {
    events.forEach(this::applyEvent);
  }
}
```

---

## Performance & Optimization

### 36. What are common Java performance bottlenecks?

**Answer:**

1. **Memory Leaks**: Unreleased references
2. **Inefficient Algorithms**: O(n²) when O(n) possible
3. **N+1 Query Problem**: Extra database calls
4. **Synchronization**: Excessive locking
5. **String Concatenation**: Use StringBuilder in loops
6. **Collection Operations**: Choosing wrong data structure

### 37. How to optimize database queries?

**Answer:**

```sql
-- 1. Use proper indexes
CREATE INDEX idx_user_email ON users(email);

-- 2. Use EXPLAIN to analyze query
EXPLAIN SELECT * FROM users WHERE email = 'test@example.com';

-- 3. Avoid SELECT *
SELECT id, name, email FROM users; -- Better

-- 4. Use LIMIT
SELECT * FROM users LIMIT 100;

-- 5. Batch operations
INSERT INTO users VALUES (...), (...), (...); -- Better than multiple inserts

-- 6. Use proper JOINs
SELECT u.name, COUNT(o.id) FROM users u
LEFT JOIN orders o ON u.id = o.user_id
GROUP BY u.id;
```

### 38. What is caching? Types and strategies

**Answer:**

```java
// Spring Cache
@Service
public class UserService {

  @Cacheable("users")
  public User getUserById(Long id) {
    // Only called if not in cache
    return userRepository.findById(id).orElse(null);
  }

  @CachePut("users")
  public User updateUser(User user) {
    return userRepository.save(user);
  }

  @CacheEvict("users")
  public void deleteUser(Long id) {
    userRepository.deleteById(id);
  }

  @CacheEvict(value = "users", allEntries = true)
  public void clearAllCache() { }
}
```

**Cache Strategies**:

- **Cache-aside**: Application manages cache
- **Write-through**: Write to cache + DB simultaneously
- **Write-behind**: Write to cache, async to DB
- **Refresh-ahead**: Refresh cache before expiry

### 39. What is pagination and why is it important?

**Answer:**

```java
// Spring Data pagination
public interface UserRepository extends JpaRepository<User, Long> {
  Page<User> findAll(Pageable pageable);
}

@GetMapping("/users")
public Page<User> getUsers(
  @RequestParam(defaultValue = "0") int page,
  @RequestParam(defaultValue = "20") int size,
  @RequestParam(defaultValue = "id,desc") String sort
) {
  Pageable pageable = PageRequest.of(page, size, Sort.by("id").descending());
  return userRepository.findAll(pageable);
}
```

**Response**:

```json
{
  "content": [...],
  "pageable": {
    "pageNumber": 0,
    "pageSize": 20
  },
  "totalElements": 500,
  "totalPages": 25,
  "last": false
}
```

### 40. What is the difference between vertical and horizontal scaling?

**Answer:**

- **Vertical Scaling**: Add more resources (CPU, RAM) to single machine
- **Horizontal Scaling**: Add more machines to distribute load

**Horizontal Scaling Challenges**:

- Session management
- Data consistency
- Load balancing
- Service discovery

---

## Concurrency & Multithreading

### 41. What is the difference between Thread and Runnable?

**Answer:**

- **Thread**: Class, can extend only one class
- **Runnable**: Interface, implement alongside other interfaces

```java
// Extend Thread
class MyThread extends Thread {
  public void run() {
    System.out.println("Running");
  }
}
new MyThread().start();

// Implement Runnable (Preferred)
class MyRunnable implements Runnable {
  public void run() {
    System.out.println("Running");
  }
}
new Thread(new MyRunnable()).start();

// Lambda
new Thread(() -> System.out.println("Running")).start();
```

### 42. What is ThreadPool? When to use ExecutorService?

**Answer:**

```java
// Single thread executor
ExecutorService executor = Executors.newSingleThreadExecutor();
executor.execute(() -> System.out.println("Task"));

// Fixed thread pool
ExecutorService executor = Executors.newFixedThreadPool(10);
for (int i = 0; i < 100; i++) {
  executor.execute(() -> doTask());
}

// Cached thread pool
ExecutorService executor = Executors.newCachedThreadPool();

// Scheduled executor
ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(5);
scheduler.schedule(() -> doTask(), 2, TimeUnit.SECONDS);
scheduler.scheduleAtFixedRate(() -> doTask(), 0, 5, TimeUnit.SECONDS);

// Graceful shutdown
executor.shutdown();
if (!executor.awaitTermination(10, TimeUnit.SECONDS)) {
  executor.shutdownNow();
}
```

### 43. What are synchronized methods and blocks?

**Answer:**

```java
// Synchronized method
public synchronized void criticalSection() {
  // Only one thread at a time
}

// Synchronized block
public void method() {
  synchronized(this) {
    // Only one thread at a time
  }
}

// Synchronized on object
Object lock = new Object();
synchronized(lock) {
  // Only one thread accessing this block with same lock
}
```

### 44. What is Volatile keyword?

**Answer:**

- Ensures variable visibility across threads
- Prevents compiler optimizations
- Not a substitute for synchronization

```java
// Variable changes visible to all threads immediately
volatile boolean flag = false;

public void method() {
  flag = true; // All threads see this immediately
}

// Without volatile - changes might be cached in thread's CPU cache
boolean flag = false;
```

### 45. Explain Lock interfaces (Lock, ReadWriteLock, ReentrantLock)

**Answer:**

```java
// ReentrantLock
Lock lock = new ReentrantLock();
try {
  lock.lock();
  // Critical section
} finally {
  lock.unlock();
}

// Try lock with timeout
if (lock.tryLock(2, TimeUnit.SECONDS)) {
  try {
    // Critical section
  } finally {
    lock.unlock();
  }
}

// ReadWriteLock - multiple readers, single writer
ReadWriteLock rwLock = new ReentrantReadWriteLock();
rwLock.readLock().lock(); // Multiple threads can read
rwLock.writeLock().lock(); // Only one thread can write
```

### 46. What is CountDownLatch, CyclicBarrier, Semaphore?

**Answer:**

```java
// CountDownLatch: Wait for N operations to complete
CountDownLatch latch = new CountDownLatch(3);
for (int i = 0; i < 3; i++) {
  executor.execute(() -> {
    doTask();
    latch.countDown();
  });
}
latch.await(); // Wait for all to complete

// CyclicBarrier: Wait for N threads to reach a point
CyclicBarrier barrier = new CyclicBarrier(3, () -> {
  System.out.println("All threads ready!");
});
for (int i = 0; i < 3; i++) {
  executor.execute(() -> {
    doPartialTask();
    barrier.await(); // Wait for others
    doRestOfTask();
  });
}

// Semaphore: Limit concurrent access
Semaphore semaphore = new Semaphore(2); // Max 2 threads
for (int i = 0; i < 10; i++) {
  executor.execute(() -> {
    semaphore.acquire();
    try {
      limitedResource();
    } finally {
      semaphore.release();
    }
  });
}
```

### 47. What is Deadlock? How to prevent it?

**Answer:**
**Deadlock Conditions**:

- Mutual exclusion
- Hold and wait
- No preemption
- Circular wait

**Prevention**:

```java
// Bad: Can cause deadlock
synchronized(resource1) {
  synchronized(resource2) {
    // Do something
  }
}

// Good: Consistent ordering
public void transfer(Account from, Account to) {
  Account first = from.getId() < to.getId() ? from : to;
  Account second = from.getId() < to.getId() ? to : from;

  synchronized(first) {
    synchronized(second) {
      // Transfer money
    }
  }
}

// Better: Use timeout
if (lock1.tryLock(1, TimeUnit.SECONDS) && lock2.tryLock(1, TimeUnit.SECONDS)) {
  try {
    // Do work
  } finally {
    lock1.unlock();
    lock2.unlock();
  }
}
```

### 48. What is ForkJoinPool?

**Answer:**

```java
// Parallel processing with divide and conquer
public class SumCalculator extends RecursiveTask<Long> {
  private long[] array;
  private int start, end;
  private static final int THRESHOLD = 1000;

  @Override
  protected Long compute() {
    if (end - start <= THRESHOLD) {
      long sum = 0;
      for (int i = start; i < end; i++) {
        sum += array[i];
      }
      return sum;
    } else {
      int mid = (start + end) / 2;
      SumCalculator left = new SumCalculator(array, start, mid);
      SumCalculator right = new SumCalculator(array, mid, end);

      left.fork();
      long rightResult = right.compute();
      long leftResult = left.join();

      return leftResult + rightResult;
    }
  }
}

// Use
ForkJoinPool pool = new ForkJoinPool();
long result = pool.invoke(new SumCalculator(array, 0, array.length));
```

---

## Design Patterns

### 49. Explain Singleton Pattern

**Answer:**

```java
// Thread-safe singleton (Bill Pugh)
public class Singleton {
  private Singleton() { }

  private static class SingletonHelper {
    private static final Singleton INSTANCE = new Singleton();
  }

  public static Singleton getInstance() {
    return SingletonHelper.INSTANCE;
  }
}

// Enum singleton (Most recommended)
public enum Singleton {
  INSTANCE;

  public void method() { }
}
Singleton.INSTANCE.method();
```

### 50. Explain Factory Pattern

**Answer:**

```java
// Factory interface
public interface PaymentFactory {
  Payment createPayment();
}

// Concrete factories
public class CreditCardFactory implements PaymentFactory {
  @Override
  public Payment createPayment() {
    return new CreditCardPayment();
  }
}

public class PayPalFactory implements PaymentFactory {
  @Override
  public Payment createPayment() {
    return new PayPalPayment();
  }
}

// Client
public class OrderService {
  public void processOrder(String paymentType) {
    PaymentFactory factory;
    if ("creditcard".equals(paymentType)) {
      factory = new CreditCardFactory();
    } else if ("paypal".equals(paymentType)) {
      factory = new PayPalFactory();
    }
    Payment payment = factory.createPayment();
    payment.process();
  }
}
```

### 51. Explain Builder Pattern

**Answer:**

```java
// Builder pattern
public class User {
  private final String name;
  private final String email;
  private final int age;
  private final String phone;

  private User(Builder builder) {
    this.name = builder.name;
    this.email = builder.email;
    this.age = builder.age;
    this.phone = builder.phone;
  }

  public static class Builder {
    private String name;
    private String email;
    private int age;
    private String phone;

    public Builder name(String name) {
      this.name = name;
      return this;
    }

    public Builder email(String email) {
      this.email = email;
      return this;
    }

    public User build() {
      return new User(this);
    }
  }
}

// Usage
User user = new User.Builder()
  .name("John")
  .email("john@example.com")
  .build();
```

### 52. Explain Strategy Pattern

**Answer:**

```java
// Strategy interface
public interface PaymentStrategy {
  void pay(double amount);
}

// Concrete strategies
public class CreditCardStrategy implements PaymentStrategy {
  @Override
  public void pay(double amount) {
    System.out.println("Paying " + amount + " via Credit Card");
  }
}

public class PayPalStrategy implements PaymentStrategy {
  @Override
  public void pay(double amount) {
    System.out.println("Paying " + amount + " via PayPal");
  }
}

// Context
public class ShoppingCart {
  private PaymentStrategy strategy;

  public void setPaymentStrategy(PaymentStrategy strategy) {
    this.strategy = strategy;
  }

  public void checkout(double total) {
    strategy.pay(total);
  }
}

// Usage
ShoppingCart cart = new ShoppingCart();
cart.setPaymentStrategy(new CreditCardStrategy());
cart.checkout(100);
```

### 53. Explain Observer Pattern

**Answer:**

```java
// Subject/Observable
public class Subject {
  private List<Observer> observers = new ArrayList<>();

  public void attach(Observer observer) {
    observers.add(observer);
  }

  public void detach(Observer observer) {
    observers.remove(observer);
  }

  public void notifyObservers() {
    observers.forEach(Observer::update);
  }
}

// Observer interface
public interface Observer {
  void update();
}

// Concrete observers
public class ConcreteObserverA implements Observer {
  @Override
  public void update() {
    System.out.println("ObserverA updated");
  }
}

// Usage with Spring events
@Component
public class UserCreatedEvent extends ApplicationEvent {
  private User user;

  public UserCreatedEvent(Object source, User user) {
    super(source);
    this.user = user;
  }
}

@Component
public class EmailNotifier {
  @EventListener
  public void onUserCreated(UserCreatedEvent event) {
    sendWelcomeEmail(event.getUser());
  }
}

@Service
public class UserService {
  @Autowired
  private ApplicationEventPublisher publisher;

  public void createUser(User user) {
    publisher.publishEvent(new UserCreatedEvent(this, user));
  }
}
```

### 54. Explain Decorator Pattern

**Answer:**

```java
// Component interface
public interface Coffee {
  double getCost();
  String getDescription();
}

// Concrete component
public class SimpleCoffee implements Coffee {
  @Override
  public double getCost() { return 2.0; }

  @Override
  public String getDescription() { return "Simple Coffee"; }
}

// Decorator
public abstract class CoffeeDecorator implements Coffee {
  protected Coffee coffee;

  public CoffeeDecorator(Coffee coffee) {
    this.coffee = coffee;
  }
}

// Concrete decorators
public class MilkDecorator extends CoffeeDecorator {
  public MilkDecorator(Coffee coffee) { super(coffee); }

  @Override
  public double getCost() { return coffee.getCost() + 0.5; }

  @Override
  public String getDescription() {
    return coffee.getDescription() + ", Milk";
  }
}

// Usage
Coffee coffee = new SimpleCoffee();
coffee = new MilkDecorator(coffee);
coffee = new SugarDecorator(coffee);
System.out.println(coffee.getDescription()); // Simple Coffee, Milk, Sugar
System.out.println(coffee.getCost()); // 3.0
```

### 55. Explain Proxy Pattern

**Answer:**

```java
// Subject interface
public interface ImageLoader {
  void display();
}

// Real subject
public class RealImageLoader implements ImageLoader {
  private String filename;

  public RealImageLoader(String filename) {
    this.filename = filename;
    loadImage();
  }

  private void loadImage() {
    System.out.println("Loading " + filename);
  }

  @Override
  public void display() {
    System.out.println("Displaying " + filename);
  }
}

// Proxy
public class ImageProxy implements ImageLoader {
  private RealImageLoader realLoader;
  private String filename;

  public ImageProxy(String filename) {
    this.filename = filename;
  }

  @Override
  public void display() {
    if (realLoader == null) {
      realLoader = new RealImageLoader(filename);
    }
    realLoader.display();
  }
}

// Usage
ImageLoader loader = new ImageProxy("photo.jpg");
loader.display(); // Loads and displays only when needed
```

---

## System Design

### 56. How would you design a URL Shortening Service (like TinyURL)?

**Answer:**

**Requirements**:

- Generate short unique URLs
- Redirect to original URL
- High availability and low latency
- Analytics: Click count, source tracking

**Design**:

```
1. API Design
   - POST /shorten: { long_url: "..." } → { short_url: "..." }
   - GET /{short_code}: Redirect to original URL

2. Database Schema
   - short_urls table: id, short_code, long_url, user_id, created_at, clicks
   - Create index on short_code for fast lookup

3. Generating short codes
   - Option 1: Base62 encoding of incremental ID
   - Option 2: Random 6-character string with collision handling
   - Option 3: Hash first 8 characters of MD5(long_url + timestamp)

4. Architecture
   - Web servers (stateless)
   - Load balancer
   - Cache layer (Redis): Cache popular URLs
   - Database
   - Message queue for analytics

5. Scale
   - Sharding by user_id or short_code range
   - Read replicas for analytics queries
   - CDN for redirects

6. Handling collisions
   - Check if short_code exists
   - Retry with different short_code or increment counter
```

### 57. How would you design an E-Commerce System?

**Answer:**

**Core Services**:

- User Service: Authentication, profiles
- Product Service: Catalog management
- Cart Service: Shopping cart
- Order Service: Order management
- Payment Service: Payment processing
- Inventory Service: Stock management
- Notification Service: Email/SMS

**Database Design**:

```sql
-- Users
CREATE TABLE users (
  id BIGINT PRIMARY KEY,
  email VARCHAR(255) UNIQUE,
  password_hash VARCHAR(255),
  created_at TIMESTAMP
);

-- Products
CREATE TABLE products (
  id BIGINT PRIMARY KEY,
  name VARCHAR(255),
  price DECIMAL(10,2),
  category_id BIGINT,
  inventory_id BIGINT
);

-- Orders
CREATE TABLE orders (
  id BIGINT PRIMARY KEY,
  user_id BIGINT,
  total_amount DECIMAL(10,2),
  status VARCHAR(50),
  created_at TIMESTAMP
);

-- Order Items
CREATE TABLE order_items (
  id BIGINT PRIMARY KEY,
  order_id BIGINT,
  product_id BIGINT,
  quantity INT,
  price DECIMAL(10,2)
);

-- Inventory
CREATE TABLE inventory (
  id BIGINT PRIMARY KEY,
  product_id BIGINT,
  quantity INT,
  updated_at TIMESTAMP
);
```

**Key Challenges & Solutions**:

```
1. Distributed Transactions
   - Use Saga pattern for order creation + payment + inventory

2. Inventory Management
   - Lock inventory during checkout
   - Use optimistic locking or distributed locks
   - Handle overselling with reservation system

3. Payment Processing
   - PCI compliance - don't store card details
   - Use payment gateway (Stripe, PayPal)
   - Idempotent payment processing

4. High Availability
   - API Gateway for routing
   - Service replication
   - Database replication with failover

5. Analytics
   - Event streaming (Kafka)
   - Data warehouse (BigQuery, Redshift)
   - Real-time dashboards
```

### 58. How would you design a Twitter-like System?

**Answer:**

**Core Features**:

- User registration and authentication
- Post tweets
- Follow/Unfollow users
- Like tweets
- Retweets
- Timeline (News Feed)
- Search

**Architecture**:

```
1. User Service
   - User profiles
   - Follow relationships

2. Tweet Service
   - Create, read, delete tweets
   - Tweet metadata (likes, retweets)

3. Feed Service
   - Generate news feed for user
   - Challenges: Fanout on write vs read

4. Search Service
   - Full-text search on tweets
   - Elasticsearch

5. Notification Service
   - Real-time notifications
   - WebSocket for push

6. Analytics Service
   - Trending topics
   - User engagement metrics
```

**Feed Generation Challenge**:

```
Option 1: Fanout on Write
- When user posts, write to all followers' feeds
- Good for read performance
- Bad for users with many followers

Option 2: Fanout on Read
- Compute feed on-the-fly when user requests
- Good for write performance
- Bad for read latency

Option 3: Hybrid
- Fanout for regular users
- Compute on-read for high-follower users
```

### 59. How would you design a Rate Limiting System?

**Answer:**

**Requirements**:

- Limit requests per user/IP
- Configurable limits
- Distributed across multiple servers

**Algorithms**:

```java
// Token Bucket Algorithm
public class TokenBucket {
  private long tokens;
  private final long capacity;
  private final long refillRate; // tokens per second
  private long lastRefillTime;

  public synchronized boolean tryConsume(long tokens) {
    refill();
    if (this.tokens >= tokens) {
      this.tokens -= tokens;
      return true;
    }
    return false;
  }

  private void refill() {
    long now = System.currentTimeMillis();
    long timePassed = now - lastRefillTime;
    long tokensToAdd = (timePassed / 1000) * refillRate;
    tokens = Math.min(capacity, tokens + tokensToAdd);
    lastRefillTime = now;
  }
}

// Sliding Window
public class SlidingWindow {
  private final Queue<Long> timestamps = new LinkedList<>();
  private final int maxRequests;
  private final long windowSize; // milliseconds

  public synchronized boolean tryConsume() {
    long now = System.currentTimeMillis();

    // Remove old timestamps
    while (!timestamps.isEmpty() && timestamps.peek() < now - windowSize) {
      timestamps.poll();
    }

    if (timestamps.size() < maxRequests) {
      timestamps.offer(now);
      return true;
    }
    return false;
  }
}
```

**Distributed Rate Limiting with Redis**:

```java
@Service
public class RateLimitingService {
  @Autowired
  private RedisTemplate<String, Long> redis;

  public boolean isAllowed(String userId, int limit, int windowSeconds) {
    String key = "rate_limit:" + userId;
    long count = redis.opsForValue().increment(key);

    if (count == 1) {
      redis.expire(key, Duration.ofSeconds(windowSeconds));
    }

    return count <= limit;
  }
}
```

### 60. How would you design a Notification System?

**Answer:**

**Components**:

```
1. Notification Service
   - Receive notification requests
   - Validate and queue notifications

2. Message Queue (Kafka/RabbitMQ)
   - Decouple sender and receiver
   - Ensure reliability

3. Workers
   - Consume notifications from queue
   - Send via appropriate channel (Email, SMS, Push)

4. Delivery Service
   - Actual sending (integrate with providers)
   - Retry logic
   - Deduplication

5. Tracking Service
   - Track delivery status
   - Analytics

6. User Preferences Service
   - User notification settings
   - Do-not-disturb timing
```

**Implementation**:

```java
@Service
public class NotificationService {
  @Autowired
  private KafkaTemplate<String, NotificationEvent> kafkaTemplate;

  public void sendNotification(NotificationEvent event) {
    kafkaTemplate.send("notifications", event.getUserId(), event);
  }
}

@Service
public class NotificationWorker {
  @KafkaListener(topics = "notifications")
  public void processNotification(NotificationEvent event) {
    switch(event.getType()) {
      case EMAIL:
        emailService.send(event);
        break;
      case SMS:
        smsService.send(event);
        break;
      case PUSH:
        pushService.send(event);
        break;
    }
  }
}

@Service
public class EmailService {
  public void send(NotificationEvent event) {
    try {
      mailSender.send(buildMessage(event));
      updateStatus(event.getId(), "DELIVERED");
    } catch (Exception e) {
      updateStatus(event.getId(), "FAILED");
      // Retry logic
    }
  }
}
```

---

## Real-World Scenarios

### 61. How would you handle a service outage or failure?

**Answer:**

- **Monitoring**: Alerts on error rates, latency, dependencies
- **Circuit Breaker**: Fail fast, fallback responses
- **Retry Logic**: Exponential backoff
- **Load Balancer**: Route away from failing instances
- **Database Failover**: Read replicas, replication
- **Cache**: Serve stale data during outage
- **Graceful Degradation**: Reduce functionality vs complete failure
- **Communication**: Notify users of issues

### 62. How would you optimize a slow API endpoint?

**Answer:**

1. **Monitor & Profile**: Identify bottleneck (DB, code, external API)
2. **Database Optimization**:

   - Add indexes
   - Optimize queries (JOIN optimization)
   - Reduce result set size
   - Pagination

3. **Application Level**:

   - Cache results
   - Async processing
   - Batch operations
   - Optimize algorithm

4. **Infrastructure**:
   - Add load balancer
   - Horizontal scaling
   - CDN for static content
   - Increase database capacity

```java
// Example: Optimize slow query
// Before
@GetMapping("/users/{id}/orders")
public List<Order> getUserOrders(@PathVariable Long id) {
  User user = userRepository.findById(id); // 1 query
  return user.getOrders(); // N+1 query problem
}

// After
@GetMapping("/users/{id}/orders")
public List<Order> getUserOrders(@PathVariable Long id) {
  return orderRepository.findByUserId(id); // 1 optimized query
}
```

### 63. How would you debug a memory leak in Java?

**Answer:**

```java
// Tools and techniques
1. JVM Flags
   -XX:+PrintGCDetails
   -XX:+PrintGCTimeStamps
   -XX:+HeapDumpOnOutOfMemoryError

2. Memory Profiling
   // Use JProfiler, YourKit, or JvisualVM

3. Code patterns causing leaks
   // Unclosed resources
   InputStream is = new FileInputStream("file");
   // Should be:
   try (InputStream is = new FileInputStream("file")) {
     // use is
   }

   // Listeners not unregistered
   view.addClickListener(listener); // Not removed later

   // Static collections
   public static List<Object> cache = new ArrayList<>(); // Never cleared

   // Thread locals
   ThreadLocal<Connection> tl = new ThreadLocal<>();
   tl.set(connection); // Must call tl.remove()

// Heap dump analysis
jmap -dump:live,format=b,file=heap.bin <PID>
jhat heap.bin
```

### 64. How do you handle distributed transactions?

**Answer:**

```java
// Saga Pattern - Choreography
@Service
public class OrderService {
  @Transactional
  public void createOrder(Order order) {
    order.setStatus("PENDING");
    orderRepo.save(order);
    kafkaTemplate.send("order-created", order);
  }
}

@Service
public class PaymentService {
  @KafkaListener(topics = "order-created")
  public void onOrderCreated(Order order) {
    try {
      processPayment(order);
      kafkaTemplate.send("payment-completed", order);
    } catch (Exception e) {
      kafkaTemplate.send("payment-failed", order);
    }
  }
}

@Service
public class InventoryService {
  @KafkaListener(topics = "payment-completed")
  public void onPaymentCompleted(Order order) {
    try {
      reserveInventory(order);
      kafkaTemplate.send("order-confirmed", order);
    } catch (Exception e) {
      // Compensating transaction
      kafkaTemplate.send("order-rollback", order);
    }
  }
}

// Saga Pattern - Orchestration
@Service
public class OrderOrchestrator {
  @Transactional
  public void createOrder(Order order) {
    orderService.create(order);

    if (!paymentService.process(order)) {
      orderService.cancel(order);
      throw new Exception("Payment failed");
    }

    if (!inventoryService.reserve(order)) {
      paymentService.refund(order);
      orderService.cancel(order);
      throw new Exception("Inventory unavailable");
    }
  }
}

// Two-Phase Commit (Less preferred in microservices)
// Coordinator asks all services to prepare
// If all agree, asks all to commit
// If any disagree, asks all to rollback
```

### 65. What is your approach to code review?

**Answer:**

- **Check Logic**: Is it correct? Edge cases?
- **Code Quality**: Readability, DRY principle, SOLID
- **Performance**: N+1 queries, unnecessary loops
- **Security**: SQL injection, XSRF, authentication
- **Testing**: Unit tests, edge cases
- **Documentation**: Clear comments for complex logic
- **Feedback**: Constructive, learning opportunity

### 66. How would you approach learning a new technology?

**Answer:**

1. **Understand fundamentals**: Core concepts, use cases
2. **Read documentation**: Official docs, guides
3. **Build small projects**: Hands-on practice
4. **Study real examples**: GitHub projects, open source
5. **Keep updated**: Blogs, conferences, communities
6. **Practice frequently**: Regular coding
7. **Teach others**: Solidifies understanding

### 67. Describe a challenging project you worked on

**Answer:**
Structure: Problem → Approach → Solution → Learning

```
Problem: High latency in user search API (10+ seconds)
  - Users searching large database
  - Complex filtering criteria
  - Multiple joins causing slow queries

Approach:
  - Analyzed database logs and query plans
  - Identified missing indexes and N+1 queries
  - Discussed with team about requirements

Solution:
  - Added proper indexes on search fields
  - Implemented Elasticsearch for full-text search
  - Added Redis caching for popular searches
  - Implemented pagination
  - Added circuit breaker for external APIs

Result: Latency reduced to <500ms, 20x improvement

Learning:
  - Importance of monitoring and profiling
  - Different tools for different scenarios
  - Collaboration with team for large changes
```

### 68. How do you stay updated with Java ecosystem?

**Answer:**

- Follow Java release notes and JEPs
- Subscribe to tech blogs: DZone, Baeldung
- Attend conferences: JavaOne, Spring I/O
- Participate in communities: Stack Overflow
- Contribute to open source
- Read books: Effective Java, Clean Code
- Follow influential Java developers on Twitter

### 69. What are your strengths and weaknesses?

**Answer:**
**Strengths**:

- Problem-solving and debugging skills
- System design and architecture thinking
- Attention to code quality and testing
- Communication and collaboration
- Continuous learning

**Weaknesses** (with improvement):

- Sometimes over-engineer simple problems (learning to balance)
- Public speaking (taking courses, practicing)
- Limited DevOps experience (learning cloud platforms)

### 70. Why do you want to join our company?

**Answer:**

- Research company beforehand
- Genuine interest in their tech stack
- Company culture and mission
- Growth opportunities
- Impact of work
- Team and leadership

---

## General Tips for Interview Success

1. **Preparation**

   - Practice coding problems (LeetCode, HackerRank)
   - Mock interviews
   - Review company's tech stack
   - Prepare your own questions

2. **During Interview**

   - Think aloud - explain your approach
   - Ask clarifying questions
   - Don't rush - quality over speed
   - Be confident but humble
   - Admit what you don't know

3. **Code Quality**

   - Write clean, readable code
   - Handle edge cases
   - Write comments for complex logic
   - Test your code mentally

4. **System Design**

   - Start with requirements
   - Don't jump to implementation
   - Discuss tradeoffs
   - Ask questions and iterate

5. **Soft Skills**

   - Active listening
   - Positive attitude
   - Humility and willingness to learn
   - Communication clarity
   - Problem-solving approach

6. **Follow-up**
   - Send thank you email
   - Reiterate interest
   - Address any concerns
   - Ask next steps timeline

---

## Additional Resources

- **Books**: Effective Java, Clean Code, Design Patterns, System Design Interview
- **Online**: LeetCode, InterviewBit, System Design Primer (GitHub)
- **Blogs**: Baeldung, DZone, JavaWorld, Spring Blog
- **Videos**: YouTube channels - Tech Primers, Coding Revolution, Cracking Interviews

---

Good luck with your interview! Remember to be yourself, stay calm, and showcase your knowledge and problem-solving abilities. You've got this! 🚀
