# Phase 2: Web Development Basics (Week 3-4)

## 📌 Overview

In this phase, you'll master building REST APIs with Spring Boot. We'll cover REST principles, HTTP methods, request/response handling, and validation.

---

## 2.1 REST API Fundamentals

### What is REST?

**REST** stands for **Representational State Transfer**. It's an architectural style for designing networked applications using HTTP requests.

### Core Principles of REST

```
1. Client-Server Architecture
   ├─ Client and Server are independent
   └─ Can evolve separately

2. Statelessness
   ├─ Each request contains all needed information
   ├─ Server doesn't store client context
   └─ Improves scalability

3. Uniform Interface
   ├─ Consistent API design
   ├─ Standard HTTP methods
   └─ Resource-based URLs

4. Cacheability
   ├─ Responses should be cacheable when appropriate
   └─ Improves performance

5. Layered System
   ├─ Architecture can be composed of layers
   └─ Client can't tell if connected directly to end server
```

### HTTP Methods (Verbs)

| Method      | Purpose                 | Idempotent | Safe   |
| ----------- | ----------------------- | ---------- | ------ |
| **GET**     | Retrieve resource       | ✅ Yes     | ✅ Yes |
| **POST**    | Create new resource     | ❌ No      | ❌ No  |
| **PUT**     | Replace entire resource | ✅ Yes     | ❌ No  |
| **PATCH**   | Partial update          | ❌ No      | ❌ No  |
| **DELETE**  | Delete resource         | ✅ Yes     | ❌ No  |
| **HEAD**    | Like GET but no body    | ✅ Yes     | ✅ Yes |
| **OPTIONS** | Describe communication  | ✅ Yes     | ✅ Yes |

### HTTP Status Codes

```
2xx - Success
  ├─ 200 OK - Request succeeded
  ├─ 201 Created - Resource created successfully
  ├─ 204 No Content - Success but no content to return

3xx - Redirection
  ├─ 301 Moved Permanently
  └─ 302 Found

4xx - Client Error
  ├─ 400 Bad Request - Invalid input
  ├─ 401 Unauthorized - Authentication required
  ├─ 403 Forbidden - Access denied
  ├─ 404 Not Found - Resource not found
  └─ 409 Conflict

5xx - Server Error
  ├─ 500 Internal Server Error
  ├─ 502 Bad Gateway
  └─ 503 Service Unavailable
```

---

## 2.2 Building REST APIs with Spring Boot

### Basic Structure

```java
@RestController
@RequestMapping("/api/resources")
public class ResourceController {

    @GetMapping
    public ResponseEntity<List<Resource>> getAll() { }

    @GetMapping("/{id}")
    public ResponseEntity<Resource> getById(@PathVariable Long id) { }

    @PostMapping
    public ResponseEntity<Resource> create(@RequestBody Resource resource) { }

    @PutMapping("/{id}")
    public ResponseEntity<Resource> update(@PathVariable Long id, @RequestBody Resource resource) { }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> delete(@PathVariable Long id) { }
}
```

### Example: Book REST API

```java
// Book.java - Model
package com.example.demo.model;

public class Book {
    private Long id;
    private String title;
    private String author;
    private Double price;

    // Constructor, getters, setters
    public Book(Long id, String title, String author, Double price) {
        this.id = id;
        this.title = title;
        this.author = author;
        this.price = price;
    }

    // Getters and Setters (omitted for brevity)
    public Long getId() { return id; }
    public String getTitle() { return title; }
    public String getAuthor() { return author; }
    public Double getPrice() { return price; }

    public void setId(Long id) { this.id = id; }
    public void setTitle(String title) { this.title = title; }
    public void setAuthor(String author) { this.author = author; }
    public void setPrice(Double price) { this.price = price; }
}
```

```java
// BookRepository.java
package com.example.demo.repository;

import com.example.demo.model.Book;
import org.springframework.stereotype.Repository;
import java.util.*;

@Repository
public class BookRepository {
    private Map<Long, Book> books = new HashMap<>();
    private Long idCounter = 1L;

    public BookRepository() {
        // Initialize with sample data
        save(new Book(null, "Spring Boot in Action", "Craig Walls", 29.99));
        save(new Book(null, "Microservices in Action", "Morgan Bruce", 45.99));
    }

    public Book save(Book book) {
        if (book.getId() == null) {
            book.setId(idCounter++);
        }
        books.put(book.getId(), book);
        return book;
    }

    public Optional<Book> findById(Long id) {
        return Optional.ofNullable(books.get(id));
    }

    public List<Book> findAll() {
        return new ArrayList<>(books.values());
    }

    public void deleteById(Long id) {
        books.remove(id);
    }
}
```

```java
// BookService.java
package com.example.demo.service;

import com.example.demo.model.Book;
import com.example.demo.repository.BookRepository;
import org.springframework.stereotype.Service;
import java.util.List;
import java.util.Optional;

@Service
public class BookService {
    private BookRepository bookRepository;

    public BookService(BookRepository bookRepository) {
        this.bookRepository = bookRepository;
    }

    public Book createBook(Book book) {
        return bookRepository.save(book);
    }

    public Optional<Book> getBook(Long id) {
        return bookRepository.findById(id);
    }

    public List<Book> getAllBooks() {
        return bookRepository.findAll();
    }

    public Book updateBook(Long id, Book bookDetails) {
        Book book = bookRepository.findById(id)
            .orElseThrow(() -> new RuntimeException("Book not found"));

        book.setTitle(bookDetails.getTitle());
        book.setAuthor(bookDetails.getAuthor());
        book.setPrice(bookDetails.getPrice());

        return bookRepository.save(book);
    }

    public void deleteBook(Long id) {
        bookRepository.deleteById(id);
    }
}
```

```java
// BookController.java
package com.example.demo.controller;

import com.example.demo.model.Book;
import com.example.demo.service.BookService;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import java.util.List;
import java.util.Optional;

@RestController
@RequestMapping("/api/books")
public class BookController {
    private BookService bookService;

    public BookController(BookService bookService) {
        this.bookService = bookService;
    }

    // GET all books
    @GetMapping
    public ResponseEntity<List<Book>> getAllBooks() {
        List<Book> books = bookService.getAllBooks();
        return ResponseEntity.ok(books);
    }

    // GET book by ID
    @GetMapping("/{id}")
    public ResponseEntity<Book> getBookById(@PathVariable Long id) {
        Optional<Book> book = bookService.getBook(id);
        return book.map(ResponseEntity::ok)
                   .orElseGet(() -> ResponseEntity.notFound().build());
    }

    // POST - Create new book
    @PostMapping
    public ResponseEntity<Book> createBook(@RequestBody Book book) {
        Book savedBook = bookService.createBook(book);
        return ResponseEntity.status(HttpStatus.CREATED).body(savedBook);
    }

    // PUT - Update book
    @PutMapping("/{id}")
    public ResponseEntity<Book> updateBook(@PathVariable Long id, @RequestBody Book bookDetails) {
        try {
            Book updatedBook = bookService.updateBook(id, bookDetails);
            return ResponseEntity.ok(updatedBook);
        } catch (RuntimeException e) {
            return ResponseEntity.notFound().build();
        }
    }

    // DELETE - Delete book
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteBook(@PathVariable Long id) {
        bookService.deleteBook(id);
        return ResponseEntity.noContent().build();
    }
}
```

---

## 2.3 Request & Response Handling

### @RequestMapping and Shorthand Annotations

```java
// Using @RequestMapping (Generic)
@RequestMapping(value = "/api/books", method = RequestMethod.GET)
public List<Book> getBooks() { }

// Using Shorthand Annotations (Recommended)
@GetMapping("/api/books")
public List<Book> getBooks() { }

@PostMapping("/api/books")
public Book createBook(@RequestBody Book book) { }

@PutMapping("/api/books/{id}")
public Book updateBook(@PathVariable Long id, @RequestBody Book book) { }

@DeleteMapping("/api/books/{id}")
public void deleteBook(@PathVariable Long id) { }
```

### Handling Request Parameters

#### 1. **@PathVariable** - Extract from URL path

```java
// URL: /api/books/5
@GetMapping("/api/books/{id}")
public Book getBook(@PathVariable Long id) {
    return bookService.getBook(id);
}

// Multiple path variables
// URL: /api/authors/3/books/5
@GetMapping("/api/authors/{authorId}/books/{bookId}")
public Book getBookByAuthor(@PathVariable Long authorId, @PathVariable Long bookId) {
    return bookService.getBookByAuthor(authorId, bookId);
}
```

#### 2. **@RequestParam** - Extract from query string

```java
// URL: /api/books?title=Spring&author=Craig
@GetMapping("/api/books")
public List<Book> searchBooks(
    @RequestParam(required = false) String title,
    @RequestParam(required = false) String author,
    @RequestParam(defaultValue = "10") Integer pageSize
) {
    return bookService.searchBooks(title, author, pageSize);
}

// URL: /api/books?sort=price&order=asc
@GetMapping("/api/books")
public List<Book> getBooksWithSorting(
    @RequestParam String sort,
    @RequestParam String order
) {
    return bookService.getBooksSorted(sort, order);
}
```

#### 3. **@RequestBody** - Extract from request body

```java
@PostMapping("/api/books")
public Book createBook(@RequestBody Book book) {
    return bookService.createBook(book);
}

// Request body:
// {
//   "title": "Spring Boot Masterclass",
//   "author": "Expert Dev",
//   "price": 49.99
// }
```

#### 4. **@RequestHeader** - Extract from headers

```java
@GetMapping("/api/books")
public List<Book> getBooks(
    @RequestHeader("Authorization") String token,
    @RequestHeader(value = "X-Custom-Header", required = false) String customHeader
) {
    return bookService.getAllBooks();
}
```

### ResponseEntity for Flexible Responses

```java
@GetMapping("/{id}")
public ResponseEntity<Book> getBook(@PathVariable Long id) {
    return bookService.getBook(id)
        .map(book -> ResponseEntity.ok(book))
        .orElse(ResponseEntity.notFound().build());
}

@PostMapping
public ResponseEntity<Book> createBook(@RequestBody Book book) {
    Book created = bookService.createBook(book);
    return ResponseEntity
        .status(HttpStatus.CREATED)
        .header("Location", "/api/books/" + created.getId())
        .body(created);
}

@DeleteMapping("/{id}")
public ResponseEntity<Void> deleteBook(@PathVariable Long id) {
    bookService.deleteBook(id);
    return ResponseEntity.noContent().build();
}
```

---

## 2.4 Input Validation

### Using Validation Annotations

Add dependency in `pom.xml`:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

```java
// Book.java with validation
package com.example.demo.model;

import jakarta.validation.constraints.*;

public class Book {

    @NotNull(message = "Book ID cannot be null")
    private Long id;

    @NotBlank(message = "Title is required")
    @Size(min = 3, max = 200, message = "Title must be between 3 and 200 characters")
    private String title;

    @NotBlank(message = "Author is required")
    @Size(min = 3, max = 100, message = "Author name must be between 3 and 100 characters")
    private String author;

    @NotNull(message = "Price is required")
    @DecimalMin(value = "0.0", inclusive = false, message = "Price must be greater than 0")
    @DecimalMax(value = "10000.0", message = "Price must not exceed 10000")
    private Double price;

    @Email(message = "Email should be valid")
    private String publisherEmail;

    @Pattern(regexp = "^[0-9]{13}$", message = "ISBN must be 13 digits")
    private String isbn;

    // Constructor, getters, setters...
}
```

### Using @Valid in Controller

```java
@PostMapping
public ResponseEntity<Book> createBook(@Valid @RequestBody Book book) {
    Book savedBook = bookService.createBook(book);
    return ResponseEntity.status(HttpStatus.CREATED).body(savedBook);
}

// PUT with validation
@PutMapping("/{id}")
public ResponseEntity<Book> updateBook(
    @PathVariable Long id,
    @Valid @RequestBody Book bookDetails
) {
    Book updatedBook = bookService.updateBook(id, bookDetails);
    return ResponseEntity.ok(updatedBook);
}
```

### Global Exception Handler for Validation Errors

```java
// GlobalExceptionHandler.java
package com.example.demo.exception;

import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;
import java.util.HashMap;
import java.util.Map;

@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<Map<String, String>> handleValidationExceptions(
        MethodArgumentNotValidException ex
    ) {
        Map<String, String> errors = new HashMap<>();

        ex.getBindingResult().getFieldErrors().forEach(error ->
            errors.put(error.getField(), error.getDefaultMessage())
        );

        return ResponseEntity
            .status(HttpStatus.BAD_REQUEST)
            .body(errors);
    }

    @ExceptionHandler(RuntimeException.class)
    public ResponseEntity<Map<String, String>> handleRuntimeException(RuntimeException ex) {
        Map<String, String> error = new HashMap<>();
        error.put("error", ex.getMessage());

        return ResponseEntity
            .status(HttpStatus.INTERNAL_SERVER_ERROR)
            .body(error);
    }
}
```

### Validation Error Response Example

```json
{
  "title": "Title must be between 3 and 200 characters",
  "author": "Author is required",
  "price": "Price must be greater than 0"
}
```

---

## 2.5 Exception Handling

### Custom Exception Class

```java
// ResourceNotFoundException.java
package com.example.demo.exception;

public class ResourceNotFoundException extends RuntimeException {
    public ResourceNotFoundException(String message) {
        super(message);
    }

    public ResourceNotFoundException(String message, Throwable cause) {
        super(message, cause);
    }
}
```

### Service with Custom Exception

```java
public Book getBook(Long id) {
    return bookRepository.findById(id)
        .orElseThrow(() -> new ResourceNotFoundException("Book not found with ID: " + id));
}
```

### Global Exception Handler

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleResourceNotFound(
        ResourceNotFoundException ex
    ) {
        ErrorResponse errorResponse = new ErrorResponse(
            HttpStatus.NOT_FOUND.value(),
            ex.getMessage(),
            System.currentTimeMillis()
        );

        return ResponseEntity
            .status(HttpStatus.NOT_FOUND)
            .body(errorResponse);
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGlobalException(Exception ex) {
        ErrorResponse errorResponse = new ErrorResponse(
            HttpStatus.INTERNAL_SERVER_ERROR.value(),
            "An error occurred: " + ex.getMessage(),
            System.currentTimeMillis()
        );

        return ResponseEntity
            .status(HttpStatus.INTERNAL_SERVER_ERROR)
            .body(errorResponse);
    }
}

// ErrorResponse.java
public class ErrorResponse {
    private int status;
    private String message;
    private long timestamp;

    public ErrorResponse(int status, String message, long timestamp) {
        this.status = status;
        this.message = message;
        this.timestamp = timestamp;
    }

    // Getters and setters...
}
```

---

## 2.6 Testing Your REST API

### Using cURL

```bash
# GET all books
curl http://localhost:8080/api/books

# GET book by ID
curl http://localhost:8080/api/books/1

# POST - Create new book
curl -X POST http://localhost:8080/api/books \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Spring Boot Masterclass",
    "author": "Expert Dev",
    "price": 49.99,
    "publisherEmail": "pub@example.com",
    "isbn": "1234567890123"
  }'

# PUT - Update book
curl -X PUT http://localhost:8080/api/books/1 \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Updated Title",
    "author": "Updated Author",
    "price": 39.99,
    "publisherEmail": "pub@example.com",
    "isbn": "1234567890123"
  }'

# DELETE book
curl -X DELETE http://localhost:8080/api/books/1

# GET with query parameters
curl "http://localhost:8080/api/books?title=Spring&pageSize=5"
```

### Using Postman

1. Create a new Collection
2. Create requests:
   - **GET** → http://localhost:8080/api/books
   - **POST** → http://localhost:8080/api/books (with JSON body)
   - **PUT** → http://localhost:8080/api/books/1 (with JSON body)
   - **DELETE** → http://localhost:8080/api/books/1

### Unit Testing with MockMvc

```java
// BookControllerTest.java
package com.example.demo.controller;

import com.example.demo.model.Book;
import com.example.demo.service.BookService;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.boot.test.mock.MockBean;
import org.springframework.http.MediaType;
import org.springframework.test.web.servlet.MockMvc;
import java.util.Optional;

import static org.mockito.Mockito.*;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

@WebMvcTest(BookController.class)
public class BookControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private BookService bookService;

    @Test
    public void testGetBookById() throws Exception {
        Book book = new Book(1L, "Spring Boot", "Craig", 29.99);
        when(bookService.getBook(1L)).thenReturn(Optional.of(book));

        mockMvc.perform(get("/api/books/1"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.title").value("Spring Boot"));
    }

    @Test
    public void testCreateBook() throws Exception {
        Book book = new Book(1L, "Spring Boot", "Craig", 29.99);
        when(bookService.createBook(any())).thenReturn(book);

        mockMvc.perform(post("/api/books")
            .contentType(MediaType.APPLICATION_JSON)
            .content("{\"title\":\"Spring Boot\",\"author\":\"Craig\",\"price\":29.99}"))
            .andExpect(status().isCreated());
    }
}
```

---

## 📚 Summary

In **Phase 2**, you learned:

- ✅ REST principles and HTTP methods
- ✅ Building REST APIs with @RestController
- ✅ Handling requests (@PathVariable, @RequestParam, @RequestBody)
- ✅ Input validation with annotations
- ✅ Exception handling with @ExceptionHandler
- ✅ Testing REST APIs

---

## 🎯 Next Steps

1. **Practice**: Build a Product REST API
2. **Experiment**: Try different validation scenarios
3. **Test**: Write unit tests for your controllers
4. **Ready for Phase 3**: Database Integration with JPA/Hibernate

---

**Ready to persist data? Let's move to Phase 3! 🚀**
