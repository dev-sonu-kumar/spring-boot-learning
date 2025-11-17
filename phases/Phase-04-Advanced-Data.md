# Phase 4: Advanced Data Access (Week 7)

## 📌 Overview

In this phase, you'll learn advanced querying techniques, transaction management, and performance optimization in Spring Boot applications.

---

## 4.1 Advanced Query Techniques

### Named Queries

**Using @NamedQuery:**

```java
@Entity
@Table(name = "users")
@NamedQuery(
    name = "User.findByEmail",
    query = "SELECT u FROM User u WHERE u.email = :email"
)
@NamedQuery(
    name = "User.findByFirstNameAndLastName",
    query = "SELECT u FROM User u WHERE u.firstName = :firstName AND u.lastName = :lastName"
)
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String firstName;
    private String lastName;
    private String email;

    // Getters and setters...
}
```

**Using in Repository:**

```java
public interface UserRepository extends JpaRepository<User, Long> {
    @Query(name = "User.findByEmail")
    Optional<User> findByEmail(@Param("email") String email);
}
```

### Native Queries

```java
public interface ProductRepository extends JpaRepository<Product, Long> {

    // Simple native query
    @Query(value = "SELECT * FROM products WHERE price > :minPrice", nativeQuery = true)
    List<Product> findByMinPrice(@Param("minPrice") Double minPrice);

    // Native query with JOIN
    @Query(value = """
        SELECT p.* FROM products p
        JOIN categories c ON p.category_id = c.id
        WHERE c.name = :categoryName
        """, nativeQuery = true)
    List<Product> findByCategory(@Param("categoryName") String categoryName);

    // Native query returning custom result
    @Query(value = """
        SELECT new map(p.id as id, p.name as name, COUNT(o.id) as orderCount)
        FROM products p
        LEFT JOIN orders o ON p.id = o.product_id
        GROUP BY p.id, p.name
        """, nativeQuery = true)
    List<Map<String, Object>> getProductStats();
}
```

### Specifications and Predicates

```java
// ProductSpecification.java
package com.example.demo.specification;

import com.example.demo.entity.Product;
import org.springframework.data.jpa.domain.Specification;

public class ProductSpecification {

    public static Specification<Product> withPrice(Double price) {
        return (root, query, criteriaBuilder) ->
            criteriaBuilder.greaterThanOrEqualTo(root.get("price"), price);
    }

    public static Specification<Product> withCategory(String category) {
        return (root, query, criteriaBuilder) ->
            criteriaBuilder.equal(root.get("category"), category);
    }

    public static Specification<Product> withNameContaining(String name) {
        return (root, query, criteriaBuilder) ->
            criteriaBuilder.like(root.get("name"), "%" + name + "%");
    }
}

// ProductRepository.java
public interface ProductRepository extends JpaRepository<Product, Long>, JpaSpecificationExecutor<Product> {
}

// Usage in Service
@Service
public class ProductService {

    private ProductRepository productRepository;

    public List<Product> searchProducts(String name, String category, Double minPrice) {
        Specification<Product> spec = Specification
            .where(ProductSpecification.withNameContaining(name))
            .and(ProductSpecification.withCategory(category))
            .and(ProductSpecification.withPrice(minPrice));

        return productRepository.findAll(spec);
    }
}
```

### Custom Repository Implementation

```java
// ProductRepositoryCustom.java
public interface ProductRepositoryCustom {
    List<Product> findProductsByComplexCriteria(String name, String category, Double price);
}

// ProductRepositoryImpl.java
@Repository
public class ProductRepositoryImpl implements ProductRepositoryCustom {

    @PersistenceContext
    private EntityManager entityManager;

    @Override
    public List<Product> findProductsByComplexCriteria(String name, String category, Double price) {
        CriteriaBuilder cb = entityManager.getCriteriaBuilder();
        CriteriaQuery<Product> query = cb.createQuery(Product.class);
        Root<Product> root = query.from(Product.class);

        List<Predicate> predicates = new ArrayList<>();

        if (name != null && !name.isEmpty()) {
            predicates.add(cb.like(root.get("name"), "%" + name + "%"));
        }

        if (category != null && !category.isEmpty()) {
            predicates.add(cb.equal(root.get("category"), category));
        }

        if (price != null) {
            predicates.add(cb.greaterThanOrEqualTo(root.get("price"), price));
        }

        query.where(cb.and(predicates.toArray(new Predicate[0])));

        return entityManager.createQuery(query).getResultList();
    }
}

// ProductRepository.java extending custom interface
public interface ProductRepository extends JpaRepository<Product, Long>, ProductRepositoryCustom {
}
```

### Pagination and Sorting

```java
// Repository with pagination
public interface UserRepository extends JpaRepository<User, Long> {
    Page<User> findByFirstName(String firstName, Pageable pageable);
    Page<User> findAll(Pageable pageable);
}

// Service using pagination
@Service
public class UserService {

    private UserRepository userRepository;

    public Page<User> getAllUsers(int page, int size, String sort) {
        Pageable pageable = PageRequest.of(page, size, Sort.by(sort));
        return userRepository.findAll(pageable);
    }

    public Page<User> searchUsers(String firstName, int page, int size) {
        Pageable pageable = PageRequest.of(page, size, Sort.by("lastName").ascending());
        return userRepository.findByFirstName(firstName, pageable);
    }
}

// Controller
@RestController
@RequestMapping("/api/users")
public class UserController {

    private UserService userService;

    @GetMapping
    public Page<User> getUsers(
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "10") int size,
        @RequestParam(defaultValue = "id") String sort
    ) {
        return userService.getAllUsers(page, size, sort);
    }
}
```

---

## 4.2 Transaction Management

### @Transactional Annotation

```java
@Service
public class TransferService {

    private AccountRepository accountRepository;

    // With default settings
    @Transactional
    public void transferMoney(Long fromId, Long toId, Double amount) {
        Account from = accountRepository.findById(fromId)
            .orElseThrow(() -> new RuntimeException("Account not found"));
        Account to = accountRepository.findById(toId)
            .orElseThrow(() -> new RuntimeException("Account not found"));

        from.setBalance(from.getBalance() - amount);
        to.setBalance(to.getBalance() + amount);

        accountRepository.save(from);
        accountRepository.save(to);
    }

    // Rollback on specific exceptions
    @Transactional(rollbackFor = {RuntimeException.class, IOException.class})
    public void transferWithValidation(Long fromId, Long toId, Double amount) {
        // Business logic...
    }

    // Don't rollback on specific exceptions
    @Transactional(noRollbackFor = {ValidationException.class})
    public void transferWithWarning(Long fromId, Long toId, Double amount) {
        // Business logic...
    }

    // Read-only transaction (optimization)
    @Transactional(readOnly = true)
    public Account getAccount(Long id) {
        return accountRepository.findById(id).orElse(null);
    }

    // Propagation levels
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void logTransaction(String details) {
        // Always create new transaction
    }

    @Transactional(propagation = Propagation.NESTED)
    public void savePoint(String data) {
        // Create savepoint
    }
}
```

### Propagation Levels

| Level                  | Behavior                                             |
| ---------------------- | ---------------------------------------------------- |
| **REQUIRED** (default) | Use existing transaction or create new               |
| **REQUIRES_NEW**       | Always create new transaction                        |
| **NESTED**             | Create savepoint if inside transaction               |
| **SUPPORTS**           | Use transaction if exists, otherwise proceed without |
| **NOT_SUPPORTED**      | Always execute non-transactional                     |
| **MANDATORY**          | Must execute inside transaction                      |
| **NEVER**              | Must execute without transaction                     |

### Isolation Levels

```java
@Service
public class IsolationService {

    // Default isolation level
    @Transactional
    public void defaultIsolation() { }

    // Dirty Read Protection
    @Transactional(isolation = Isolation.READ_COMMITTED)
    public void readCommitted() { }

    // Phantom Read Protection
    @Transactional(isolation = Isolation.REPEATABLE_READ)
    public void repeatableRead() { }

    // Strictest - Serializable
    @Transactional(isolation = Isolation.SERIALIZABLE)
    public void serializable() { }

    // Allows dirty reads (fastest)
    @Transactional(isolation = Isolation.READ_UNCOMMITTED)
    public void readUncommitted() { }
}
```

| Level                | Dirty Read | Non-Repeatable Read | Phantom Read |
| -------------------- | ---------- | ------------------- | ------------ |
| **READ_UNCOMMITTED** | ✅ Yes     | ✅ Yes              | ✅ Yes       |
| **READ_COMMITTED**   | ❌ No      | ✅ Yes              | ✅ Yes       |
| **REPEATABLE_READ**  | ❌ No      | ❌ No               | ✅ Yes       |
| **SERIALIZABLE**     | ❌ No      | ❌ No               | ❌ No        |

### Programmatic Transactions

```java
@Service
public class ProgrammaticTransactionService {

    private TransactionTemplate transactionTemplate;
    private AccountRepository accountRepository;

    public ProgrammaticTransactionService(TransactionTemplate transactionTemplate,
                                         AccountRepository accountRepository) {
        this.transactionTemplate = transactionTemplate;
        this.accountRepository = accountRepository;
    }

    public void transferMoney(Long fromId, Long toId, Double amount) {
        transactionTemplate.execute(status -> {
            Account from = accountRepository.findById(fromId).orElse(null);
            Account to = accountRepository.findById(toId).orElse(null);

            from.setBalance(from.getBalance() - amount);
            to.setBalance(to.getBalance() + amount);

            accountRepository.save(from);
            accountRepository.save(to);

            return null;
        });
    }
}
```

---

## 4.3 N+1 Problem & Fetch Strategies

### The N+1 Problem

```java
// Problem: This causes N+1 queries
@Entity
public class Author {
    @Id
    private Long id;
    private String name;

    @OneToMany(mappedBy = "author", fetch = FetchType.LAZY)
    private List<Book> books;
}

@Service
public class AuthorService {

    private AuthorRepository authorRepository;

    public List<AuthorDTO> getAllAuthorsWithBooks() {
        List<Author> authors = authorRepository.findAll(); // 1 query

        authors.forEach(author ->
            author.getBooks().size() // N queries - one per author!
        );

        return authors.stream()
            .map(this::convertToDTO)
            .collect(Collectors.toList());
    }
}
```

### Solution 1: Eager Loading

```java
@Entity
public class Author {
    @Id
    private Long id;
    private String name;

    @OneToMany(mappedBy = "author", fetch = FetchType.EAGER)
    private List<Book> books;
}

// Now books are fetched with author
```

### Solution 2: JPQL Query with JOIN FETCH

```java
public interface AuthorRepository extends JpaRepository<Author, Long> {
    @Query("SELECT DISTINCT a FROM Author a LEFT JOIN FETCH a.books")
    List<Author> findAllWithBooks();
}

// This retrieves everything in one query
```

### Solution 3: EntityGraph

```java
@Entity
public class Author {
    @Id
    private Long id;
    private String name;

    @OneToMany(mappedBy = "author")
    private List<Book> books;
}

public interface AuthorRepository extends JpaRepository<Author, Long> {
    @EntityGraph(attributePaths = {"books"})
    List<Author> findAll();

    @EntityGraph(attributePaths = {"books"})
    Optional<Author> findById(Long id);
}
```

### Solution 4: Projections (DTO)

```java
public interface AuthorDTO {
    Long getId();
    String getName();
    List<BookDTO> getBooks();
}

public interface BookDTO {
    Long getId();
    String getTitle();
}

public interface AuthorRepository extends JpaRepository<Author, Long> {
    List<AuthorDTO> findAllProjectedBy();
}
```

---

## 📚 Summary

In **Phase 4**, you learned:

- ✅ Advanced query techniques (Named, Native, Specifications)
- ✅ Pagination and sorting
- ✅ Transaction management and propagation
- ✅ Isolation levels and concurrency
- ✅ N+1 problem and solutions
- ✅ Fetch strategies and optimization

---

## 🎯 Next Steps

1. **Practice**: Implement complex queries with specifications
2. **Optimize**: Identify and fix N+1 problems
3. **Understand**: Learn transaction behavior deeply
4. **Ready for Phase 5**: Security - Authentication & Authorization

---

**Ready to secure your application? Let's move to Phase 5! 🚀**
