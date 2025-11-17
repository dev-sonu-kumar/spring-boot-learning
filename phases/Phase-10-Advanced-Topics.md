# Phase 10: Advanced Topics (Week 15-16)

## 📌 Overview

In this phase, you'll explore reactive programming, event-driven architecture, GraphQL, and other advanced concepts for building modern Spring Boot applications.

---

## 10.1 Reactive Programming with Project Reactor

### Add Dependencies

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>

<dependency>
    <groupId>io.projectreactor</groupId>
    <artifactId>reactor-core</artifactId>
</dependency>
```

### Mono and Flux

```java
// Mono - 0 or 1 element
public class MonoExample {

    public Mono<User> getUserById(Long id) {
        return Mono.just(new User(id, "John Doe", "john@example.com"));
    }

    public Mono<User> getUserByIdWithDelay(Long id) {
        return Mono.just(new User(id, "John Doe", "john@example.com"))
            .delayElement(Duration.ofSeconds(2));
    }

    public Mono<User> getUserByIdEmpty() {
        return Mono.empty();
    }

    public Mono<User> getUserByIdError() {
        return Mono.error(new RuntimeException("User not found"));
    }
}

// Flux - Multiple elements (0 to many)
public class FluxExample {

    public Flux<User> getAllUsers() {
        return Flux.just(
            new User(1L, "John Doe", "john@example.com"),
            new User(2L, "Jane Smith", "jane@example.com"),
            new User(3L, "Bob Johnson", "bob@example.com")
        );
    }

    public Flux<User> getAllUsersWithDelay() {
        return Flux.just(
            new User(1L, "John Doe", "john@example.com"),
            new User(2L, "Jane Smith", "jane@example.com"),
            new User(3L, "Bob Johnson", "bob@example.com")
        ).delayElement(Duration.ofMillis(500));
    }

    public Flux<User> getAllUsersFromDatabase(UserRepository repository) {
        return Flux.fromIterable(repository.findAll());
    }
}
```

### Reactive Controller

```java
@RestController
@RequestMapping("/api/users")
public class ReactiveUserController {

    private UserRepository userRepository;

    public ReactiveUserController(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    @GetMapping("/{id}")
    public Mono<User> getUserById(@PathVariable Long id) {
        return Mono.fromCallable(() ->
            userRepository.findById(id).orElseThrow()
        ).subscribeOn(Schedulers.boundedElastic());
    }

    @GetMapping
    public Flux<User> getAllUsers() {
        return Flux.fromIterable(userRepository.findAll())
            .subscribeOn(Schedulers.boundedElastic());
    }

    @PostMapping
    public Mono<User> createUser(@RequestBody User user) {
        return Mono.fromCallable(() -> userRepository.save(user))
            .subscribeOn(Schedulers.boundedElastic());
    }
}
```

### Reactive Operations

```java
@Service
public class ReactiveUserService {

    private UserRepository userRepository;

    // map - transform each element
    public Flux<UserDTO> getAllUsersAsDTO() {
        return Flux.fromIterable(userRepository.findAll())
            .map(user -> new UserDTO(user.getId(), user.getName()));
    }

    // filter - filter elements
    public Flux<User> getActiveUsers() {
        return Flux.fromIterable(userRepository.findAll())
            .filter(User::isActive);
    }

    // flatMap - transform and flatten
    public Flux<Order> getUserOrders(Long userId) {
        return Flux.just(userId)
            .flatMap(id -> {
                User user = userRepository.findById(id).orElse(null);
                return Flux.fromIterable(user.getOrders());
            });
    }

    // zip - combine multiple streams
    public Flux<UserOrderPair> getUsersWithOrders() {
        Flux<User> users = Flux.fromIterable(userRepository.findAll());
        Flux<Order> orders = getAllOrders();

        return Flux.zip(users, orders)
            .map(tuple -> new UserOrderPair(tuple.getT1(), tuple.getT2()));
    }

    // merge - combine multiple streams
    public Flux<Event> getAllEvents() {
        return Flux.merge(
            getUserEvents(),
            getOrderEvents(),
            getProductEvents()
        );
    }

    // cache - cache the result
    public Mono<List<User>> getAllUsersCached() {
        return Mono.fromCallable(() -> userRepository.findAll())
            .cache();
    }
}
```

---

## 10.2 Event-Driven Architecture

### Spring Events

```java
// UserCreatedEvent.java
public class UserCreatedEvent extends ApplicationEvent {
    private User user;

    public UserCreatedEvent(Object source, User user) {
        super(source);
        this.user = user;
    }

    public User getUser() {
        return user;
    }
}

// UserService.java - Publisher
@Service
public class UserService {

    private UserRepository userRepository;
    private ApplicationEventPublisher eventPublisher;

    public UserService(UserRepository userRepository,
                      ApplicationEventPublisher eventPublisher) {
        this.userRepository = userRepository;
        this.eventPublisher = eventPublisher;
    }

    public User createUser(User user) {
        User saved = userRepository.save(user);
        eventPublisher.publishEvent(new UserCreatedEvent(this, saved));
        return saved;
    }
}

// UserEventListener.java - Subscriber
@Component
public class UserEventListener {

    @EventListener
    public void handleUserCreatedEvent(UserCreatedEvent event) {
        User user = event.getUser();
        System.out.println("User created: " + user.getEmail());
        // Send welcome email, initialize user settings, etc.
    }

    @EventListener
    @Async
    public void handleUserCreatedEventAsync(UserCreatedEvent event) {
        // Async processing
    }

    @TransactionalEventListener
    public void handleAfterCommit(UserCreatedEvent event) {
        // Executed after transaction commit
    }
}

// Multiple listeners
@Component
public class AnalyticsListener {

    @EventListener
    public void trackUserCreation(UserCreatedEvent event) {
        // Track in analytics service
    }
}
```

### Event Sourcing

```java
// Event.java
@Entity
public class UserEvent {

    @Id
    @GeneratedValue
    private Long id;

    private Long aggregateId; // User ID
    private String eventType; // CREATED, UPDATED, DELETED
    private String eventData; // JSON
    private LocalDateTime timestamp;

    // Getters and setters...
}

// EventStore.java
public interface EventStore extends JpaRepository<UserEvent, Long> {
    List<UserEvent> findByAggregateIdOrderByTimestamp(Long aggregateId);
}

// EventSourcingService.java
@Service
public class EventSourcingService {

    private EventStore eventStore;
    private UserRepository userRepository;
    private ObjectMapper objectMapper;

    public void publishEvent(Long userId, String eventType, Object eventData) {
        UserEvent event = new UserEvent();
        event.setAggregateId(userId);
        event.setEventType(eventType);
        event.setEventData(objectMapper.writeValueAsString(eventData));
        event.setTimestamp(LocalDateTime.now());

        eventStore.save(event);
    }

    public User reconstructUserFromEvents(Long userId) {
        List<UserEvent> events = eventStore.findByAggregateIdOrderByTimestamp(userId);
        User user = new User();

        for (UserEvent event : events) {
            applyEvent(user, event);
        }

        return user;
    }

    private void applyEvent(User user, UserEvent event) {
        switch (event.getEventType()) {
            case "CREATED" -> {
                // Apply creation event
            }
            case "UPDATED" -> {
                // Apply update event
            }
            case "DELETED" -> {
                // Apply deletion event
            }
        }
    }
}
```

---

## 10.3 GraphQL Integration

### Add GraphQL Dependencies

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-graphql</artifactId>
</dependency>
```

### GraphQL Schema

Create `src/main/resources/graphql/schema.graphqls`:

```graphql
type User {
  id: ID!
  name: String!
  email: String!
  orders: [Order!]!
}

type Order {
  id: ID!
  orderNumber: String!
  total: Float!
  user: User!
}

type Query {
  user(id: ID!): User
  users: [User!]!
  order(id: ID!): Order
  orders: [Order!]!
}

type Mutation {
  createUser(name: String!, email: String!): User!
  updateUser(id: ID!, name: String!, email: String!): User!
  deleteUser(id: ID!): Boolean!
  createOrder(userId: ID!, total: Float!): Order!
}
```

### GraphQL Controller

```java
@Controller
@RequestMapping("/graphql")
public class UserGraphQLController {

    private UserRepository userRepository;
    private OrderRepository orderRepository;

    public UserGraphQLController(UserRepository userRepository,
                                OrderRepository orderRepository) {
        this.userRepository = userRepository;
        this.orderRepository = orderRepository;
    }

    @QueryMapping
    public User user(@Argument Long id) {
        return userRepository.findById(id).orElse(null);
    }

    @QueryMapping
    public List<User> users() {
        return userRepository.findAll();
    }

    @QueryMapping
    public Order order(@Argument Long id) {
        return orderRepository.findById(id).orElse(null);
    }

    @MutationMapping
    public User createUser(@Argument String name, @Argument String email) {
        User user = new User();
        user.setName(name);
        user.setEmail(email);
        return userRepository.save(user);
    }

    @MutationMapping
    public User updateUser(@Argument Long id, @Argument String name, @Argument String email) {
        return userRepository.findById(id)
            .map(user -> {
                user.setName(name);
                user.setEmail(email);
                return userRepository.save(user);
            })
            .orElse(null);
    }

    @MutationMapping
    public boolean deleteUser(@Argument Long id) {
        userRepository.deleteById(id);
        return true;
    }

    @SchemaMapping
    public List<Order> orders(User user) {
        return orderRepository.findByUserId(user.getId());
    }
}
```

### Query Examples

```graphql
# Get user with orders
query {
  user(id: 1) {
    id
    name
    email
    orders {
      id
      orderNumber
      total
    }
  }
}

# Create user
mutation {
  createUser(name: "John Doe", email: "john@example.com") {
    id
    name
    email
  }
}

# Update user
mutation {
  updateUser(id: 1, name: "Jane Doe", email: "jane@example.com") {
    id
    name
    email
  }
}
```

---

## 📚 Summary

In **Phase 10**, you learned:

- ✅ Reactive programming with Mono and Flux
- ✅ Reactive operations (map, filter, flatMap, zip)
- ✅ Event-driven architecture with Spring Events
- ✅ Event sourcing pattern
- ✅ GraphQL integration
- ✅ GraphQL queries and mutations

---

**Ready to monitor and observe your system? Let's move to Phase 11! 🚀**
