# Spring Boot Annotations - Complete Guide

A comprehensive guide to essential Spring Boot annotations with practical examples, best practices, and common use cases.

## Table of Contents

1. [Controller Layer Annotations](#controller-layer-annotations)
2. [Service Layer Annotations](#service-layer-annotations)
3. [Data Access Annotations](#data-access-annotations)
4. [Configuration Annotations](#configuration-annotations)
5. [Validation Annotations](#validation-annotations)
6. [Best Practices](#best-practices)
7. [Common Pitfalls](#common-pitfalls)

---

## Controller Layer Annotations

### @Controller vs @RestController

#### @Controller

Traditional MVC controller that returns view names. Used when you need to render HTML pages.

```java
@Controller
@RequestMapping("/web")
public class WebController {
    
    @GetMapping("/home")
    public String homePage(Model model) {
        model.addAttribute("message", "Welcome to Spring Boot!");
        return "home"; // Returns view name (home.html)
    }
    
    @PostMapping("/submit")
    public String submitForm(@ModelAttribute User user, Model model) {
        // Process form data
        model.addAttribute("user", user);
        return "result";
    }
}
```

#### @RestController

Combines `@Controller` and `@ResponseBody`. Perfect for REST APIs that return JSON/XML data.

```java
@RestController
@RequestMapping("/api/v1")
public class UserRestController {
    
    @Autowired
    private UserService userService;
    
    @GetMapping("/users")
    public List<User> getAllUsers() {
        return userService.findAllUsers();
    }
    
    @GetMapping("/users/{id}")
    public ResponseEntity<User> getUserById(@PathVariable Long id) {
        User user = userService.findById(id);
        return ResponseEntity.ok(user);
    }
}
```

**Best Practice**: Use `@RestController` for APIs and `@Controller` for web pages.

---

### @RequestMapping and Its Variants

#### @RequestMapping

The parent annotation for mapping HTTP requests to handler methods.

```java
@RestController
@RequestMapping("/api/products")
public class ProductController {
    
    // Basic mapping
    @RequestMapping(value = "/search", method = RequestMethod.GET)
    public List<Product> searchProducts() {
        return productService.findAll();
    }
    
    // Multiple HTTP methods
    @RequestMapping(value = "/bulk", method = {RequestMethod.POST, RequestMethod.PUT})
    public ResponseEntity<String> bulkOperation(@RequestBody List<Product> products) {
        // Handle bulk operations
        return ResponseEntity.ok("Bulk operation completed");
    }
    
    // With headers and params
    @RequestMapping(
        value = "/advanced", 
        method = RequestMethod.GET,
        headers = "Accept=application/json",
        params = "version=v1"
    )
    public ResponseEntity<Product> advancedSearch() {
        // Advanced search logic
        return ResponseEntity.ok(new Product());
    }
}
```

#### Specialized Mapping Annotations

```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    @GetMapping // Equivalent to @RequestMapping(method = RequestMethod.GET)
    public List<User> getUsers() {
        return userService.findAll();
    }
    
    @PostMapping
    public ResponseEntity<User> createUser(@RequestBody @Valid User user) {
        User savedUser = userService.save(user);
        return ResponseEntity.status(HttpStatus.CREATED).body(savedUser);
    }
    
    @PutMapping("/{id}")
    public ResponseEntity<User> updateUser(@PathVariable Long id, @RequestBody @Valid User user) {
        User updatedUser = userService.update(id, user);
        return ResponseEntity.ok(updatedUser);
    }
    
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
        userService.delete(id);
        return ResponseEntity.noContent().build();
    }
    
    @PatchMapping("/{id}/status")
    public ResponseEntity<User> updateUserStatus(@PathVariable Long id, @RequestParam String status) {
        User user = userService.updateStatus(id, status);
        return ResponseEntity.ok(user);
    }
}
```

---

### Parameter Binding Annotations

#### @RequestParam

Extract query parameters, form data, or multipart file data.

```java
@RestController
@RequestMapping("/api/search")
public class SearchController {
    
    // Basic usage
    @GetMapping("/users")
    public List<User> searchUsers(@RequestParam String name) {
        return userService.findByName(name);
    }
    
    // Optional parameter with default value
    @GetMapping("/products")
    public Page<Product> getProducts(
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "10") int size,
            @RequestParam(required = false) String category) {
        return productService.findProducts(page, size, category);
    }
    
    // Multiple values
    @GetMapping("/items")
    public List<Item> getItemsByIds(@RequestParam List<Long> ids) {
        return itemService.findByIds(ids);
    }
    
    // Map for dynamic parameters
    @GetMapping("/filter")
    public List<Product> filterProducts(@RequestParam Map<String, String> filters) {
        return productService.filter(filters);
    }
}
```

#### @PathVariable

Extract values from URI path.

```java
@RestController
@RequestMapping("/api")
public class ResourceController {
    
    // Basic path variable
    @GetMapping("/users/{id}")
    public User getUser(@PathVariable Long id) {
        return userService.findById(id);
    }
    
    // Multiple path variables
    @GetMapping("/users/{userId}/orders/{orderId}")
    public Order getUserOrder(@PathVariable Long userId, @PathVariable Long orderId) {
        return orderService.findByUserAndOrderId(userId, orderId);
    }
    
    // Optional path variable
    @GetMapping({"/products", "/products/{category}"})
    public List<Product> getProducts(@PathVariable(required = false) String category) {
        return category != null ? 
            productService.findByCategory(category) : 
            productService.findAll();
    }
    
    // Path variable with regex
    @GetMapping("/files/{filename:.+}")
    public ResponseEntity<Resource> downloadFile(@PathVariable String filename) {
        Resource file = fileService.loadFile(filename);
        return ResponseEntity.ok().body(file);
    }
}
```

#### @RequestBody

Bind HTTP request body to Java object.

```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    @PostMapping
    public ResponseEntity<User> createUser(@RequestBody @Valid User user) {
        User savedUser = userService.save(user);
        return ResponseEntity.status(HttpStatus.CREATED).body(savedUser);
    }
    
    @PutMapping("/{id}")
    public ResponseEntity<User> updateUser(
            @PathVariable Long id, 
            @RequestBody @Valid UserUpdateRequest request) {
        User updatedUser = userService.update(id, request);
        return ResponseEntity.ok(updatedUser);
    }
    
    // Handling raw JSON
    @PostMapping("/raw")
    public ResponseEntity<String> processRawJson(@RequestBody String rawJson) {
        // Process raw JSON string
        return ResponseEntity.ok("Processed: " + rawJson);
    }
}
```

---

### Response Handling

#### @ResponseEntity

Customize HTTP responses with status codes, headers, and body.

```java
@RestController
@RequestMapping("/api/files")
public class FileController {
    
    @GetMapping("/{id}")
    public ResponseEntity<FileResource> downloadFile(@PathVariable Long id) {
        try {
            FileResource file = fileService.getFile(id);
            
            return ResponseEntity.ok()
                    .header(HttpHeaders.CONTENT_DISPOSITION, "attachment; filename=\"" + file.getFilename() + "\"")
                    .header(HttpHeaders.CONTENT_TYPE, file.getContentType())
                    .body(file);
                    
        } catch (FileNotFoundException e) {
            return ResponseEntity.notFound().build();
        } catch (Exception e) {
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).build();
        }
    }
    
    @PostMapping("/upload")
    public ResponseEntity<?> uploadFile(@RequestParam("file") MultipartFile file) {
        if (file.isEmpty()) {
            return ResponseEntity.badRequest()
                    .body(Map.of("error", "File is empty"));
        }
        
        try {
            String fileId = fileService.save(file);
            return ResponseEntity.status(HttpStatus.CREATED)
                    .body(Map.of("fileId", fileId, "message", "File uploaded successfully"));
        } catch (Exception e) {
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
                    .body(Map.of("error", "Failed to upload file"));
        }
    }
}
```

---

### Data Binding Customization

#### @InitBinder

Customize data binding for web requests.

```java
@Controller
public class UserFormController {
    
    @InitBinder
    public void initBinder(WebDataBinder binder) {
        // Custom date format
        SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");
        dateFormat.setLenient(false);
        binder.registerCustomEditor(Date.class, new CustomDateEditor(dateFormat, false));
        
        // Trim strings
        binder.registerCustomEditor(String.class, new StringTrimmerEditor(true));
        
        // Exclude sensitive fields from binding
        binder.setDisallowedFields("password", "id");
    }
    
    @InitBinder("user")
    public void initUserBinder(WebDataBinder binder) {
        // Specific binding rules for User objects
        binder.setRequiredFields("name", "email");
    }
    
    @PostMapping("/register")
    public String registerUser(@ModelAttribute @Valid User user, BindingResult result) {
        if (result.hasErrors()) {
            return "registration-form";
        }
        userService.save(user);
        return "redirect:/success";
    }
}
```

---

## Service Layer Annotations

### @Service

Mark service layer components.

```java
@Service
@Transactional(readOnly = true) // Default read-only transactions
public class UserService {
    
    private final UserRepository userRepository;
    private final EmailService emailService;
    
    // Constructor injection (preferred)
    public UserService(UserRepository userRepository, EmailService emailService) {
        this.userRepository = userRepository;
        this.emailService = emailService;
    }
    
    @Transactional // Override for write operations
    public User createUser(User user) {
        // Validate user
        validateUser(user);
        
        // Save user
        User savedUser = userRepository.save(user);
        
        // Send welcome email
        emailService.sendWelcomeEmail(savedUser.getEmail());
        
        return savedUser;
    }
    
    public Optional<User> findById(Long id) {
        return userRepository.findById(id);
    }
    
    private void validateUser(User user) {
        if (userRepository.existsByEmail(user.getEmail())) {
            throw new UserAlreadyExistsException("Email already exists: " + user.getEmail());
        }
    }
}
```

### @Component

Generic stereotype annotation for Spring-managed components.

```java
@Component
public class FileProcessor {
    
    @Value("${app.upload.directory}")
    private String uploadDirectory;
    
    @EventListener
    public void handleFileUploadEvent(FileUploadEvent event) {
        // Process file upload
        processFile(event.getFile());
    }
    
    private void processFile(MultipartFile file) {
        // File processing logic
    }
}
```

---

## Data Access Annotations

### @Repository

Mark data access layer components with exception translation.

```java
@Repository
public class CustomUserRepository {
    
    @PersistenceContext
    private EntityManager entityManager;
    
    public List<User> findActiveUsersByRole(String role) {
        return entityManager.createQuery(
                "SELECT u FROM User u WHERE u.active = true AND u.role = :role", User.class)
                .setParameter("role", role)
                .getResultList();
    }
    
    public List<UserStatistics> getUserStatistics() {
        return entityManager.createNativeQuery(
                "SELECT u.role, COUNT(*) as count FROM users u GROUP BY u.role",
                "UserStatisticsMapping")
                .getResultList();
    }
}
```

### JPA Annotations Example

```java
@Entity
@Table(name = "users")
@EntityListeners(AuditingEntityListener.class)
public class User {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false, unique = true, length = 100)
    @Email
    private String email;
    
    @Column(nullable = false, length = 50)
    @Size(min = 2, max = 50)
    private String name;
    
    @Enumerated(EnumType.STRING)
    private UserStatus status;
    
    @CreatedDate
    private LocalDateTime createdDate;
    
    @LastModifiedDate
    private LocalDateTime lastModifiedDate;
    
    @OneToMany(mappedBy = "user", cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    private List<Order> orders = new ArrayList<>();
    
    // Constructors, getters, setters
}
```

---

## Configuration Annotations

### @Configuration and @Bean

```java
@Configuration
@EnableWebSecurity
@EnableJpaAuditing
public class ApplicationConfig {
    
    @Bean
    @Primary
    public ObjectMapper objectMapper() {
        return new ObjectMapper()
                .configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false)
                .configure(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS, false)
                .registerModule(new JavaTimeModule());
    }
    
    @Bean
    @ConditionalOnProperty(name = "app.cache.enabled", havingValue = "true")
    public CacheManager cacheManager() {
        return new ConcurrentMapCacheManager("users", "products");
    }
    
    @Bean
    @ConfigurationProperties(prefix = "app.external.api")
    public ExternalApiConfig externalApiConfig() {
        return new ExternalApiConfig();
    }
}
```

### @Profile

Environment-specific configurations.

```java
@Configuration
public class DatabaseConfig {
    
    @Bean
    @Profile("dev")
    public DataSource devDataSource() {
        HikariConfig config = new HikariConfig();
        config.setJdbcUrl("jdbc:h2:mem:devdb");
        config.setUsername("sa");
        config.setPassword("");
        return new HikariDataSource(config);
    }
    
    @Bean
    @Profile("prod")
    public DataSource prodDataSource(
            @Value("${spring.datasource.url}") String url,
            @Value("${spring.datasource.username}") String username,
            @Value("${spring.datasource.password}") String password) {
        
        HikariConfig config = new HikariConfig();
        config.setJdbcUrl(url);
        config.setUsername(username);
        config.setPassword(password);
        config.setMaximumPoolSize(20);
        config.setMinimumIdle(5);
        return new HikariDataSource(config);
    }
}
```

---

## Validation Annotations

### Built-in Validation

```java
@Entity
public class Product {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @NotBlank(message = "Product name is required")
    @Size(min = 2, max = 100, message = "Product name must be between 2 and 100 characters")
    private String name;
    
    @NotNull(message = "Price is required")
    @DecimalMin(value = "0.01", message = "Price must be greater than 0")
    @Digits(integer = 10, fraction = 2, message = "Price format is invalid")
    private BigDecimal price;
    
    @Min(value = 0, message = "Stock cannot be negative")
    private Integer stock;
    
    @Email(message = "Invalid email format")
    private String contactEmail;
    
    @Pattern(regexp = "^[A-Z]{2,3}-\\d{4,6}$", message = "Invalid product code format")
    private String productCode;
    
    @Past(message = "Manufacturing date must be in the past")
    private LocalDate manufacturedDate;
    
    // Getters and setters
}
```

### Custom Validation

```java
@Target({ElementType.FIELD, ElementType.METHOD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = UniqueEmailValidator.class)
@Documented
public @interface UniqueEmail {
    String message() default "Email already exists";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}

@Component
public class UniqueEmailValidator implements ConstraintValidator<UniqueEmail, String> {
    
    @Autowired
    private UserRepository userRepository;
    
    @Override
    public boolean isValid(String email, ConstraintValidatorContext context) {
        return email != null && !userRepository.existsByEmail(email);
    }
}

// Usage
@Entity
public class User {
    @UniqueEmail
    @Email
    private String email;
}
```

---

## Best Practices

### 1. Controller Layer Best Practices

```java
@RestController
@RequestMapping("/api/v1/users")
@Validated // Enable method-level validation
@Slf4j
public class UserController {
    
    private final UserService userService;
    
    // Use constructor injection
    public UserController(UserService userService) {
        this.userService = userService;
    }
    
    @GetMapping
    public ResponseEntity<Page<UserDTO>> getUsers(
            @RequestParam(defaultValue = "0") @Min(0) int page,
            @RequestParam(defaultValue = "20") @Min(1) @Max(100) int size,
            @RequestParam(required = false) String search) {
        
        log.info("Fetching users - page: {}, size: {}, search: {}", page, size, search);
        
        Page<UserDTO> users = userService.findUsers(page, size, search);
        return ResponseEntity.ok(users);
    }
    
    @PostMapping
    public ResponseEntity<UserDTO> createUser(@RequestBody @Valid CreateUserRequest request) {
        log.info("Creating user with email: {}", request.getEmail());
        
        UserDTO user = userService.createUser(request);
        
        return ResponseEntity.status(HttpStatus.CREATED)
                .location(URI.create("/api/v1/users/" + user.getId()))
                .body(user);
    }
}
```

### 2. Service Layer Best Practices

```java
@Service
@Transactional(readOnly = true)
@Slf4j
public class UserService {
    
    private final UserRepository userRepository;
    private final UserMapper userMapper;
    private final ApplicationEventPublisher eventPublisher;
    
    public UserService(UserRepository userRepository, 
                      UserMapper userMapper, 
                      ApplicationEventPublisher eventPublisher) {
        this.userRepository = userRepository;
        this.userMapper = userMapper;
        this.eventPublisher = eventPublisher;
    }
    
    @Transactional
    public UserDTO createUser(CreateUserRequest request) {
        // Validate business rules
        validateCreateUserRequest(request);
        
        // Create and save user
        User user = userMapper.toEntity(request);
        User savedUser = userRepository.save(user);
        
        // Publish domain event
        eventPublisher.publishEvent(new UserCreatedEvent(savedUser.getId()));
        
        log.info("Created user with id: {}", savedUser.getId());
        
        return userMapper.toDTO(savedUser);
    }
    
    private void validateCreateUserRequest(CreateUserRequest request) {
        if (userRepository.existsByEmail(request.getEmail())) {
            throw new BusinessException("Email already exists: " + request.getEmail());
        }
    }
}
```

### 3. Exception Handling Best Practices

```java
@RestControllerAdvice
@Slf4j
public class GlobalExceptionHandler {
    
    @ExceptionHandler(ValidationException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ErrorResponse handleValidation(ValidationException ex) {
        log.warn("Validation error: {}", ex.getMessage());
        return new ErrorResponse("VALIDATION_ERROR", ex.getMessage());
    }
    
    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ErrorResponse handleMethodArgumentNotValid(MethodArgumentNotValidException ex) {
        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getFieldErrors().forEach(error -> 
            errors.put(error.getField(), error.getDefaultMessage()));
        
        return new ErrorResponse("VALIDATION_ERROR", "Invalid input", errors);
    }
    
    @ExceptionHandler(EntityNotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public ErrorResponse handleNotFound(EntityNotFoundException ex) {
        log.warn("Entity not found: {}", ex.getMessage());
        return new ErrorResponse("NOT_FOUND", ex.getMessage());
    }
}
```

---

## Common Pitfalls

### ❌ Don't Do This

```java
// 1. Using field injection
@RestController
public class BadController {
    @Autowired
    private UserService userService; // Avoid field injection
    
    // 2. Not using ResponseEntity for REST APIs
    @GetMapping("/users/{id}")
    public User getUser(@PathVariable Long id) { // Missing proper response handling
        return userService.findById(id); // What if user not found?
    }
    
    // 3. Not validating input
    @PostMapping("/users")
    public User createUser(@RequestBody User user) { // Missing @Valid
        return userService.save(user);
    }
}

// 4. Exposing entities directly
@Entity
public class User {
    private String password; // Sensitive data exposed in JSON
    // ... other fields
}
```

### ✅ Do This Instead

```java
// 1. Use constructor injection
@RestController
@RequestMapping("/api/users")
public class GoodController {
    
    private final UserService userService;
    
    public GoodController(UserService userService) {
        this.userService = userService;
    }
    
    // 2. Use ResponseEntity with proper error handling
    @GetMapping("/{id}")
    public ResponseEntity<UserDTO> getUser(@PathVariable Long id) {
        return userService.findById(id)
                .map(ResponseEntity::ok)
                .orElse(ResponseEntity.notFound().build());
    }
    
    // 3. Always validate input
    @PostMapping
    public ResponseEntity<UserDTO> createUser(@RequestBody @Valid CreateUserRequest request) {
        UserDTO user = userService.createUser(request);
        return ResponseEntity.status(HttpStatus.CREATED).body(user);
    }
}

// 4. Use DTOs to control data exposure
public class UserDTO {
    private Long id;
    private String name;
    private String email;
    // No password field - secure!
}
```

---

## Additional Resources

- [Spring Boot Documentation](https://docs.spring.io/spring-boot/docs/current/reference/html/)
- [Spring Framework Annotations](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-annotation-config)
- [Bean Validation Specification](https://beanvalidation.org/2.0/spec/)
- [Spring Security Reference](https://docs.spring.io/spring-security/reference/)

---

*This guide covers the most commonly used Spring Boot annotations. For more advanced use cases, refer to the official Spring documentation.*
