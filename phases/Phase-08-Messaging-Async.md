# Phase 8: Messaging & Async Processing (Week 12)

## 📌 Overview

In this phase, you'll learn asynchronous processing, message brokers (RabbitMQ, Kafka), and event-driven architecture.

---

## 8.1 Asynchronous Processing

### @Async Annotation

```java
// Enable async processing
@SpringBootApplication
@EnableAsync
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}

// Service with async methods
@Service
public class EmailService {

    // Simple async method
    @Async
    public void sendEmail(String to, String message) {
        System.out.println("Sending email to " + to + " in background thread");
        try {
            Thread.sleep(3000); // Simulate long operation
            System.out.println("Email sent successfully");
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }

    // Async with return value using CompletableFuture
    @Async
    public CompletableFuture<String> sendEmailAsync(String to, String message) {
        return CompletableFuture.supplyAsync(() -> {
            try {
                Thread.sleep(2000);
                return "Email sent to " + to;
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                return "Failed";
            }
        });
    }

    // Async with exception handling
    @Async
    public CompletableFuture<String> sendEmailWithError(String to, String message) {
        return CompletableFuture.supplyAsync(() -> {
            if (to == null || to.isEmpty()) {
                throw new IllegalArgumentException("Invalid email");
            }
            return "Email sent to " + to;
        });
    }
}

// Controller using async service
@RestController
@RequestMapping("/api/emails")
public class EmailController {

    private EmailService emailService;

    public EmailController(EmailService emailService) {
        this.emailService = emailService;
    }

    @PostMapping("/send")
    public ResponseEntity<String> sendEmail(@RequestBody EmailRequest request) {
        emailService.sendEmail(request.getTo(), request.getMessage());
        return ResponseEntity.accepted().body("Email queued for sending");
    }

    @PostMapping("/send-async")
    public CompletableFuture<String> sendEmailAsync(@RequestBody EmailRequest request) {
        return emailService.sendEmailAsync(request.getTo(), request.getMessage());
    }
}

// EmailRequest.java
public class EmailRequest {
    private String to;
    private String message;

    // Getters and setters...
}
```

### Executor Configuration

```java
@Configuration
@EnableAsync
public class AsyncConfig {

    @Bean(name = "taskExecutor")
    public TaskExecutor taskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(2);
        executor.setMaxPoolSize(5);
        executor.setQueueCapacity(500);
        executor.setThreadNamePrefix("async-");
        executor.initialize();
        return executor;
    }

    @Bean
    public AsyncUncaughtExceptionHandler asyncUncaughtExceptionHandler() {
        return (throwable, method, params) -> {
            System.err.println("Async method " + method.getName() +
                             " threw exception: " + throwable.getMessage());
        };
    }
}

// Use custom executor
@Service
public class ProcessingService {

    @Async("taskExecutor")
    public void processLargeDataset(List<Data> data) {
        // Processing logic
    }
}
```

---

## 8.2 RabbitMQ Integration

### Add RabbitMQ Dependencies

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

### RabbitMQ Configuration

```properties
# application.properties
spring.rabbitmq.host=localhost
spring.rabbitmq.port=5672
spring.rabbitmq.username=guest
spring.rabbitmq.password=guest
spring.rabbitmq.virtual-host=/
```

### RabbitMQ Producer & Consumer

```java
// RabbitMQConfig.java
@Configuration
public class RabbitMQConfig {

    public static final String QUEUE_NAME = "user.queue";
    public static final String EXCHANGE_NAME = "user.exchange";
    public static final String ROUTING_KEY = "user.created";

    @Bean
    public Queue queue() {
        return new Queue(QUEUE_NAME, true);
    }

    @Bean
    public TopicExchange exchange() {
        return new TopicExchange(EXCHANGE_NAME, true, false);
    }

    @Bean
    public Binding binding(Queue queue, TopicExchange exchange) {
        return BindingBuilder.bind(queue)
            .to(exchange)
            .with(ROUTING_KEY);
    }
}

// UserEvent.java
public class UserEvent {
    private Long userId;
    private String email;
    private String action;

    // Constructor, getters, setters...
    public UserEvent(Long userId, String email, String action) {
        this.userId = userId;
        this.email = email;
        this.action = action;
    }
}

// UserEventProducer.java
@Service
public class UserEventProducer {

    private RabbitTemplate rabbitTemplate;

    public UserEventProducer(RabbitTemplate rabbitTemplate) {
        this.rabbitTemplate = rabbitTemplate;
    }

    public void publishUserCreatedEvent(User user) {
        UserEvent event = new UserEvent(user.getId(), user.getEmail(), "CREATED");
        rabbitTemplate.convertAndSend(
            RabbitMQConfig.EXCHANGE_NAME,
            RabbitMQConfig.ROUTING_KEY,
            event
        );
    }
}

// UserEventConsumer.java
@Service
public class UserEventConsumer {

    @RabbitListener(queues = RabbitMQConfig.QUEUE_NAME)
    public void handleUserCreatedEvent(UserEvent event) {
        System.out.println("Received event: " + event.getAction() +
                         " for user: " + event.getEmail());
        // Process event - send welcome email, update analytics, etc.
    }
}

// Usage in UserService
@Service
public class UserService {

    private UserRepository userRepository;
    private UserEventProducer eventProducer;

    public UserService(UserRepository userRepository, UserEventProducer eventProducer) {
        this.userRepository = userRepository;
        this.eventProducer = eventProducer;
    }

    public User createUser(User user) {
        User saved = userRepository.save(user);
        eventProducer.publishUserCreatedEvent(saved);
        return saved;
    }
}
```

---

## 8.3 Apache Kafka Integration

### Add Kafka Dependencies

```xml
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
</dependency>
```

### Kafka Configuration

```properties
# application.properties
spring.kafka.bootstrap-servers=localhost:9092
spring.kafka.producer.key-serializer=org.apache.kafka.common.serialization.StringSerializer
spring.kafka.producer.value-serializer=org.springframework.kafka.support.serializer.JsonSerializer
spring.kafka.consumer.group-id=user-service-group
spring.kafka.consumer.key-deserializer=org.apache.kafka.common.serialization.StringDeserializer
spring.kafka.consumer.value-deserializer=org.springframework.kafka.support.serializer.JsonDeserializer
spring.kafka.consumer.properties.spring.json.trusted.packages=*
```

### Kafka Producer & Consumer

```java
// KafkaConfig.java
@Configuration
public class KafkaConfig {

    public static final String USER_TOPIC = "user-events";
    public static final String ORDER_TOPIC = "order-events";
}

// UserEventKafkaProducer.java
@Service
public class UserEventKafkaProducer {

    private KafkaTemplate<String, UserEvent> kafkaTemplate;

    public UserEventKafkaProducer(KafkaTemplate<String, UserEvent> kafkaTemplate) {
        this.kafkaTemplate = kafkaTemplate;
    }

    public void publishUserCreatedEvent(User user) {
        UserEvent event = new UserEvent(user.getId(), user.getEmail(), "CREATED");

        kafkaTemplate.send(KafkaConfig.USER_TOPIC,
                          String.valueOf(user.getId()),
                          event)
            .addCallback(
                result -> System.out.println("Published event successfully"),
                ex -> System.err.println("Failed to publish event: " + ex.getMessage())
            );
    }
}

// UserEventKafkaConsumer.java
@Service
public class UserEventKafkaConsumer {

    @KafkaListener(topics = KafkaConfig.USER_TOPIC, groupId = "user-service")
    public void handleUserCreatedEvent(UserEvent event) {
        System.out.println("Received Kafka event: " + event.getAction() +
                         " for user: " + event.getEmail());
        // Process event
    }
}
```

---

## 8.4 CompletableFuture for Async Operations

```java
@Service
public class DataProcessingService {

    // Chaining async operations
    public CompletableFuture<Result> processDataChain(String input) {
        return CompletableFuture.supplyAsync(() -> fetchData(input))
            .thenApplyAsync(this::transformData)
            .thenApplyAsync(this::enrichData)
            .thenApplyAsync(this::analyzeData)
            .exceptionally(ex -> {
                System.err.println("Error in processing: " + ex.getMessage());
                return null;
            });
    }

    // Parallel async operations
    public CompletableFuture<CombinedResult> processParallel(String id) {
        CompletableFuture<Data> dataFuture = CompletableFuture
            .supplyAsync(() -> fetchData(id));

        CompletableFuture<Metadata> metadataFuture = CompletableFuture
            .supplyAsync(() -> fetchMetadata(id));

        CompletableFuture<Analytics> analyticsFuture = CompletableFuture
            .supplyAsync(() -> fetchAnalytics(id));

        return CompletableFuture.allOf(dataFuture, metadataFuture, analyticsFuture)
            .thenApplyAsync(v -> new CombinedResult(
                dataFuture.join(),
                metadataFuture.join(),
                analyticsFuture.join()
            ));
    }

    private Data fetchData(String input) {
        // Implementation
        return new Data();
    }

    private Data transformData(Data data) {
        // Implementation
        return data;
    }

    private Data enrichData(Data data) {
        // Implementation
        return data;
    }

    private Result analyzeData(Data data) {
        // Implementation
        return new Result();
    }

    private Metadata fetchMetadata(String id) {
        // Implementation
        return new Metadata();
    }

    private Analytics fetchAnalytics(String id) {
        // Implementation
        return new Analytics();
    }
}
```

---

## 📚 Summary

In **Phase 8**, you learned:

- ✅ @Async annotation and CompletableFuture
- ✅ Thread pool executor configuration
- ✅ RabbitMQ producer and consumer
- ✅ Kafka integration
- ✅ Asynchronous exception handling
- ✅ Parallel and chained async operations

---

**Ready to build microservices? Let's move to Phase 9! 🚀**
