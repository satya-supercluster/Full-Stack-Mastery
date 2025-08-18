# Layered Architecture in Spring Boot - Complete Guide

## 📋 Table of Contents

1. [What is Layered Architecture?](#what-is-layered-architecture)
2. [Architecture Overview](#architecture-overview)
3. [Layer Breakdown](#layer-breakdown)
4. [Project Structure](#project-structure)
5. [Complete Code Example](#complete-code-example)
6. [Best Practices](#best-practices)
7. [Benefits & Drawbacks](#benefits--drawbacks)
8. [Common Patterns](#common-patterns)
9. [Testing Strategy](#testing-strategy)

## What is Layered Architecture?

Layered Architecture is a software design pattern that organizes code into horizontal layers, where each layer has a specific responsibility and only communicates with adjacent layers. In Spring Boot applications, this pattern promotes **separation of concerns**, **modularity**, and **maintainability**.

## Architecture Overview

```
┌─────────────────────────────────┐
│        Client/Browser           │
└─────────────┬───────────────────┘
              │ HTTP Requests
              ▼
┌─────────────────────────────────┐
│      Controller Layer           │  ← Presentation Layer
│   (REST API Endpoints)          │
└─────────────┬───────────────────┘
              │ Method Calls
              ▼
┌─────────────────────────────────┐
│        Service Layer            │  ← Business Logic Layer
│     (Business Logic)            │
└─────────────┬───────────────────┘
              │ Method Calls
              ▼
┌─────────────────────────────────┐
│      Repository Layer           │  ← Data Access Layer
│      (Data Access)              │
└─────────────┬───────────────────┘
              │ SQL Queries
              ▼
┌─────────────────────────────────┐
│         Database                │
└─────────────────────────────────┘
```

## Layer Breakdown

### 1. Controller Layer (Presentation Layer)

**Responsibility**: Handle HTTP requests and responses

**Key Features**:

- Maps HTTP endpoints to service methods
- Validates request parameters
- Handles HTTP status codes
- Manages request/response serialization

### 2. Service Layer (Business Logic Layer)

**Responsibility**: Implement business logic and rules

**Key Features**:

- Contains core business logic
- Orchestrates calls to multiple repositories
- Handles transactions
- Validates business rules

### 3. Repository Layer (Data Access Layer)

**Responsibility**: Manage data persistence and retrieval

**Key Features**:

- Abstracts database operations
- Provides CRUD operations
- Handles database-specific logic
- Uses Spring Data JPA for implementation

## Project Structure

```
src/
└── main/
    └── java/
        └── com/
            └── example/
                └── demo/
                    ├── DemoApplication.java
                    ├── controller/
                    │   └── UserController.java
                    ├── service/
                    │   ├── UserService.java
                    │   └── impl/
                    │       └── UserServiceImpl.java
                    ├── repository/
                    │   └── UserRepository.java
                    ├── model/
                    │   └── User.java
                    └── dto/
                        ├── UserRequestDTO.java
                        └── UserResponseDTO.java
```

## Complete Code Example

### 1. Entity/Model Layer

```java
// User.java
package com.example.demo.model;

import javax.persistence.*;
import javax.validation.constraints.Email;
import javax.validation.constraints.NotBlank;
import java.time.LocalDateTime;

@Entity
@Table(name = "users")
public class User {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @NotBlank(message = "Name is required")
    @Column(nullable = false)
    private String name;
    
    @Email(message = "Email should be valid")
    @Column(unique = true, nullable = false)
    private String email;
    
    @Column(name = "created_at")
    private LocalDateTime createdAt;
    
    @Column(name = "updated_at")
    private LocalDateTime updatedAt;
    
    // Constructors
    public User() {}
    
    public User(String name, String email) {
        this.name = name;
        this.email = email;
        this.createdAt = LocalDateTime.now();
        this.updatedAt = LocalDateTime.now();
    }
    
    // Getters and Setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    
    public String getName() { return name; }
    public void setName(String name) { 
        this.name = name;
        this.updatedAt = LocalDateTime.now();
    }
    
    public String getEmail() { return email; }
    public void setEmail(String email) { 
        this.email = email;
        this.updatedAt = LocalDateTime.now();
    }
    
    public LocalDateTime getCreatedAt() { return createdAt; }
    public void setCreatedAt(LocalDateTime createdAt) { this.createdAt = createdAt; }
    
    public LocalDateTime getUpdatedAt() { return updatedAt; }
    public void setUpdatedAt(LocalDateTime updatedAt) { this.updatedAt = updatedAt; }
}
```

### 2. DTO (Data Transfer Objects)

```java
// UserRequestDTO.java
package com.example.demo.dto;

import javax.validation.constraints.Email;
import javax.validation.constraints.NotBlank;

public class UserRequestDTO {
    
    @NotBlank(message = "Name is required")
    private String name;
    
    @Email(message = "Email should be valid")
    @NotBlank(message = "Email is required")
    private String email;
    
    // Constructors
    public UserRequestDTO() {}
    
    public UserRequestDTO(String name, String email) {
        this.name = name;
        this.email = email;
    }
    
    // Getters and Setters
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    
    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }
}

// UserResponseDTO.java
package com.example.demo.dto;

import java.time.LocalDateTime;

public class UserResponseDTO {
    
    private Long id;
    private String name;
    private String email;
    private LocalDateTime createdAt;
    private LocalDateTime updatedAt;
    
    // Constructors
    public UserResponseDTO() {}
    
    public UserResponseDTO(Long id, String name, String email, 
                          LocalDateTime createdAt, LocalDateTime updatedAt) {
        this.id = id;
        this.name = name;
        this.email = email;
        this.createdAt = createdAt;
        this.updatedAt = updatedAt;
    }
    
    // Getters and Setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    
    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }
    
    public LocalDateTime getCreatedAt() { return createdAt; }
    public void setCreatedAt(LocalDateTime createdAt) { this.createdAt = createdAt; }
    
    public LocalDateTime getUpdatedAt() { return updatedAt; }
    public void setUpdatedAt(LocalDateTime updatedAt) { this.updatedAt = updatedAt; }
}
```

### 3. Repository Layer

```java
// UserRepository.java
package com.example.demo.repository;

import com.example.demo.model.User;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;
import org.springframework.stereotype.Repository;

import java.util.List;
import java.util.Optional;

@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    
    // Custom query methods
    Optional<User> findByEmail(String email);
    
    List<User> findByNameContainingIgnoreCase(String name);
    
    @Query("SELECT u FROM User u WHERE u.email = :email")
    Optional<User> findUserByEmail(@Param("email") String email);
    
    @Query("SELECT COUNT(u) FROM User u WHERE u.email = :email")
    Long countByEmail(@Param("email") String email);
    
    boolean existsByEmail(String email);
}
```

### 4. Service Layer

```java
// UserService.java (Interface)
package com.example.demo.service;

import com.example.demo.dto.UserRequestDTO;
import com.example.demo.dto.UserResponseDTO;

import java.util.List;

public interface UserService {
    
    UserResponseDTO createUser(UserRequestDTO userRequestDTO);
    
    UserResponseDTO getUserById(Long id);
    
    UserResponseDTO getUserByEmail(String email);
    
    List<UserResponseDTO> getAllUsers();
    
    List<UserResponseDTO> searchUsersByName(String name);
    
    UserResponseDTO updateUser(Long id, UserRequestDTO userRequestDTO);
    
    void deleteUser(Long id);
    
    boolean existsByEmail(String email);
}

// UserServiceImpl.java (Implementation)
package com.example.demo.service.impl;

import com.example.demo.dto.UserRequestDTO;
import com.example.demo.dto.UserResponseDTO;
import com.example.demo.model.User;
import com.example.demo.repository.UserRepository;
import com.example.demo.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import javax.persistence.EntityNotFoundException;
import java.util.List;
import java.util.stream.Collectors;

@Service
@Transactional
public class UserServiceImpl implements UserService {
    
    private final UserRepository userRepository;
    
    @Autowired
    public UserServiceImpl(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
    
    @Override
    public UserResponseDTO createUser(UserRequestDTO userRequestDTO) {
        // Business validation
        if (existsByEmail(userRequestDTO.getEmail())) {
            throw new IllegalArgumentException("User with email " + userRequestDTO.getEmail() + " already exists");
        }
        
        // Convert DTO to Entity
        User user = new User(userRequestDTO.getName(), userRequestDTO.getEmail());
        
        // Save user
        User savedUser = userRepository.save(user);
        
        // Convert Entity to Response DTO
        return convertToResponseDTO(savedUser);
    }
    
    @Override
    @Transactional(readOnly = true)
    public UserResponseDTO getUserById(Long id) {
        User user = userRepository.findById(id)
                .orElseThrow(() -> new EntityNotFoundException("User not found with id: " + id));
        return convertToResponseDTO(user);
    }
    
    @Override
    @Transactional(readOnly = true)
    public UserResponseDTO getUserByEmail(String email) {
        User user = userRepository.findByEmail(email)
                .orElseThrow(() -> new EntityNotFoundException("User not found with email: " + email));
        return convertToResponseDTO(user);
    }
    
    @Override
    @Transactional(readOnly = true)
    public List<UserResponseDTO> getAllUsers() {
        List<User> users = userRepository.findAll();
        return users.stream()
                .map(this::convertToResponseDTO)
                .collect(Collectors.toList());
    }
    
    @Override
    @Transactional(readOnly = true)
    public List<UserResponseDTO> searchUsersByName(String name) {
        List<User> users = userRepository.findByNameContainingIgnoreCase(name);
        return users.stream()
                .map(this::convertToResponseDTO)
                .collect(Collectors.toList());
    }
    
    @Override
    public UserResponseDTO updateUser(Long id, UserRequestDTO userRequestDTO) {
        User existingUser = userRepository.findById(id)
                .orElseThrow(() -> new EntityNotFoundException("User not found with id: " + id));
        
        // Business validation - check if email is being changed to an existing one
        if (!existingUser.getEmail().equals(userRequestDTO.getEmail()) && 
            existsByEmail(userRequestDTO.getEmail())) {
            throw new IllegalArgumentException("Email " + userRequestDTO.getEmail() + " is already in use");
        }
        
        // Update entity
        existingUser.setName(userRequestDTO.getName());
        existingUser.setEmail(userRequestDTO.getEmail());
        
        User updatedUser = userRepository.save(existingUser);
        return convertToResponseDTO(updatedUser);
    }
    
    @Override
    public void deleteUser(Long id) {
        if (!userRepository.existsById(id)) {
            throw new EntityNotFoundException("User not found with id: " + id);
        }
        userRepository.deleteById(id);
    }
    
    @Override
    @Transactional(readOnly = true)
    public boolean existsByEmail(String email) {
        return userRepository.existsByEmail(email);
    }
    
    // Helper method to convert Entity to Response DTO
    private UserResponseDTO convertToResponseDTO(User user) {
        return new UserResponseDTO(
                user.getId(),
                user.getName(),
                user.getEmail(),
                user.getCreatedAt(),
                user.getUpdatedAt()
        );
    }
}
```

### 5. Controller Layer

```java
// UserController.java
package com.example.demo.controller;

import com.example.demo.dto.UserRequestDTO;
import com.example.demo.dto.UserResponseDTO;
import com.example.demo.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import javax.validation.Valid;
import java.util.List;

@RestController
@RequestMapping("/api/users")
@CrossOrigin(origins = "*")
public class UserController {
    
    private final UserService userService;
    
    @Autowired
    public UserController(UserService userService) {
        this.userService = userService;
    }
    
    // Create a new user
    @PostMapping
    public ResponseEntity<UserResponseDTO> createUser(@Valid @RequestBody UserRequestDTO userRequestDTO) {
        try {
            UserResponseDTO createdUser = userService.createUser(userRequestDTO);
            return new ResponseEntity<>(createdUser, HttpStatus.CREATED);
        } catch (IllegalArgumentException e) {
            return new ResponseEntity<>(null, HttpStatus.BAD_REQUEST);
        }
    }
    
    // Get user by ID
    @GetMapping("/{id}")
    public ResponseEntity<UserResponseDTO> getUserById(@PathVariable Long id) {
        try {
            UserResponseDTO user = userService.getUserById(id);
            return new ResponseEntity<>(user, HttpStatus.OK);
        } catch (EntityNotFoundException e) {
            return new ResponseEntity<>(null, HttpStatus.NOT_FOUND);
        }
    }
    
    // Get user by email
    @GetMapping("/email/{email}")
    public ResponseEntity<UserResponseDTO> getUserByEmail(@PathVariable String email) {
        try {
            UserResponseDTO user = userService.getUserByEmail(email);
            return new ResponseEntity<>(user, HttpStatus.OK);
        } catch (EntityNotFoundException e) {
            return new ResponseEntity<>(null, HttpStatus.NOT_FOUND);
        }
    }
    
    // Get all users
    @GetMapping
    public ResponseEntity<List<UserResponseDTO>> getAllUsers() {
        List<UserResponseDTO> users = userService.getAllUsers();
        return new ResponseEntity<>(users, HttpStatus.OK);
    }
    
    // Search users by name
    @GetMapping("/search")
    public ResponseEntity<List<UserResponseDTO>> searchUsersByName(@RequestParam String name) {
        List<UserResponseDTO> users = userService.searchUsersByName(name);
        return new ResponseEntity<>(users, HttpStatus.OK);
    }
    
    // Update user
    @PutMapping("/{id}")
    public ResponseEntity<UserResponseDTO> updateUser(@PathVariable Long id, 
                                                     @Valid @RequestBody UserRequestDTO userRequestDTO) {
        try {
            UserResponseDTO updatedUser = userService.updateUser(id, userRequestDTO);
            return new ResponseEntity<>(updatedUser, HttpStatus.OK);
        } catch (EntityNotFoundException e) {
            return new ResponseEntity<>(null, HttpStatus.NOT_FOUND);
        } catch (IllegalArgumentException e) {
            return new ResponseEntity<>(null, HttpStatus.BAD_REQUEST);
        }
    }
    
    // Delete user
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
        try {
            userService.deleteUser(id);
            return new ResponseEntity<>(HttpStatus.NO_CONTENT);
        } catch (EntityNotFoundException e) {
            return new ResponseEntity<>(HttpStatus.NOT_FOUND);
        }
    }
    
    // Check if user exists by email
    @GetMapping("/exists/{email}")
    public ResponseEntity<Boolean> checkUserExists(@PathVariable String email) {
        boolean exists = userService.existsByEmail(email);
        return new ResponseEntity<>(exists, HttpStatus.OK);
    }
}
```

### 6. Exception Handling

```java
// GlobalExceptionHandler.java
package com.example.demo.exception;

import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.validation.FieldError;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

import javax.persistence.EntityNotFoundException;
import java.util.HashMap;
import java.util.Map;

@RestControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(EntityNotFoundException.class)
    public ResponseEntity<Map<String, String>> handleEntityNotFoundException(EntityNotFoundException ex) {
        Map<String, String> error = new HashMap<>();
        error.put("error", ex.getMessage());
        return new ResponseEntity<>(error, HttpStatus.NOT_FOUND);
    }
    
    @ExceptionHandler(IllegalArgumentException.class)
    public ResponseEntity<Map<String, String>> handleIllegalArgumentException(IllegalArgumentException ex) {
        Map<String, String> error = new HashMap<>();
        error.put("error", ex.getMessage());
        return new ResponseEntity<>(error, HttpStatus.BAD_REQUEST);
    }
    
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<Map<String, String>> handleValidationExceptions(MethodArgumentNotValidException ex) {
        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getAllErrors().forEach((error) -> {
            String fieldName = ((FieldError) error).getField();
            String errorMessage = error.getDefaultMessage();
            errors.put(fieldName, errorMessage);
        });
        return new ResponseEntity<>(errors, HttpStatus.BAD_REQUEST);
    }
}
```

## Best Practices

### 1. **Dependency Injection**

- Use constructor injection over field injection
- Inject interfaces, not implementations
- Keep dependencies minimal in each layer

### 2. **Transaction Management**

- Use `@Transactional` on service methods
- Use `@Transactional(readOnly = true)` for read operations
- Handle transaction boundaries properly

### 3. **Error Handling**

- Implement global exception handling
- Use specific exception types
- Return appropriate HTTP status codes

### 4. **Validation**

- Validate at the controller layer using Bean Validation
- Implement business rule validation in the service layer
- Use DTOs to control data flow

### 5. **Testing**

- Write unit tests for each layer
- Use mocking for external dependencies
- Implement integration tests for the complete flow

## Benefits & Drawbacks

### ✅ Benefits

1. **Separation of Concerns**: Each layer has a specific responsibility
2. **Modularity**: Easy to modify one layer without affecting others
3. **Testability**: Each layer can be tested independently
4. **Maintainability**: Clear structure makes code easier to maintain
5. **Scalability**: Easy to scale individual layers
6. **Reusability**: Service layer can be reused by different controllers

### ❌ Drawbacks

1. **Over-engineering**: Can be overkill for simple applications
2. **Performance**: Additional layers can introduce latency
3. **Complexity**: More files and classes to maintain
4. **Tight Coupling**: Layers can become tightly coupled if not designed properly

## Common Patterns

### 1. **DTO Pattern**

Use Data Transfer Objects to control data flow between layers:

```java
// Mapping utility
public class UserMapper {
    public static UserResponseDTO toResponseDTO(User user) {
        return new UserResponseDTO(
            user.getId(),
            user.getName(),
            user.getEmail(),
            user.getCreatedAt(),
            user.getUpdatedAt()
        );
    }
    
    public static User toEntity(UserRequestDTO dto) {
        return new User(dto.getName(), dto.getEmail());
    }
}
```

### 2. **Repository Pattern**

Abstract data access logic:

```java
@Repository
public interface GenericRepository<T, ID> extends JpaRepository<T, ID> {
    // Common methods for all entities
}

public interface UserRepository extends GenericRepository<User, Long> {
    // User-specific methods
}
```

### 3. **Service Interface Pattern**

Define service contracts:

```java
public interface CrudService<REQUEST_DTO, RESPONSE_DTO, ID> {
    RESPONSE_DTO create(REQUEST_DTO dto);
    RESPONSE_DTO getById(ID id);
    List<RESPONSE_DTO> getAll();
    RESPONSE_DTO update(ID id, REQUEST_DTO dto);
    void delete(ID id);
}
```

## Testing Strategy

### 1. **Unit Tests**

```java
// Service Layer Test
@ExtendWith(MockitoExtension.class)
class UserServiceImplTest {
    
    @Mock
    private UserRepository userRepository;
    
    @InjectMocks
    private UserServiceImpl userService;
    
    @Test
    void createUser_ShouldReturnUserResponseDTO() {
        // Given
        UserRequestDTO requestDTO = new UserRequestDTO("John Doe", "john@example.com");
        User savedUser = new User("John Doe", "john@example.com");
        savedUser.setId(1L);
        
        when(userRepository.existsByEmail("john@example.com")).thenReturn(false);
        when(userRepository.save(any(User.class))).thenReturn(savedUser);
        
        // When
        UserResponseDTO result = userService.createUser(requestDTO);
        
        // Then
        assertThat(result.getName()).isEqualTo("John Doe");
        assertThat(result.getEmail()).isEqualTo("john@example.com");
        verify(userRepository).save(any(User.class));
    }
}

// Controller Layer Test
@WebMvcTest(UserController.class)
class UserControllerTest {
    
    @Autowired
    private MockMvc mockMvc;
    
    @MockBean
    private UserService userService;
    
    @Test
    void createUser_ShouldReturnCreated() throws Exception {
        // Given
        UserRequestDTO requestDTO = new UserRequestDTO("John Doe", "john@example.com");
        UserResponseDTO responseDTO = new UserResponseDTO(1L, "John Doe", "john@example.com", 
                                                         LocalDateTime.now(), LocalDateTime.now());
        
        when(userService.createUser(any(UserRequestDTO.class))).thenReturn(responseDTO);
        
        // When & Then
        mockMvc.perform(post("/api/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content("{\"name\":\"John Doe\",\"email\":\"john@example.com\"}"))
                .andExpect(status().isCreated())
                .andExpect(jsonPath("$.name").value("John Doe"))
                .andExpect(jsonPath("$.email").value("john@example.com"));
    }
}
```

### 2. **Integration Tests**

```java
@SpringBootTest
@TestPropertySource(locations = "classpath:application-test.properties")
@Transactional
class UserIntegrationTest {
    
    @Autowired
    private TestRestTemplate restTemplate;
    
    @Autowired
    private UserRepository userRepository;
    
    @Test
    void createAndRetrieveUser_ShouldWork() {
        // Given
        UserRequestDTO requestDTO = new UserRequestDTO("Jane Doe", "jane@example.com");
        
        // When - Create user
        ResponseEntity<UserResponseDTO> createResponse = restTemplate.postForEntity(
                "/api/users", requestDTO, UserResponseDTO.class);
        
        // Then - Verify creation
        assertThat(createResponse.getStatusCode()).isEqualTo(HttpStatus.CREATED);
        
        Long userId = createResponse.getBody().getId();
        
        // When - Retrieve user
        ResponseEntity<UserResponseDTO> getResponse = restTemplate.getForEntity(
                "/api/users/" + userId, UserResponseDTO.class);
        
        // Then - Verify retrieval
        assertThat(getResponse.getStatusCode()).isEqualTo(HttpStatus.OK);
        assertThat(getResponse.getBody().getName()).isEqualTo("Jane Doe");
    }
}
```

## Running Unit Tests

### 1. **Prerequisites**

Add these dependencies to your `pom.xml`:

```xml
<dependencies>
    <!-- Spring Boot Test Starter -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
    
    <!-- Testcontainers (for integration tests with real database) -->
    <dependency>
        <groupId>org.testcontainers</groupId>
        <artifactId>junit-jupiter</artifactId>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.testcontainers</groupId>
        <artifactId>postgresql</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

### 2. **Test Configuration Files**

Create `src/test/resources/application-test.properties`:

```properties
# Test Database Configuration
spring.datasource.url=jdbc:h2://mem:testdb;DB_CLOSE_DELAY=-1;DB_CLOSE_ON_EXIT=FALSE
spring.datasource.driver-class-name=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=

# JPA Configuration for Tests
spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
spring.jpa.hibernate.ddl-auto=create-drop
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true

# Logging Configuration
logging.level.org.springframework.test=DEBUG
logging.level.org.hibernate.SQL=DEBUG
logging.level.org.hibernate.type.descriptor.sql.BasicBinder=TRACE
```

### 3. **Complete Test Examples**

#### Repository Layer Tests

```java
// UserRepositoryTest.java
package com.example.demo.repository;

import com.example.demo.model.User;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;
import org.springframework.boot.test.autoconfigure.orm.jpa.TestEntityManager;
import org.springframework.test.context.TestPropertySource;

import java.util.List;
import java.util.Optional;

import static org.assertj.core.api.Assertions.assertThat;

@DataJpaTest
@TestPropertySource(locations = "classpath:application-test.properties")
class UserRepositoryTest {
    
    @Autowired
    private TestEntityManager entityManager;
    
    @Autowired
    private UserRepository userRepository;
    
    private User testUser;
    
    @BeforeEach
    void setUp() {
        testUser = new User("John Doe", "john@example.com");
        entityManager.persistAndFlush(testUser);
    }
    
    @Test
    void findByEmail_ShouldReturnUser_WhenEmailExists() {
        // When
        Optional<User> found = userRepository.findByEmail("john@example.com");
        
        // Then
        assertThat(found).isPresent();
        assertThat(found.get().getName()).isEqualTo("John Doe");
        assertThat(found.get().getEmail()).isEqualTo("john@example.com");
    }
    
    @Test
    void findByEmail_ShouldReturnEmpty_WhenEmailDoesNotExist() {
        // When
        Optional<User> found = userRepository.findByEmail("nonexistent@example.com");
        
        // Then
        assertThat(found).isEmpty();
    }
    
    @Test
    void findByNameContainingIgnoreCase_ShouldReturnMatchingUsers() {
        // Given
        User anotherUser = new User("Jane Doe", "jane@example.com");
        entityManager.persistAndFlush(anotherUser);
        
        // When
        List<User> found = userRepository.findByNameContainingIgnoreCase("doe");
        
        // Then
        assertThat(found).hasSize(2);
        assertThat(found).extracting(User::getName).contains("John Doe", "Jane Doe");
    }
    
    @Test
    void existsByEmail_ShouldReturnTrue_WhenEmailExists() {
        // When
        boolean exists = userRepository.existsByEmail("john@example.com");
        
        // Then
        assertThat(exists).isTrue();
    }
    
    @Test
    void existsByEmail_ShouldReturnFalse_WhenEmailDoesNotExist() {
        // When
        boolean exists = userRepository.existsByEmail("nonexistent@example.com");
        
        // Then
        assertThat(exists).isFalse();
    }
}
```

#### Service Layer Tests

```java
// UserServiceImplTest.java
package com.example.demo.service.impl;

import com.example.demo.dto.UserRequestDTO;
import com.example.demo.dto.UserResponseDTO;
import com.example.demo.model.User;
import com.example.demo.repository.UserRepository;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;

import javax.persistence.EntityNotFoundException;
import java.time.LocalDateTime;
import java.util.Arrays;
import java.util.List;
import java.util.Optional;

import static org.assertj.core.api.Assertions.assertThat;
import static org.assertj.core.api.Assertions.assertThatThrownBy;
import static org.mockito.ArgumentMatchers.any;
import static org.mockito.ArgumentMatchers.anyString;
import static org.mockito.Mockito.*;

@ExtendWith(MockitoExtension.class)
class UserServiceImplTest {
    
    @Mock
    private UserRepository userRepository;
    
    @InjectMocks
    private UserServiceImpl userService;
    
    private User testUser;
    private UserRequestDTO userRequestDTO;
    private UserResponseDTO expectedResponseDTO;
    
    @BeforeEach
    void setUp() {
        testUser = new User("John Doe", "john@example.com");
        testUser.setId(1L);
        
        userRequestDTO = new UserRequestDTO("John Doe", "john@example.com");
        
        expectedResponseDTO = new UserResponseDTO(
            1L, "John Doe", "john@example.com", 
            LocalDateTime.now(), LocalDateTime.now()
        );
    }
    
    @Test
    void createUser_ShouldReturnUserResponseDTO_WhenValidRequest() {
        // Given
        when(userRepository.existsByEmail("john@example.com")).thenReturn(false);
        when(userRepository.save(any(User.class))).thenReturn(testUser);
        
        // When
        UserResponseDTO result = userService.createUser(userRequestDTO);
        
        // Then
        assertThat(result.getName()).isEqualTo("John Doe");
        assertThat(result.getEmail()).isEqualTo("john@example.com");
        assertThat(result.getId()).isEqualTo(1L);
        
        verify(userRepository).existsByEmail("john@example.com");
        verify(userRepository).save(any(User.class));
    }
    
    @Test
    void createUser_ShouldThrowException_WhenEmailAlreadyExists() {
        // Given
        when(userRepository.existsByEmail("john@example.com")).thenReturn(true);
        
        // When & Then
        assertThatThrownBy(() -> userService.createUser(userRequestDTO))
            .isInstanceOf(IllegalArgumentException.class)
            .hasMessageContaining("already exists");
        
        verify(userRepository).existsByEmail("john@example.com");
        verify(userRepository, never()).save(any(User.class));
    }
    
    @Test
    void getUserById_ShouldReturnUser_WhenUserExists() {
        // Given
        when(userRepository.findById(1L)).thenReturn(Optional.of(testUser));
        
        // When
        UserResponseDTO result = userService.getUserById(1L);
        
        // Then
        assertThat(result.getId()).isEqualTo(1L);
        assertThat(result.getName()).isEqualTo("John Doe");
        assertThat(result.getEmail()).isEqualTo("john@example.com");
        
        verify(userRepository).findById(1L);
    }
    
    @Test
    void getUserById_ShouldThrowException_WhenUserDoesNotExist() {
        // Given
        when(userRepository.findById(1L)).thenReturn(Optional.empty());
        
        // When & Then
        assertThatThrownBy(() -> userService.getUserById(1L))
            .isInstanceOf(EntityNotFoundException.class)
            .hasMessageContaining("User not found with id: 1");
        
        verify(userRepository).findById(1L);
    }
    
    @Test
    void getAllUsers_ShouldReturnAllUsers() {
        // Given
        User user2 = new User("Jane Doe", "jane@example.com");
        user2.setId(2L);
        
        when(userRepository.findAll()).thenReturn(Arrays.asList(testUser, user2));
        
        // When
        List<UserResponseDTO> result = userService.getAllUsers();
        
        // Then
        assertThat(result).hasSize(2);
        assertThat(result.get(0).getName()).isEqualTo("John Doe");
        assertThat(result.get(1).getName()).isEqualTo("Jane Doe");
        
        verify(userRepository).findAll();
    }
    
    @Test
    void updateUser_ShouldUpdateAndReturnUser_WhenValidRequest() {
        // Given
        UserRequestDTO updateRequest = new UserRequestDTO("John Updated", "john.updated@example.com");
        User updatedUser = new User("John Updated", "john.updated@example.com");
        updatedUser.setId(1L);
        
        when(userRepository.findById(1L)).thenReturn(Optional.of(testUser));
        when(userRepository.existsByEmail("john.updated@example.com")).thenReturn(false);
        when(userRepository.save(any(User.class))).thenReturn(updatedUser);
        
        // When
        UserResponseDTO result = userService.updateUser(1L, updateRequest);
        
        // Then
        assertThat(result.getName()).isEqualTo("John Updated");
        assertThat(result.getEmail()).isEqualTo("john.updated@example.com");
        
        verify(userRepository).findById(1L);
        verify(userRepository).save(any(User.class));
    }
    
    @Test
    void deleteUser_ShouldDeleteUser_WhenUserExists() {
        // Given
        when(userRepository.existsById(1L)).thenReturn(true);
        
        // When
        userService.deleteUser(1L);
        
        // Then
        verify(userRepository).existsById(1L);
        verify(userRepository).deleteById(1L);
    }
    
    @Test
    void deleteUser_ShouldThrowException_WhenUserDoesNotExist() {
        // Given
        when(userRepository.existsById(1L)).thenReturn(false);
        
        // When & Then
        assertThatThrownBy(() -> userService.deleteUser(1L))
            .isInstanceOf(EntityNotFoundException.class)
            .hasMessageContaining("User not found with id: 1");
        
        verify(userRepository).existsById(1L);
        verify(userRepository, never()).deleteById(anyLong());
    }
}
```

#### Controller Layer Tests

```java
// UserControllerTest.java
package com.example.demo.controller;

import com.example.demo.dto.UserRequestDTO;
import com.example.demo.dto.UserResponseDTO;
import com.example.demo.service.UserService;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.http.MediaType;
import org.springframework.test.web.servlet.MockMvc;

import javax.persistence.EntityNotFoundException;
import java.time.LocalDateTime;
import java.util.Arrays;
import java.util.List;

import static org.mockito.ArgumentMatchers.any;
import static org.mockito.ArgumentMatchers.anyLong;
import static org.mockito.Mockito.when;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultHandlers.print;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

@WebMvcTest(UserController.class)
class UserControllerTest {
    
    @Autowired
    private MockMvc mockMvc;
    
    @MockBean
    private UserService userService;
    
    @Autowired
    private ObjectMapper objectMapper;
    
    private UserRequestDTO userRequestDTO;
    private UserResponseDTO userResponseDTO;
    
    @BeforeEach
    void setUp() {
        userRequestDTO = new UserRequestDTO("John Doe", "john@example.com");
        userResponseDTO = new UserResponseDTO(
            1L, "John Doe", "john@example.com", 
            LocalDateTime.now(), LocalDateTime.now()
        );
    }
    
    @Test
    void createUser_ShouldReturnCreatedUser_WhenValidRequest() throws Exception {
        // Given
        when(userService.createUser(any(UserRequestDTO.class))).thenReturn(userResponseDTO);
        
        // When & Then
        mockMvc.perform(post("/api/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(userRequestDTO)))
                .andDo(print())
                .andExpect(status().isCreated())
                .andExpect(jsonPath("$.id").value(1L))
                .andExpect(jsonPath("$.name").value("John Doe"))
                .andExpect(jsonPath("$.email").value("john@example.com"));
    }
    
    @Test
    void createUser_ShouldReturnBadRequest_WhenInvalidRequest() throws Exception {
        // Given
        UserRequestDTO invalidRequest = new UserRequestDTO("", "invalid-email");
        
        // When & Then
        mockMvc.perform(post("/api/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(invalidRequest)))
                .andDo(print())
                .andExpect(status().isBadRequest());
    }
    
    @Test
    void getUserById_ShouldReturnUser_WhenUserExists() throws Exception {
        // Given
        when(userService.getUserById(1L)).thenReturn(userResponseDTO);
        
        // When & Then
        mockMvc.perform(get("/api/users/1"))
                .andDo(print())
                .andExpected(status().isOk())
                .andExpect(jsonPath("$.id").value(1L))
                .andExpect(jsonPath("$.name").value("John Doe"))
                .andExpect(jsonPath("$.email").value("john@example.com"));
    }
    
    @Test
    void getUserById_ShouldReturnNotFound_WhenUserDoesNotExist() throws Exception {
        // Given
        when(userService.getUserById(1L)).thenThrow(new EntityNotFoundException("User not found"));
        
        // When & Then
        mockMvc.perform(get("/api/users/1"))
                .andDo(print())
                .andExpect(status().isNotFound());
    }
    
    @Test
    void getAllUsers_ShouldReturnAllUsers() throws Exception {
        // Given
        UserResponseDTO user2 = new UserResponseDTO(
            2L, "Jane Doe", "jane@example.com",
            LocalDateTime.now(), LocalDateTime.now()
        );
        List<UserResponseDTO> users = Arrays.asList(userResponseDTO, user2);
        
        when(userService.getAllUsers()).thenReturn(users);
        
        // When & Then
        mockMvc.perform(get("/api/users"))
                .andDo(print())
                .andExpect(status().isOk())
                .andExpected(jsonPath("$.length()").value(2))
                .andExpected(jsonPath("$[0].name").value("John Doe"))
                .andExpected(jsonPath("$[1].name").value("Jane Doe"));
    }
    
    @Test
    void updateUser_ShouldReturnUpdatedUser_WhenValidRequest() throws Exception {
        // Given
        UserRequestDTO updateRequest = new UserRequestDTO("John Updated", "john.updated@example.com");
        UserResponseDTO updatedResponse = new UserResponseDTO(
            1L, "John Updated", "john.updated@example.com",
            LocalDateTime.now(), LocalDateTime.now()
        );
        
        when(userService.updateUser(anyLong(), any(UserRequestDTO.class))).thenReturn(updatedResponse);
        
        // When & Then
        mockMvc.perform(put("/api/users/1")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(updateRequest)))
                .andDo(print())
                .andExpect(status().isOk())
                .andExpected(jsonPath("$.name").value("John Updated"))
                .andExpected(jsonPath("$.email").value("john.updated@example.com"));
    }
    
    @Test
    void deleteUser_ShouldReturnNoContent_WhenUserExists() throws Exception {
        // When & Then
        mockMvc.perform(delete("/api/users/1"))
                .andDo(print())
                .andExpect(status().isNoContent());
    }
}
```

### 4. **Running Tests - Different Methods**

#### Method 1: Using Maven Commands

```bash
# Run all tests
mvn test

# Run tests with verbose output
mvn test -Dtest.verbose=true

# Run specific test class
mvn test -Dtest=UserServiceImplTest

# Run specific test method
mvn test -Dtest=UserServiceImplTest#createUser_ShouldReturnUserResponseDTO_WhenValidRequest

# Run tests with coverage report
mvn test jacoco:report

# Run tests and skip compilation
mvn surefire:test

# Run tests with specific profile
mvn test -Ptest

# Run integration tests only
mvn test -Dtest=*IntegrationTest

# Run unit tests only (excluding integration tests)
mvn test -Dtest=!*IntegrationTest
```

#### Method 2: Using Gradle Commands

```bash
# Run all tests
./gradlew test

# Run specific test class
./gradlew test --tests UserServiceImplTest

# Run specific test method
./gradlew test --tests "UserServiceImplTest.createUser_ShouldReturnUserResponseDTO_WhenValidRequest"

# Run tests with continuous build
./gradlew test --continuous

# Run tests with detailed output
./gradlew test --info

# Run tests and generate reports
./gradlew test jacocoTestReport
```

#### Method 3: Using IDE

**IntelliJ IDEA:**

- Right-click on test class/method → "Run Test"
- Use Ctrl+Shift+F10 (Windows/Linux) or Cmd+Shift+R (Mac)
- Use the green arrow buttons in the gutter

**Eclipse:**

- Right-click on test class → "Run As" → "JUnit Test"
- Use Alt+Shift+X, T shortcut

**VS Code:**

- Install Java Test Runner extension
- Use CodeLens "Run Test" links
- Use Ctrl+Shift+P → "Java: Run Tests"

### 5. **Test Reports and Coverage**

#### Configure JaCoCo for Coverage (Maven)

```xml
<plugin>
    <groupId>org.jacoco</groupId>
    <artifactId>jacoco-maven-plugin</artifactId>
    <version>0.8.8</version>
    <executions>
        <execution>
            <goals>
                <goal>prepare-agent</goal>
            </goals>
        </execution>
        <execution>
            <id>report</id>
            <phase>test</phase>
            <goals>
                <goal>report</goal>
            </goals>
        </execution>
        <execution>
            <id>check</id>
            <goals>
                <goal>check</goal>
            </goals>
            <configuration>
                <rules>
                    <rule>
                        <element>PACKAGE</element>
                        <limits>
                            <limit>
                                <counter>LINE</counter>
                                <value>COVEREDRATIO</value>
                                <minimum>0.80</minimum>
                            </limit>
                        </limits>
                    </rule>
                </rules>
            </configuration>
        </execution>
    </executions>
</plugin>
```

#### Run Tests with Coverage

```bash
# Generate coverage report
mvn clean test jacoco:report

# View coverage report
open target/site/jacoco/index.html
```

### 6. **Test Categories and Profiles**

#### Create Test Categories

```java
// Marker interfaces for test categories
public interface UnitTest {}
public interface IntegrationTest {}
public interface SlowTest {}

// Apply categories to tests
@Tag("unit")
class UserServiceImplTest {
    // Unit tests
}

@Tag("integration")
@SpringBootTest
class UserIntegrationTest {
    // Integration tests  
}
```

#### Maven Profiles for Different Test Types

```xml
<profiles>
    <profile>
        <id>unit-tests</id>
        <build>
            <plugins>
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-surefire-plugin</artifactId>
                    <configuration>
                        <groups>unit</groups>
                    </configuration>
                </plugin>
            </plugins>
        </build>
    </profile>
    
    <profile>
        <id>integration-tests</id>
        <build>
            <plugins>
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-surefire-plugin</artifactId>
                    <configuration>
                        <groups>integration</groups>
                    </configuration>
                </plugin>
            </plugins>
        </build>
    </profile>
</profiles>
```

#### Run Specific Test Categories

```bash
# Run only unit tests
mvn test -Punit-tests

# Run only integration tests  
mvn test -Pintegration-tests
```

### 7. **Continuous Integration Setup**

#### GitHub Actions Example

```yaml
# .github/workflows/test.yml
name: Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v2
    
    - name: Set up JDK 11
      uses: actions/setup-java@v2
      with:
        java-version: '11'
        distribution: 'temurin'
    
    - name: Cache Maven packages
      uses: actions/cache@v2
      with:
        path: ~/.m2
        key: ${{ runner.os }}-m2-${{ hashFiles('**/pom.xml') }}
        
    - name: Run Unit Tests
      run: mvn test -Punit-tests
      
    - name: Run Integration Tests
      run: mvn test -Pintegration-tests
      
    - name: Generate Coverage Report
      run: mvn jacoco:report
      
    - name: Upload Coverage to Codecov
      uses: codecov/codecov-action@v2
```

### 8. **Troubleshooting Common Issues**

#### Issue 1: Tests Not Found

```bash
# Make sure test classes end with "Test"
# Make sure they're in src/test/java
# Check Maven/Gradle configuration
```

#### Issue 2: Database Connection Issues

```bash
# Check application-test.properties
# Ensure H2 database is in test dependencies
# Verify @DataJpaTest annotation
```

#### Issue 3: Mock Issues

```bash
# Ensure @ExtendWith(MockitoExtension.class)
# Check @Mock and @InjectMocks annotations
# Verify mock behavior setup with when().thenReturn()
```

### 9. **Best Practices for Test Execution**

1. **Run tests frequently during development**
2. **Use test categories to run specific test suites**
3. **Set up continuous integration to run tests automatically**
4. **Monitor test coverage and aim for 80%+ coverage**
5. **Use test profiles for different environments**
6. **Keep integration tests separate from unit tests**
7. **Use test containers for database integration tests**

## 🎯 Key Takeaways

1. **Layered architecture promotes clean, maintainable code**
2. **Each layer should have a single responsibility**
3. **Use dependency injection to loosely couple layers**
4. **Implement proper error handling and validation**
5. **Write comprehensive tests for each layer**
6. **Use DTOs to control data flow between layers**
7. **Apply transactions appropriately in the service layer**
8. **Run tests frequently using various methods (Maven, Gradle, IDE)**
9. **Use test categories and profiles for organized test execution**
10. **Monitor test coverage and maintain high code quality**

This architecture pattern is ideal for medium to large Spring Boot applications where maintainability, testability, and scalability are important considerations.
