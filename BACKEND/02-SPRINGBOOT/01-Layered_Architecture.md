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

## 🎯 Key Takeaways

1. **Layered architecture promotes clean, maintainable code**
2. **Each layer should have a single responsibility**
3. **Use dependency injection to loosely couple layers**
4. **Implement proper error handling and validation**
5. **Write comprehensive tests for each layer**
6. **Use DTOs to control data flow between layers**
7. **Apply transactions appropriately in the service layer**

This architecture pattern is ideal for medium to large Spring Boot applications where maintainability, testability, and scalability are important considerations.
