# Phase 1: Fundamentals (Week 1-2)

## 📌 Overview

In this phase, you'll learn the core concepts that form the foundation of Spring Boot development. We'll cover Spring Boot basics, Dependency Injection, and essential annotations.

---

## 1.1 Spring Boot Basics

### What is Spring Boot?

Spring Boot is a **framework that simplifies building stand-alone, production-grade Spring applications**. It's built on top of the Spring Framework and removes boilerplate code.

#### Key Characteristics:

```
✅ Auto-configuration: Automatically configures Spring application
✅ Embedded Servers: Tomcat, Jetty, or Undertow embedded
✅ Production Ready: Includes metrics, health checks, monitoring
✅ Convention over Configuration: Sensible defaults
✅ Minimal XML: Little to no XML configuration needed
```

### Spring Boot vs Spring Framework

| Feature            | Spring Framework     | Spring Boot                     |
| ------------------ | -------------------- | ------------------------------- |
| **Setup Time**     | 30-40 minutes        | 5-10 minutes                    |
| **Configuration**  | XML/Java based       | Auto-configuration + properties |
| **Dependencies**   | Manual specification | Starter POMs                    |
| **Server**         | External (Tomcat)    | Embedded                        |
| **Monitoring**     | Manual setup         | Built-in Actuator               |
| **Learning Curve** | Moderate             | Beginner-friendly               |

### Why Choose Spring Boot?

1. **Speed**: Rapid application development
2. **Productivity**: Less boilerplate code
3. **Flexibility**: Easy to override defaults
4. **Integration**: Works seamlessly with microservices
5. **Community**: Large and active community

---

## 1.2 Setting Up Your First Spring Boot Project

### Prerequisites

- **Java**: JDK 11+ (preferably JDK 17 or 21)
- **Build Tool**: Maven 3.6+ or Gradle 7.0+
- **IDE**: IntelliJ IDEA (recommended), Eclipse, or VS Code
- **Git**: For version control

### Method 1: Using Spring Initializr (Recommended)

**Step 1:** Visit https://start.spring.io

**Step 2:** Configure your project:

```
- Project: Maven
- Language: Java
- Spring Boot Version: 3.1.5 (or latest)
- Group: com.example
- Artifact: my-spring-app
- Package Name: com.example.demo
- Packaging: Jar
- Java Version: 17
```

**Step 3:** Add dependencies:

- Spring Web
- Spring Boot DevTools
- Lombok (optional, for less boilerplate)

**Step 4:** Click "Generate" and extract the ZIP file

### Method 2: Manual Setup with Maven

**1. Create `pom.xml`:**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>my-spring-app</artifactId>
    <version>1.0.0</version>
    <packaging>jar</packaging>

    <name>My Spring App</name>
    <description>My First Spring Boot Application</description>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.1.5</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <properties>
        <java.version>17</java.version>
    </properties>

    <dependencies>
        <!-- Spring Web Starter -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- Spring Boot DevTools (for hot reload) -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>

        <!-- Testing -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
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

**2. Create folder structure:**

```
my-spring-app/
├── src/
│   ├── main/
│   │   ├── java/com/example/demo/
│   │   │   └── DemoApplication.java
│   │   └── resources/
│   │       └── application.properties
│   └── test/
│       └── java/com/example/demo/
│           └── DemoApplicationTests.java
└── pom.xml
```

**3. Create main class:**

```java
package com.example.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class DemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }

}
```

### Running Your First Application

```bash
# Using Maven
mvn spring-boot:run

# Or build and run JAR
mvn clean package
java -jar target/my-spring-app-1.0.0.jar
```

**Output:**

```
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_|\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::                (v3.1.5)

2024-11-17 10:30:00.000  INFO 1234 --- [main] c.e.demo.DemoApplication : Started DemoApplication
```

✅ Your Spring Boot app is now running!

---

## 1.3 Project Structure & Files Explained

### Key Files in a Spring Boot Project

#### 1. **application.properties** (or application.yml)

Located in `src/main/resources/`, this file contains application configuration:

```properties
# Server Configuration
server.port=8080
server.servlet.context-path=/api

# Application Name
spring.application.name=my-spring-app

# Logging
logging.level.root=INFO
logging.level.com.example.demo=DEBUG

# Data Source (if using database)
spring.datasource.url=jdbc:mysql://localhost:3306/mydb
spring.datasource.username=root
spring.datasource.password=password
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
```

#### 2. **application.yml** (Alternative format)

Same as properties but in YAML format:

```yaml
server:
  port: 8080
  servlet:
    context-path: /api

spring:
  application:
    name: my-spring-app
  datasource:
    url: jdbc:mysql://localhost:3306/mydb
    username: root
    password: password
    driver-class-name: com.mysql.cj.jdbc.Driver

logging:
  level:
    root: INFO
    com.example.demo: DEBUG
```

#### 3. **POM.xml** (Project Object Model)

Defines project dependencies and build configuration.

#### 4. **DemoApplication.java** (Main Class)

Entry point of your Spring Boot application. Contains `@SpringBootApplication` annotation.

---

## 1.4 Understanding @SpringBootApplication

The `@SpringBootApplication` annotation is a composite annotation that combines three important annotations:

```java
@SpringBootApplication
public class DemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

### Equivalent to:

```java
@Configuration
@EnableAutoConfiguration
@ComponentScan
public class DemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

### What each annotation does:

| Annotation                   | Purpose                                                  |
| ---------------------------- | -------------------------------------------------------- |
| **@Configuration**           | Marks class as configuration source; can define beans    |
| **@EnableAutoConfiguration** | Auto-configures Spring based on classpath dependencies   |
| **@ComponentScan**           | Scans for @Component, @Service, @Repository, @Controller |

---

## 1.5 Dependency Injection & IoC Container

### What is Inversion of Control (IoC)?

**Traditional Approach (Without IoC):**

```java
public class UserService {
    private UserRepository userRepository;

    public UserService() {
        // We create the dependency ourselves
        this.userRepository = new UserRepository();
    }
}
```

**Problem:**

- Tightly coupled code
- Hard to test (can't mock UserRepository)
- Difficult to manage dependencies

### IoC Approach (Spring Way):

```java
public class UserService {
    private UserRepository userRepository;

    // Spring creates and injects the dependency
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}
```

**Benefit:**

- Loose coupling
- Easy to test (can inject mock objects)
- Spring Container manages dependencies
- Flexible and maintainable

### How Spring IoC Container Works

```
1. Application Starts
   ↓
2. Spring Scans for Components (@Component, @Service, etc.)
   ↓
3. Creates Bean Instances
   ↓
4. Stores Beans in IoC Container
   ↓
5. Injects Dependencies where needed (@Autowired)
   ↓
6. Application Ready to Use!
```

---

## 1.6 Core Annotations

### Class-Level Annotations

#### 1. **@Component**

Generic annotation to declare a class as a Spring component.

```java
@Component
public class EmailService {
    public void sendEmail(String to, String message) {
        System.out.println("Sending email to: " + to);
        System.out.println("Message: " + message);
    }
}
```

#### 2. **@Service**

Specialization of @Component for business logic/service layer.

```java
@Service
public class UserService {
    private UserRepository userRepository;

    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    public void registerUser(String name, String email) {
        // Business logic
    }
}
```

#### 3. **@Repository**

Specialization of @Component for data access layer.

```java
@Repository
public class UserRepository {
    public User findById(Long id) {
        // Database logic
        return null;
    }
}
```

#### 4. **@Controller**

For handling HTTP requests (returns views).

```java
@Controller
public class UserController {
    @GetMapping("/users")
    public String listUsers(Model model) {
        // Return view name
        return "users/list";
    }
}
```

#### 5. **@RestController**

For REST API endpoints (returns JSON/XML).

```java
@RestController
public class UserRestController {
    @GetMapping("/api/users")
    public List<User> getAllUsers() {
        return users;
    }
}
```

#### 6. **@Configuration**

Declares a class as a configuration class for defining beans.

```java
@Configuration
public class AppConfig {
    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```

### Field-Level Annotations

#### 1. **@Autowired**

Requests Spring to inject a dependency.

```java
@Service
public class UserService {
    @Autowired
    private UserRepository userRepository;

    // OR Constructor Injection (Recommended)
    private UserRepository userRepository;

    @Autowired
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}
```

#### 2. **@Qualifier**

Specifies which bean to inject when multiple exist.

```java
@Service
public class UserService {
    @Autowired
    @Qualifier("mysqlUserRepository")
    private UserRepository userRepository;
}
```

#### 3. **@Primary**

Marks a bean as the default choice for injection.

```java
@Configuration
public class AppConfig {
    @Bean
    @Primary
    public UserRepository mysqlUserRepository() {
        return new MySQLUserRepository();
    }

    @Bean
    public UserRepository mongoUserRepository() {
        return new MongoUserRepository();
    }
}
```

### Method-Level Annotations

#### **@Bean**

Declares a method that produces a bean.

```java
@Configuration
public class AppConfig {
    @Bean
    public DataSource dataSource() {
        return new DataSource();
    }

    @Bean(name = "customRestTemplate")
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```

---

## 1.7 Bean Scope

Defines the lifecycle and visibility of a bean in the container.

### 1. **Singleton (Default)**

One instance for entire application.

```java
@Component
@Scope("singleton")
public class SingletonService {
    // Only one instance created
}
```

### 2. **Prototype**

New instance created every time.

```java
@Component
@Scope("prototype")
public class PrototypeService {
    // New instance for each request
}
```

### 3. **Request**

New instance per HTTP request.

```java
@Component
@Scope("request")
public class RequestService {
    // One instance per request
}
```

### 4. **Session**

One instance per user session.

```java
@Component
@Scope("session")
public class SessionService {
    // One instance per session
}
```

### Example: Singleton vs Prototype

```java
@Service
@Scope("singleton")
public class SingletonCounter {
    private int count = 0;

    public void increment() {
        count++;
    }

    public int getCount() {
        return count;
    }
}

@Service
@Scope("prototype")
public class PrototypeCounter {
    private int count = 0;

    public void increment() {
        count++;
    }

    public int getCount() {
        return count;
    }
}

// Usage
@RestController
public class TestController {
    @Autowired
    private SingletonCounter singletonCounter;

    @Autowired
    private PrototypeCounter prototypeCounter;

    @GetMapping("/test")
    public String test() {
        singletonCounter.increment();
        prototypeCounter.increment();

        // All requests will share same SingletonCounter (count keeps increasing)
        // Each request gets new PrototypeCounter (count always 1)
        return "Singleton: " + singletonCounter.getCount() +
               ", Prototype: " + prototypeCounter.getCount();
    }
}
```

---

## 1.8 Practical Example: Complete Application

Let's build a simple User Management application.

### Step 1: Create Models

```java
// User.java
package com.example.demo.model;

public class User {
    private Long id;
    private String name;
    private String email;

    // Constructor
    public User(Long id, String name, String email) {
        this.id = id;
        this.name = name;
        this.email = email;
    }

    // Getters and Setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }

    public String getName() { return name; }
    public void setName(String name) { this.name = name; }

    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }
}
```

### Step 2: Create Repository

```java
// UserRepository.java
package com.example.demo.repository;

import com.example.demo.model.User;
import org.springframework.stereotype.Repository;
import java.util.*;

@Repository
public class UserRepository {
    private Map<Long, User> users = new HashMap<>();
    private Long idCounter = 1L;

    public User save(User user) {
        user.setId(idCounter++);
        users.put(user.getId(), user);
        return user;
    }

    public Optional<User> findById(Long id) {
        return Optional.ofNullable(users.get(id));
    }

    public List<User> findAll() {
        return new ArrayList<>(users.values());
    }

    public void delete(Long id) {
        users.remove(id);
    }
}
```

### Step 3: Create Service

```java
// UserService.java
package com.example.demo.service;

import com.example.demo.model.User;
import com.example.demo.repository.UserRepository;
import org.springframework.stereotype.Service;
import java.util.List;
import java.util.Optional;

@Service
public class UserService {
    private UserRepository userRepository;

    // Constructor Injection
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    public User createUser(String name, String email) {
        User user = new User(null, name, email);
        return userRepository.save(user);
    }

    public Optional<User> getUser(Long id) {
        return userRepository.findById(id);
    }

    public List<User> getAllUsers() {
        return userRepository.findAll();
    }

    public void deleteUser(Long id) {
        userRepository.delete(id);
    }
}
```

### Step 4: Create REST Controller

```java
// UserController.java
package com.example.demo.controller;

import com.example.demo.model.User;
import com.example.demo.service.UserService;
import org.springframework.web.bind.annotation.*;
import java.util.List;
import java.util.Optional;

@RestController
@RequestMapping("/api/users")
public class UserController {
    private UserService userService;

    public UserController(UserService userService) {
        this.userService = userService;
    }

    @GetMapping
    public List<User> getAllUsers() {
        return userService.getAllUsers();
    }

    @GetMapping("/{id}")
    public Optional<User> getUserById(@PathVariable Long id) {
        return userService.getUser(id);
    }

    @PostMapping
    public User createUser(@RequestBody User user) {
        return userService.createUser(user.getName(), user.getEmail());
    }

    @DeleteMapping("/{id}")
    public void deleteUser(@PathVariable Long id) {
        userService.deleteUser(id);
    }
}
```

### Step 5: Test the Application

```bash
# Start the application
mvn spring-boot:run

# Create a user
curl -X POST http://localhost:8080/api/users \
  -H "Content-Type: application/json" \
  -d '{"name":"John Doe","email":"john@example.com"}'

# Get all users
curl http://localhost:8080/api/users

# Get user by ID
curl http://localhost:8080/api/users/1

# Delete user
curl -X DELETE http://localhost:8080/api/users/1
```

---

## 📚 Summary

In **Phase 1**, you learned:

- ✅ What Spring Boot is and why use it
- ✅ Setting up your first project using Spring Initializr
- ✅ Understanding IoC and Dependency Injection
- ✅ Core Spring annotations (@Component, @Service, @Repository, etc.)
- ✅ Bean scopes and lifecycle
- ✅ Building a complete User Management application

---

## 🎯 Next Steps

1. **Practice**: Create variations of the User Management app
2. **Explore**: Try different dependency injection methods
3. **Code**: Build a Product Management system using what you learned
4. **Ready for Phase 2**: Web Development - REST APIs

---

## ❓ Practice Questions

1. What's the difference between @Component and @Service?
2. How does IoC Container create and manage beans?
3. What are the benefits of Constructor Injection over Field Injection?
4. When would you use @Scope("prototype") instead of @Scope("singleton")?
5. How does @Autowired find the correct bean to inject?

**Solutions**: Try implementing these concepts in your own code!

---

## 📖 Additional Resources

- [Spring Framework Official Docs](https://spring.io/projects/spring-framework)
- [Spring Boot Auto Configuration](https://docs.spring.io/spring-boot/docs/current/reference/html/using.html#using.auto-configuration)
- [Spring Bean Scopes](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes)

---

**Ready for Phase 2? Let's build REST APIs! 🚀**
