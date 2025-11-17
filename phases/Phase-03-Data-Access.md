# Phase 3: Data Access & Persistence (Week 5-6)

## 📌 Overview

In this phase, you'll learn how to persist data using JPA/Hibernate and Spring Data JPA. We'll cover entity relationships, database configuration, and query methods.

---

## 3.1 JPA (Java Persistence API) Fundamentals

### What is JPA?

**JPA** (Java Persistence API) is a standard interface for accessing databases in Java applications. It abstracts away database-specific details and provides a consistent API.

### ORM (Object-Relational Mapping)

ORM is a technique to map Java objects to database tables:

```
Java Classes ←→ ORM Framework ←→ Database Tables
   (Objects)    (Hibernate)      (Rows)
```

### Benefits of JPA/ORM

- **Abstraction**: Work with objects instead of SQL
- **Portability**: Switch databases without code changes
- **Productivity**: Less boilerplate code
- **Maintainability**: Business logic separated from persistence logic

---

## 3.2 Setting Up Database & Dependencies

### Add Dependencies to pom.xml

```xml
<!-- Spring Data JPA -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>

<!-- MySQL Database -->
<dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.33</version>
</dependency>

<!-- OR H2 Database (for development/testing) -->
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>runtime</scope>
</dependency>

<!-- Lombok (optional) -->
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
</dependency>
```

### Configure application.properties

```properties
# MySQL Configuration
spring.datasource.url=jdbc:mysql://localhost:3306/springdb
spring.datasource.username=root
spring.datasource.password=password
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

# JPA/Hibernate Configuration
spring.jpa.database-platform=org.hibernate.dialect.MySQL8Dialect
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true

# Logging
logging.level.org.hibernate.SQL=DEBUG
logging.level.org.hibernate.type.descriptor.sql.BasicBinder=TRACE
```

### Or application.yml

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/springdb
    username: root
    password: password
    driver-class-name: com.mysql.cj.jdbc.Driver

  jpa:
    database-platform: org.hibernate.dialect.MySQL8Dialect
    hibernate:
      ddl-auto: update
    show-sql: true
    properties:
      hibernate:
        format_sql: true

logging:
  level:
    org.hibernate.SQL: DEBUG
    org.hibernate.type.descriptor.sql.BasicBinder: TRACE
```

---

## 3.3 Entity Classes and Annotations

### Basic Entity

```java
package com.example.demo.entity;

import jakarta.persistence.*;
import jakarta.validation.constraints.*;

@Entity
@Table(name = "users")
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "first_name", nullable = false, length = 100)
    @NotBlank(message = "First name is required")
    private String firstName;

    @Column(name = "last_name", nullable = false, length = 100)
    @NotBlank(message = "Last name is required")
    private String lastName;

    @Column(unique = true, nullable = false)
    @Email(message = "Email should be valid")
    private String email;

    @Column(name = "created_at", nullable = false, updatable = false)
    @Temporal(TemporalType.TIMESTAMP)
    private LocalDateTime createdAt;

    @Column(name = "updated_at")
    @Temporal(TemporalType.TIMESTAMP)
    private LocalDateTime updatedAt;

    // Default Constructor
    public User() {}

    // Constructor with parameters
    public User(String firstName, String lastName, String email) {
        this.firstName = firstName;
        this.lastName = lastName;
        this.email = email;
        this.createdAt = LocalDateTime.now();
    }

    // Lifecycle callbacks
    @PrePersist
    protected void onCreate() {
        createdAt = LocalDateTime.now();
    }

    @PreUpdate
    protected void onUpdate() {
        updatedAt = LocalDateTime.now();
    }

    // Getters and Setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }

    public String getFirstName() { return firstName; }
    public void setFirstName(String firstName) { this.firstName = firstName; }

    public String getLastName() { return lastName; }
    public void setLastName(String lastName) { this.lastName = lastName; }

    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }

    public LocalDateTime getCreatedAt() { return createdAt; }
    public LocalDateTime getUpdatedAt() { return updatedAt; }
}
```

### Entity Annotations Explained

| Annotation          | Purpose                                              |
| ------------------- | ---------------------------------------------------- |
| **@Entity**         | Marks class as JPA entity (mapped to database table) |
| **@Table**          | Specifies table name and constraints                 |
| **@Id**             | Marks field as primary key                           |
| **@GeneratedValue** | Auto-generates primary key                           |
| **@Column**         | Defines column mapping and constraints               |
| **@Temporal**       | Specifies temporal type (DATE, TIME, TIMESTAMP)      |
| **@PrePersist**     | Lifecycle callback before INSERT                     |
| **@PostPersist**    | Lifecycle callback after INSERT                      |
| **@PreUpdate**      | Lifecycle callback before UPDATE                     |
| **@PostUpdate**     | Lifecycle callback after UPDATE                      |
| **@PreRemove**      | Lifecycle callback before DELETE                     |
| **@PostRemove**     | Lifecycle callback after DELETE                      |

---

## 3.4 Entity Relationships

### One-to-Many Relationship

```java
// Department.java
@Entity
@Table(name = "departments")
public class Department {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String name;

    // One department has many employees
    @OneToMany(mappedBy = "department", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Employee> employees = new ArrayList<>();

    // Constructors, getters, setters...
    public Department() {}
    public Department(String name) { this.name = name; }

    public Long getId() { return id; }
    public String getName() { return name; }
    public List<Employee> getEmployees() { return employees; }

    public void addEmployee(Employee employee) {
        employees.add(employee);
        employee.setDepartment(this);
    }
}

// Employee.java
@Entity
@Table(name = "employees")
public class Employee {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String name;

    // Many employees belong to one department
    @ManyToOne
    @JoinColumn(name = "department_id", nullable = false)
    private Department department;

    // Constructors, getters, setters...
    public Employee() {}
    public Employee(String name, Department department) {
        this.name = name;
        this.department = department;
    }

    public Long getId() { return id; }
    public String getName() { return name; }
    public Department getDepartment() { return department; }
    public void setDepartment(Department department) { this.department = department; }
}
```

### Many-to-Many Relationship

```java
// Student.java
@Entity
@Table(name = "students")
public class Student {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String name;

    // Many students can enroll in many courses
    @ManyToMany(cascade = CascadeType.PERSIST)
    @JoinTable(
        name = "student_course",
        joinColumns = @JoinColumn(name = "student_id"),
        inverseJoinColumns = @JoinColumn(name = "course_id")
    )
    private Set<Course> courses = new HashSet<>();

    // Constructors, getters, setters...
    public void addCourse(Course course) {
        courses.add(course);
        course.getStudents().add(this);
    }
}

// Course.java
@Entity
@Table(name = "courses")
public class Course {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String title;

    @ManyToMany(mappedBy = "courses")
    private Set<Student> students = new HashSet<>();

    // Constructors, getters, setters...
}
```

### One-to-One Relationship

```java
// User.java
@Entity
@Table(name = "users")
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String email;

    @OneToOne(cascade = CascadeType.ALL)
    @JoinColumn(name = "profile_id", referencedColumnName = "id")
    private Profile profile;

    // Constructors, getters, setters...
}

// Profile.java
@Entity
@Table(name = "profiles")
public class Profile {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String bio;

    @OneToOne(mappedBy = "profile")
    private User user;

    // Constructors, getters, setters...
}
```

---

## 3.5 Spring Data JPA

### Creating Repository Interface

```java
package com.example.demo.repository;

import com.example.demo.entity.User;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;
import java.util.List;
import java.util.Optional;

public interface UserRepository extends JpaRepository<User, Long> {

    // Method names automatically generate queries
    Optional<User> findByEmail(String email);

    List<User> findByFirstName(String firstName);

    List<User> findByLastNameIgnoreCase(String lastName);

    List<User> findByFirstNameAndLastName(String firstName, String lastName);

    // Custom JPQL query
    @Query("SELECT u FROM User u WHERE u.email = :email")
    Optional<User> findUserByEmail(@Param("email") String email);

    // Native SQL query
    @Query(value = "SELECT * FROM users WHERE email LIKE %:email%", nativeQuery = true)
    List<User> searchByEmail(@Param("email") String email);

    // Deletion
    void deleteByEmail(String email);
}
```

### Common Query Methods

| Method Name                                        | Equivalent SQL                                               |
| -------------------------------------------------- | ------------------------------------------------------------ |
| `findById(Long id)`                                | `SELECT * FROM users WHERE id = ?`                           |
| `findAll()`                                        | `SELECT * FROM users`                                        |
| `findByEmail(String email)`                        | `SELECT * FROM users WHERE email = ?`                        |
| `findByFirstName(String firstName)`                | `SELECT * FROM users WHERE first_name = ?`                   |
| `findByFirstNameIgnoreCase(String firstName)`      | `SELECT * FROM users WHERE LOWER(first_name) = LOWER(?)`     |
| `findByFirstNameAndLastName(String fn, String ln)` | `SELECT * FROM users WHERE first_name = ? AND last_name = ?` |
| `findByEmailContaining(String email)`              | `SELECT * FROM users WHERE email LIKE %?%`                   |
| `deleteById(Long id)`                              | `DELETE FROM users WHERE id = ?`                             |

---

## 3.6 Service Layer with JPA

```java
package com.example.demo.service;

import com.example.demo.entity.User;
import com.example.demo.repository.UserRepository;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
import java.util.List;
import java.util.Optional;

@Service
@Transactional
public class UserService {

    private UserRepository userRepository;

    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    public User createUser(User user) {
        return userRepository.save(user);
    }

    public Optional<User> getUserById(Long id) {
        return userRepository.findById(id);
    }

    public List<User> getAllUsers() {
        return userRepository.findAll();
    }

    public Optional<User> getUserByEmail(String email) {
        return userRepository.findByEmail(email);
    }

    public User updateUser(Long id, User userDetails) {
        return userRepository.findById(id)
            .map(user -> {
                user.setFirstName(userDetails.getFirstName());
                user.setLastName(userDetails.getLastName());
                user.setEmail(userDetails.getEmail());
                return userRepository.save(user);
            })
            .orElseThrow(() -> new RuntimeException("User not found"));
    }

    public void deleteUser(Long id) {
        userRepository.deleteById(id);
    }

    public List<User> searchByFirstName(String firstName) {
        return userRepository.findByFirstName(firstName);
    }
}
```

---

## 3.7 Database Initialization

### Using schema.sql and data.sql

Create files in `src/main/resources/`:

**schema.sql:**

```sql
CREATE TABLE IF NOT EXISTS departments (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL UNIQUE
);

CREATE TABLE IF NOT EXISTS employees (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    salary DECIMAL(10, 2),
    department_id BIGINT NOT NULL,
    FOREIGN KEY (department_id) REFERENCES departments(id)
);
```

**data.sql:**

```sql
INSERT INTO departments (name) VALUES ('IT');
INSERT INTO departments (name) VALUES ('HR');
INSERT INTO departments (name) VALUES ('Finance');

INSERT INTO employees (name, salary, department_id) VALUES ('John Doe', 50000, 1);
INSERT INTO employees (name, salary, department_id) VALUES ('Jane Smith', 45000, 1);
INSERT INTO employees (name, salary, department_id) VALUES ('Bob Johnson', 40000, 2);
```

### Configure in application.properties

```properties
spring.sql.init.mode=always
spring.sql.init.data-locations=classpath:data.sql
```

---

## 📚 Summary

In **Phase 3**, you learned:

- ✅ JPA fundamentals and ORM concepts
- ✅ Entity mapping and annotations
- ✅ Entity relationships (One-to-Many, Many-to-Many, One-to-One)
- ✅ Spring Data JPA repository
- ✅ Custom queries with @Query
- ✅ Database initialization with SQL files

---

## 🎯 Next Steps

1. **Practice**: Build a complete entity model with relationships
2. **Experiment**: Try different query methods
3. **Optimize**: Learn about N+1 problem and fetch strategies
4. **Ready for Phase 4**: Advanced Data Access & Transactions

---

**Ready to master advanced queries and transactions? Let's move to Phase 4! 🚀**
