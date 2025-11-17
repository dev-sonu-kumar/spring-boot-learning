# Phase 6: Testing (Week 10)

## 📌 Overview

In this phase, you'll learn to write comprehensive unit and integration tests for your Spring Boot applications using JUnit, Mockito, and TestRestTemplate.

---

## 6.1 Unit Testing Basics

### Add Testing Dependencies

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>

<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-core</artifactId>
    <scope>test</scope>
</dependency>

<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-junit-jupiter</artifactId>
    <scope>test</scope>
</dependency>
```

### Basic Unit Test

```java
// Calculator.java
public class Calculator {
    public int add(int a, int b) {
        return a + b;
    }

    public int subtract(int a, int b) {
        return a - b;
    }

    public int multiply(int a, int b) {
        return a * b;
    }

    public int divide(int a, int b) {
        if (b == 0) {
            throw new IllegalArgumentException("Cannot divide by zero");
        }
        return a / b;
    }
}

// CalculatorTest.java
@ExtendWith(MockitoExtension.class)
public class CalculatorTest {

    private Calculator calculator;

    @BeforeEach
    public void setUp() {
        calculator = new Calculator();
    }

    @Test
    public void testAddition() {
        int result = calculator.add(5, 3);
        assertEquals(8, result);
    }

    @Test
    public void testSubtraction() {
        int result = calculator.subtract(10, 4);
        assertEquals(6, result);
    }

    @Test
    public void testMultiplication() {
        int result = calculator.multiply(4, 5);
        assertEquals(20, result);
    }

    @Test
    public void testDivisionByZero() {
        assertThrows(IllegalArgumentException.class, () -> {
            calculator.divide(10, 0);
        });
    }

    @Test
    @DisplayName("Division should return correct result")
    public void testDivision() {
        int result = calculator.divide(20, 4);
        assertEquals(5, result);
    }
}
```

---

## 6.2 Mocking with Mockito

### Mock Objects and Stubs

```java
// UserService.java
@Service
public class UserService {

    private UserRepository userRepository;
    private EmailService emailService;

    public UserService(UserRepository userRepository, EmailService emailService) {
        this.userRepository = userRepository;
        this.emailService = emailService;
    }

    public User registerUser(String email, String password) {
        User user = new User(email, password);
        User savedUser = userRepository.save(user);
        emailService.sendWelcomeEmail(email);
        return savedUser;
    }

    public Optional<User> getUserById(Long id) {
        return userRepository.findById(id);
    }
}

// UserServiceTest.java
@ExtendWith(MockitoExtension.class)
public class UserServiceTest {

    @Mock
    private UserRepository userRepository;

    @Mock
    private EmailService emailService;

    @InjectMocks
    private UserService userService;

    @Test
    public void testRegisterUser() {
        // Arrange
        User mockUser = new User("john@example.com", "password123");
        when(userRepository.save(any(User.class))).thenReturn(mockUser);

        // Act
        User result = userService.registerUser("john@example.com", "password123");

        // Assert
        assertNotNull(result);
        assertEquals("john@example.com", result.getEmail());

        // Verify email was sent
        verify(emailService, times(1)).sendWelcomeEmail("john@example.com");
        verify(userRepository, times(1)).save(any(User.class));
    }

    @Test
    public void testGetUserById() {
        // Arrange
        User mockUser = new User("jane@example.com", "password123");
        when(userRepository.findById(1L)).thenReturn(Optional.of(mockUser));

        // Act
        Optional<User> result = userService.getUserById(1L);

        // Assert
        assertTrue(result.isPresent());
        assertEquals("jane@example.com", result.get().getEmail());

        verify(userRepository, times(1)).findById(1L);
    }

    @Test
    public void testGetUserById_NotFound() {
        // Arrange
        when(userRepository.findById(999L)).thenReturn(Optional.empty());

        // Act
        Optional<User> result = userService.getUserById(999L);

        // Assert
        assertFalse(result.isPresent());
    }
}
```

### Argument Matchers

```java
@Test
public void testArgumentMatchers() {
    // any() - matches any argument
    when(userRepository.save(any(User.class))).thenReturn(new User());

    // anyString() - matches any string
    when(emailService.sendEmail(anyString(), anyString())).thenReturn(true);

    // eq() - exact match
    when(userRepository.findById(eq(1L))).thenReturn(Optional.empty());

    // ArgumentCaptor - capture arguments
    ArgumentCaptor<User> captor = ArgumentCaptor.forClass(User.class);
    verify(userRepository).save(captor.capture());
    User capturedUser = captor.getValue();
}
```

---

## 6.3 Integration Testing

### @SpringBootTest

```java
@SpringBootTest
public class UserServiceIntegrationTest {

    @Autowired
    private UserService userService;

    @Autowired
    private UserRepository userRepository;

    @BeforeEach
    public void setUp() {
        userRepository.deleteAll();
    }

    @Test
    public void testRegisterAndRetrieveUser() {
        // Register user
        User user = userService.registerUser("integration@test.com", "password");

        // Verify user was saved
        Optional<User> retrieved = userRepository.findByEmail("integration@test.com");
        assertTrue(retrieved.isPresent());
        assertEquals("integration@test.com", retrieved.get().getEmail());
    }
}
```

### @WebMvcTest for Controllers

```java
@WebMvcTest(UserController.class)
public class UserControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private UserService userService;

    @Test
    public void testGetAllUsers() throws Exception {
        List<User> users = Arrays.asList(
            new User("user1@test.com", "pass1"),
            new User("user2@test.com", "pass2")
        );

        when(userService.getAllUsers()).thenReturn(users);

        mockMvc.perform(get("/api/users"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.length()").value(2))
            .andExpect(jsonPath("$[0].email").value("user1@test.com"));
    }

    @Test
    public void testCreateUser() throws Exception {
        User user = new User("new@test.com", "password");
        when(userService.createUser(any(User.class))).thenReturn(user);

        mockMvc.perform(post("/api/users")
            .contentType(MediaType.APPLICATION_JSON)
            .content("{\"email\":\"new@test.com\",\"password\":\"password\"}"))
            .andExpect(status().isCreated())
            .andExpect(jsonPath("$.email").value("new@test.com"));
    }
}
```

### @DataJpaTest for Repositories

```java
@DataJpaTest
public class UserRepositoryTest {

    @Autowired
    private UserRepository userRepository;

    @Test
    public void testFindByEmail() {
        User user = new User("test@example.com", "password");
        userRepository.save(user);

        Optional<User> found = userRepository.findByEmail("test@example.com");

        assertTrue(found.isPresent());
        assertEquals("test@example.com", found.get().getEmail());
    }

    @Test
    public void testFindByEmail_NotFound() {
        Optional<User> found = userRepository.findByEmail("nonexistent@example.com");

        assertFalse(found.isPresent());
    }
}
```

---

## 6.4 Test Containers for Integration Tests

### Using Testcontainers

```xml
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>testcontainers</artifactId>
    <version>1.19.0</version>
    <scope>test</scope>
</dependency>

<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>mysql</artifactId>
    <version>1.19.0</version>
    <scope>test</scope>
</dependency>
```

### Test with MySQL Container

```java
@SpringBootTest
@Testcontainers
public class IntegrationWithMySQLTest {

    @Container
    static MySQLContainer<?> mysql = new MySQLContainer<>("mysql:8.0")
        .withDatabaseName("testdb")
        .withUsername("test")
        .withPassword("test");

    @DynamicPropertySource
    static void properties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", mysql::getJdbcUrl);
        registry.add("spring.datasource.username", mysql::getUsername);
        registry.add("spring.datasource.password", mysql::getPassword);
    }

    @Autowired
    private UserRepository userRepository;

    @Test
    public void testWithRealDatabase() {
        User user = new User("test@example.com", "password");
        userRepository.save(user);

        Optional<User> found = userRepository.findByEmail("test@example.com");
        assertTrue(found.isPresent());
    }
}
```

---

## 6.5 Test Organization and Best Practices

### Test Class Structure

```java
@ExtendWith(MockitoExtension.class)
public class CompleteServiceTest {

    @Mock
    private UserRepository userRepository;

    @InjectMocks
    private UserService userService;

    // Given - Setup
    @BeforeEach
    public void setUp() {
        // Initialize test data
    }

    // When - Execute
    // Then - Assert
    @Nested
    @DisplayName("User Creation Tests")
    class UserCreationTests {

        @Test
        @DisplayName("Should create user with valid data")
        public void shouldCreateUserWithValidData() {
            // Arrange
            User user = new User("test@example.com", "password");
            when(userRepository.save(any())).thenReturn(user);

            // Act
            User created = userService.createUser(user);

            // Assert
            assertNotNull(created);
            assertEquals("test@example.com", created.getEmail());
        }
    }

    @Nested
    @DisplayName("User Retrieval Tests")
    class UserRetrievalTests {

        @Test
        public void shouldRetrieveUserById() {
            // Test implementation
        }
    }
}
```

---

## 📚 Summary

In **Phase 6**, you learned:

- ✅ JUnit 5 basics and assertions
- ✅ Mockito for mocking dependencies
- ✅ @WebMvcTest for controller testing
- ✅ @DataJpaTest for repository testing
- ✅ @SpringBootTest for integration testing
- ✅ TestContainers for database testing
- ✅ Test organization and best practices

---

**Ready to optimize your application? Let's move to Phase 7! 🚀**
