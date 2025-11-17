# Phase 12: Real-World Practices (Week 18)

## 📌 Overview

In this final phase, you'll learn API design best practices, documentation, versioning strategies, CI/CD pipeline setup, and deploying to production.

---

## 12.1 API Design & Best Practices

### RESTful API Principles

```
✅ Use HTTP methods correctly (GET, POST, PUT, PATCH, DELETE)
✅ Use proper HTTP status codes
✅ Resource-based URLs (not action-based)
✅ Versioning strategy
✅ Consistent naming conventions
✅ Pagination for large datasets
✅ Filtering and sorting
✅ Error responses with details
```

### Good API Design Example

```java
// ❌ Bad API Design
@GetMapping("/getUser/{userId}")
public User get(@PathVariable Long userId) { }

@GetMapping("/getUserOrders/{userId}")
public List<Order> getUserOrders(@PathVariable Long userId) { }

@PostMapping("/createUser")
public User create(@RequestBody User user) { }

// ✅ Good API Design
@GetMapping("/users/{userId}")
public ResponseEntity<User> getUser(@PathVariable Long userId) { }

@GetMapping("/users/{userId}/orders")
public ResponseEntity<Page<Order>> getUserOrders(
    @PathVariable Long userId,
    @RequestParam(defaultValue = "0") int page,
    @RequestParam(defaultValue = "20") int size
) { }

@PostMapping("/users")
public ResponseEntity<User> createUser(@Valid @RequestBody User user) { }
```

### Standard Response Structure

```java
@Component
@RestControllerAdvice
public class ApiResponseAdvice {

    @PostMapping("/{id}")
    public ResponseEntity<ApiResponse<User>> updateUser(
        @PathVariable Long id,
        @RequestBody User user
    ) {
        User updated = userService.updateUser(id, user);
        return ResponseEntity.ok(
            ApiResponse.success(updated, "User updated successfully")
        );
    }
}

// ApiResponse.java
public class ApiResponse<T> {
    private boolean success;
    private T data;
    private String message;
    private LocalDateTime timestamp;

    private ApiResponse(boolean success, T data, String message) {
        this.success = success;
        this.data = data;
        this.message = message;
        this.timestamp = LocalDateTime.now();
    }

    public static <T> ApiResponse<T> success(T data, String message) {
        return new ApiResponse<>(true, data, message);
    }

    public static <T> ApiResponse<T> error(String message) {
        return new ApiResponse<>(false, null, message);
    }

    // Getters...
}
```

---

## 12.2 API Documentation with Springdoc OpenAPI

### Add Dependencies

```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.0.0</version>
</dependency>
```

### Configure Swagger/OpenAPI

```properties
# application.properties
springdoc.api-docs.path=/api-docs
springdoc.swagger-ui.path=/swagger-ui.html
springdoc.swagger-ui.operations-sorter=method
springdoc.swagger-ui.tags-sorter=alpha
```

### Documented Controller

```java
@RestController
@RequestMapping("/api/users")
@Tag(name = "User Management", description = "APIs for managing users")
public class UserController {

    private UserService userService;

    @GetMapping
    @Operation(summary = "Get all users", description = "Retrieve a paginated list of all users")
    @Parameters({
        @Parameter(name = "page", description = "Page number (0-indexed)"),
        @Parameter(name = "size", description = "Page size"),
        @Parameter(name = "sort", description = "Sort criteria")
    })
    public ResponseEntity<Page<User>> getAllUsers(
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "20") int size,
        @RequestParam(defaultValue = "id") String sort
    ) {
        return ResponseEntity.ok(userService.getAllUsers(page, size, sort));
    }

    @GetMapping("/{id}")
    @Operation(summary = "Get user by ID", description = "Retrieve a specific user by ID")
    @ApiResponse(responseCode = "200", description = "User found")
    @ApiResponse(responseCode = "404", description = "User not found")
    public ResponseEntity<User> getUserById(@PathVariable Long id) {
        return userService.getUser(id)
            .map(ResponseEntity::ok)
            .orElse(ResponseEntity.notFound().build());
    }

    @PostMapping
    @Operation(summary = "Create user", description = "Create a new user")
    @ApiResponse(responseCode = "201", description = "User created successfully")
    @ApiResponse(responseCode = "400", description = "Invalid input")
    public ResponseEntity<User> createUser(@Valid @RequestBody User user) {
        User created = userService.createUser(user);
        return ResponseEntity.status(HttpStatus.CREATED).body(created);
    }
}

// User.java with Swagger documentation
@Data
@Entity
@Schema(description = "User entity")
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Schema(description = "User unique identifier", example = "1")
    private Long id;

    @Column(nullable = false)
    @NotBlank(message = "Name is required")
    @Schema(description = "User full name", example = "John Doe")
    private String name;

    @Column(unique = true, nullable = false)
    @Email(message = "Email must be valid")
    @Schema(description = "User email address", example = "john@example.com")
    private String email;

    @Column(nullable = false)
    @NotBlank(message = "Password is required")
    @Schema(description = "User password", example = "password123")
    private String password;
}
```

Access Swagger UI: `http://localhost:8080/swagger-ui.html`

---

## 12.3 API Versioning

### URL-Based Versioning

```java
@RestController
@RequestMapping("/api/v1/users")
public class UserControllerV1 {

    @GetMapping("/{id}")
    public ResponseEntity<UserV1> getUser(@PathVariable Long id) {
        // V1 response
        return ResponseEntity.ok(new UserV1());
    }
}

@RestController
@RequestMapping("/api/v2/users")
public class UserControllerV2 {

    @GetMapping("/{id}")
    public ResponseEntity<UserV2> getUser(@PathVariable Long id) {
        // V2 response with additional fields
        return ResponseEntity.ok(new UserV2());
    }
}
```

### Header-Based Versioning

```java
@RestController
@RequestMapping("/api/users")
public class UserController {

    @GetMapping(value = "/{id}", headers = "API-Version=1")
    public ResponseEntity<UserV1> getUserV1(@PathVariable Long id) {
        return ResponseEntity.ok(new UserV1());
    }

    @GetMapping(value = "/{id}", headers = "API-Version=2")
    public ResponseEntity<UserV2> getUserV2(@PathVariable Long id) {
        return ResponseEntity.ok(new UserV2());
    }
}
```

---

## 12.4 CI/CD Pipeline with GitHub Actions

Create `.github/workflows/ci-cd.yml`:

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3

      - name: Set up JDK 17
        uses: actions/setup-java@v3
        with:
          java-version: "17"
          distribution: "temurin"

      - name: Build with Maven
        run: mvn clean package -DskipTests

      - name: Run Tests
        run: mvn test

      - name: Run Code Quality Check
        run: mvn spotbugs:check

      - name: Build Docker Image
        run: docker build -t user-service:${{ github.sha }} .

      - name: Push Docker Image
        run: |
          echo ${{ secrets.DOCKER_PASSWORD }} | docker login -u ${{ secrets.DOCKER_USERNAME }} --password-stdin
          docker tag user-service:${{ github.sha }} ${{ secrets.DOCKER_USERNAME }}/user-service:latest
          docker push ${{ secrets.DOCKER_USERNAME }}/user-service:latest

  deploy:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'

    steps:
      - name: Deploy to Production
        run: |
          # Deploy commands
          echo "Deploying to production..."
```

---

## 12.5 Deployment Strategies

### Docker Deployment

```dockerfile
# Multi-stage build
FROM maven:3.8-openjdk-17 AS builder
WORKDIR /app
COPY . .
RUN mvn clean package -DskipTests

FROM openjdk:17-jdk-slim
WORKDIR /app
COPY --from=builder /app/target/*.jar app.jar

ENV JAVA_OPTS="-Xmx512m -Xms256m -XX:+UseG1GC"
EXPOSE 8080

HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:8080/actuator/health || exit 1

ENTRYPOINT ["java", "-jar", "app.jar"]
```

### Kubernetes Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: user-service
spec:
  replicas: 3
  selector:
    matchLabels:
      app: user-service
  template:
    metadata:
      labels:
        app: user-service
    spec:
      containers:
        - name: user-service
          image: myregistry/user-service:latest
          ports:
            - containerPort: 8080
          env:
            - name: SPRING_PROFILES_ACTIVE
              value: "prod"
            - name: SPRING_DATASOURCE_URL
              valueFrom:
                secretKeyRef:
                  name: db-secret
                  key: url
            - name: SPRING_DATASOURCE_USERNAME
              valueFrom:
                secretKeyRef:
                  name: db-secret
                  key: username
            - name: SPRING_DATASOURCE_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: db-secret
                  key: password
          resources:
            requests:
              memory: "256Mi"
              cpu: "250m"
            limits:
              memory: "512Mi"
              cpu: "500m"
          livenessProbe:
            httpGet:
              path: /actuator/health
              port: 8080
            initialDelaySeconds: 30
            periodSeconds: 10
          readinessProbe:
            httpGet:
              path: /actuator/health
              port: 8080
            initialDelaySeconds: 10
            periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: user-service
spec:
  selector:
    app: user-service
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
  type: LoadBalancer
```

---

## 12.6 Production Checklist

```
✅ Security
  - Enable HTTPS/TLS
  - Implement authentication and authorization
  - Validate all inputs
  - Use security headers (CORS, HSTS, etc.)
  - Regular security audits

✅ Performance
  - Implement caching strategies
  - Use connection pooling
  - Monitor database queries
  - Load testing

✅ Reliability
  - Circuit breakers for external calls
  - Retry policies
  - Graceful shutdown
  - Health checks

✅ Observability
  - Structured logging
  - Distributed tracing
  - Metrics and monitoring
  - Alerting

✅ Deployment
  - Infrastructure as Code
  - Blue-Green deployment
  - Automated rollback capability
  - Documentation

✅ Data Management
  - Database backups
  - Migration strategy
  - Data validation
  - Disaster recovery plan

✅ Compliance
  - GDPR compliance
  - Data privacy
  - Audit logs
  - Compliance checks
```

---

## 12.7 12-Factor App Principles

```
1. Codebase - One codebase tracked in version control
2. Dependencies - Explicitly declare dependencies
3. Config - Store config in environment variables
4. Backing Services - Treat databases as attached resources
5. Build/Run/Release - Strict separation
6. Processes - Stateless and share-nothing
7. Port Binding - Self-contained web server
8. Concurrency - Horizontal scaling
9. Disposability - Fast startup and graceful shutdown
10. Dev/Prod Parity - Keep environments similar
11. Logs - Write to stdout/stderr
12. Admin Processes - Run as one-off tasks
```

### 12-Factor Compliant Configuration

```properties
# application.properties
spring.application.name=${APP_NAME:my-app}
server.port=${PORT:8080}

# Database
spring.datasource.url=${DATABASE_URL}
spring.datasource.username=${DATABASE_USER}
spring.datasource.password=${DATABASE_PASSWORD}

# Security
jwt.secret=${JWT_SECRET}
jwt.expiration=${JWT_EXPIRATION:86400000}

# Logging
logging.level.root=${LOG_LEVEL:INFO}
```

---

## 📚 Summary

In **Phase 12**, you learned:

- ✅ RESTful API design principles
- ✅ API documentation with Springdoc OpenAPI
- ✅ API versioning strategies
- ✅ CI/CD with GitHub Actions
- ✅ Docker deployment
- ✅ Kubernetes deployment
- ✅ Production checklist
- ✅ 12-Factor App principles

---

## 🎓 Congratulations! You've Completed the Spring Boot Roadmap! 🎉

---

## 📚 Next Steps for Continuous Learning

1. **Deep Dive Topics**:

   - Advanced Kubernetes
   - Service Mesh (Istio)
   - Advanced Kafka patterns
   - CQRS and Event Sourcing

2. **Practice Projects**:

   - Build a production-ready microservices platform
   - Implement real-time data processing pipeline
   - Create a machine learning integration

3. **Community Involvement**:

   - Contribute to Spring projects on GitHub
   - Write technical blogs
   - Participate in Spring Boot forums

4. **Stay Updated**:
   - Follow Spring Blog
   - Attend Spring Boot conferences
   - Read latest Spring releases

---

## 🌟 Key Takeaways

- Spring Boot simplifies enterprise Java development
- Always design APIs with REST principles
- Use proper testing strategies
- Monitor and observe production systems
- Follow 12-Factor App principles
- Keep security as top priority
- Document your APIs well
- Automate testing and deployment
- Scale horizontally
- Keep learning and evolving

---

## 📖 Recommended Resources

- **Official Docs**: https://spring.io/projects/spring-boot
- **Spring Academy**: https://spring.academy/
- **Baeldung Tutorials**: https://www.baeldung.com
- **Spring Blog**: https://spring.io/blog
- **GitHub**: https://github.com/spring-projects/spring-boot

---

**You're now ready to build enterprise-grade Spring Boot applications! 🚀**

**Happy Coding! 🎉**
