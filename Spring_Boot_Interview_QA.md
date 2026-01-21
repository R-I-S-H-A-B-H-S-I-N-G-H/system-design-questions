# Spring Boot Interview Questions & Answers

This document contains in-depth answers to Spring and Spring Boot interview questions.

## Table of Contents
1. [Spring Boot Basics & Core Concepts](#spring-boot-basics)
2. [Configuration & Auto-Configuration](#configuration)
3. [Spring MVC & REST APIs](#rest-apis)
4. [Data Access (JPA, Hibernate)](#data-access)
5. [Spring Security](#spring-security)
6. [Advanced Spring Boot (Actuator, Testing, Deployment)](#advanced-topics)

---

## Spring Boot Basics

### 1. What is Spring Framework?
**Answer:**
Spring is an open-source, lightweight, enterprise-level framework for Java. It provides comprehensive infrastructure support for developing Java applications. Key features include Dependency Injection (DI), Aspect-Oriented Programming (AOP), and abstraction over complex APIs (JDBC, JMS).

### 2. What problems does Spring solve?
**Answer:**
*   **Tight Coupling:** Solved via Dependency Injection (IoC).
*   **Boilerplate Code:** Reduces code for JDBC, Exception Handling, etc.
*   **Testability:** Makes code easy to unit test by injecting mock dependencies.
*   **Cross-Cutting Concerns:** Handles logging, security, transaction management via AOP.

### 3. What is Dependency Injection (DI)?
**Answer:**
DI is a design pattern where the dependencies of a class (objects it needs) are provided from the outside rather than created by the class itself.
*   **Inversion of Control (IoC):** The control of creating objects is inverted from the programmer to the container (Spring).
*   Types: Constructor Injection (Recommended), Setter Injection, Field Injection (@Autowired).

### 4. What are Spring Beans?
**Answer:**
A Spring Bean is an object that is instantiated, assembled, and managed by the Spring IoC container. They are created based on configuration metadata (XML or Annotations).

### 5. Difference between @Component, @Service, @Repository, and @Controller?
**Answer:**
All are specializations of `@Component` (Stereotypes).
*   `@Component`: Generic stereotype for any Spring-managed component.
*   `@Repository`: For DAO layer. Enables automatic translation of database exceptions.
*   `@Service`: For Business Service layer. Holds business logic.
*   `@Controller`: For Presentation layer (Spring MVC). Handles web requests.

### 6. What is @Autowired?
**Answer:**
Annotation for automatic dependency injection. Spring looks for a bean of the matching type in the context and injects it.
*   Can be used on: Constructors, Setters, Fields.
*   `required=true` by default.

### 7. What is Spring Boot? How is it different from Spring?
**Answer:**
Spring Boot is an extension of the Spring Framework that eliminates boilerplate configuration.
*   **Spring:** Requires manual setup (XML/Java Config) for DispatcherServlet, ComponentScan, ViewResolver, etc.
*   **Spring Boot:** "Opinionated". Provides Auto-Configuration, Embedded Servers (Tomcat), and Starter dependencies to get an app running quickly.

### 8. What is application.properties / application.yml used for?
**Answer:**
Central configuration file to externalize configuration properties (DB credentials, port, logging).
*   Located in `src/main/resources`.
*   Properties format: `server.port=8080`
*   YAML format: Hierarchical structure.

### 9. What is an embedded server in Spring Boot?
**Answer:**
Spring Boot includes a web server (Tomcat, Jetty, or Undertow) inside the application JAR. This means you don't need to deploy a WAR file to an external server installation. You run the app as a simple Java application (`java -jar`).

### 10. How do you create a REST API in Spring Boot?
**Answer:**
1.  Add `spring-boot-starter-web` dependency.
2.  Create a class annotated with `@RestController`.
3.  Define methods with mapping annotations (`@GetMapping`, `@PostMapping`).
```java
@RestController
public class MyController {
    @GetMapping("/hello")
    public String sayHello() { return "Hello"; }
}
```

### 11. What is @RestController vs @Controller?
**Answer:**
*   `@Controller`: Used for traditional MVC. Returns a View name (JSP/Thymeleaf).
*   `@RestController`: Used for REST APIs. Combines `@Controller` + `@ResponseBody`. Returns data (JSON/XML) directly to the response body.

### 12. What are Spring Boot Starters?
**Answer:**
Starters are dependency descriptors (POMs) that aggregate common dependencies for a specific task.
*   Example: `spring-boot-starter-web` includes Spring MVC, Jackson, Tomcat, Validation.
*   Simplifies `pom.xml` / `build.gradle`.

### 13. What is Spring Boot Auto-Configuration?
**Answer:**
Spring Boot automatically configures your Spring application based on the jar dependencies added.
*   Example: If H2 database is on classpath, it configures an in-memory DB.
*   Triggered by `@EnableAutoConfiguration` (part of `@SpringBootApplication`).
*   Implemented via `@ConditionalOnClass`, `@ConditionalOnMissingBean`.

### 14. What are Spring Boot Profiles?
**Answer:**
Mechanism to separate parts of application configuration and make it only available in certain environments (dev, test, prod).
*   Files: `application-dev.properties`, `application-prod.properties`.
*   Activation: `spring.profiles.active=dev`.

### 15. What is Spring ApplicationContext?
**Answer:**
The central interface to the Spring IoC container. It instantiates, instantiates, configures, and assembles beans.
*   Superset of `BeanFactory`. Adds AOP, Event handling, i18n.

### 16. What is Component Scanning?
**Answer:**
Process where Spring looks for classes annotated with `@Component` (and stereotypes) in specified packages and creates beans for them.
*   Configured via `@ComponentScan`.
*   In Spring Boot, the main class package is scanned by default.

### 17. What is @Configuration?
**Answer:**
Indicates that a class declares one or more `@Bean` methods and may be processed by the Spring container to generate bean definitions and service requests for those beans at runtime.

### 18. How to define beans using Java Config?
**Answer:**
```java
@Configuration
public class AppConfig {
    @Bean
    public MyService myService() {
        return new MyServiceImpl();
    }
}
```

### 19. What is @PostConstruct and @PreDestroy?
**Answer:**
Lifecycle annotations (JSR-250).
*   `@PostConstruct`: Method executed after dependency injection is done. Used for initialization.
*   `@PreDestroy`: Method executed just before the bean is destroyed (context shutdown). Used for cleanup.

### 20. Difference between Singleton and Prototype scope?
**Answer:**
*   **Singleton (Default):** Only one instance of the bean is created per Spring IoC container.
*   **Prototype:** A new instance is created every time the bean is requested.
*   Defined via `@Scope("prototype")`.


### 21. What is @Value annotation?
**Answer:**
Used to inject values from external properties files (`application.properties`) into fields.
*   Example: `@Value("${server.port}") int port;`

### 22. What are Spring Events?
**Answer:**
A mechanism for loosely coupled communication between beans using the Observer pattern.
*   **Publisher:** Publishes an event.
*   **Listener:** Annotated with `@EventListener` to handle the event.

### 23. Difference between Dependency Injection via Constructor vs Setter?
**Answer:**
*   **Constructor:** Mandatory dependencies. Ensures object is in valid state upon creation. Thread-safe (fields can be final). Recommended.
*   **Setter:** Optional dependencies. Can be changed later (mutability). Circular dependencies workaround.

### 24. How to handle exceptions in Spring MVC?
**Answer:**
1.  **Local:** `@ExceptionHandler` in the controller.
2.  **Global:** `@ControllerAdvice` / `@RestControllerAdvice` class with `@ExceptionHandler` methods.
3.  **Status:** `ResponseStatusException`.

### 25. What is @ControllerAdvice and @RestControllerAdvice?
**Answer:**
*   `@ControllerAdvice`: Global exception handling and data binding for standard Controllers (returns Views).
*   `@RestControllerAdvice`: Extends `@ControllerAdvice` and `@ResponseBody`. Used for REST APIs to return JSON errors globally.

### 26. What is @Valid and @Validated?
**Answer:**
*   `@Valid` (standard JSR-303): Triggers validation on the object.
*   `@Validated` (Spring specific): Supports validation groups and method-level validation.

### 27. What is Bean Validation (JSR 303)?
**Answer:**
Standard API for object validation via annotations.
*   `@NotNull`, `@Size`, `@Email`, `@Min`, `@Max`.
*   Implementation: Hibernate Validator (default in Spring Boot).

### 28. What is @PathVariable vs @RequestParam?
**Answer:**
*   `@PathVariable`: Extracts values from URI path segments. `/users/{id}` -> `/users/123`. Mandatory by default.
*   `@RequestParam`: Extracts query parameters. `/users?id=123`. Optional by default.

### 29. Difference between @RequestBody and @ModelAttribute?
**Answer:**
*   `@RequestBody`: Deserializes HTTP Request Body (JSON/XML) into a Java Object. Used in REST.
*   `@ModelAttribute`: Binds form data (x-www-form-urlencoded) or URI params to an object. Used in MVC.

### 30. What is @RequestMapping?
**Answer:**
Generic annotation to map HTTP requests to handler methods.
*   Shortcuts: `@GetMapping`, `@PostMapping`, `@PutMapping`, `@DeleteMapping`, `@PatchMapping`.

### 31. What is ResponseEntity?
**Answer:**
Represents the whole HTTP response: status code, headers, and body.
*   Allows full control over the response.
*   `return new ResponseEntity<>("Hello", HttpStatus.OK);`

### 32. How do you handle CORS in Spring Boot?
**Answer:**
1.  **Global:** Override `addCorsMappings` in `WebMvcConfigurer`.
2.  **Controller-level:** Add `@CrossOrigin(origins = "http://localhost:3000")` to controller/method.

### 33. How to enable Swagger / OpenAPI documentation?
**Answer:**
Add `springdoc-openapi-starter-webmvc-ui` (Spring Boot 3) or `springfox` (legacy).
*   Access UI at: `/swagger-ui.html` or `/swagger-ui/index.html`.

### 34. What is Spring Boot Actuator?
**Answer:**
A module that provides production-ready features to monitor and manage the application.
*   Endpoints: `/actuator/health`, `/actuator/info`, `/actuator/metrics`, `/actuator/loggers`.

### 35. What is spring-boot-devtools?
**Answer:**
Dependency for development-time features.
*   **Hot Swap / Live Reload:** Automatically restarts application when classpath files change.
*   Disables caching for templates.

### 36. What is Spring Data JPA?
**Answer:**
Abstraction over JPA (Java Persistence API). Reduces boilerplate code required to implement data access layers.
*   Define interfaces extending `JpaRepository`.
*   Automatic query generation from method names (`findByName`).

### 37. Difference between @Entity and @Table?
**Answer:**
*   `@Entity`: Marks class as a JPA entity (mapped to DB). Mandatory.
*   `@Table`: Customizes table name, schema, indexes. Optional (defaults to class name).

### 38. What is @Id and @GeneratedValue?
**Answer:**
*   `@Id`: Primary Key.
*   `@GeneratedValue`: Generation strategy (`AUTO`, `IDENTITY`, `SEQUENCE`, `TABLE`).

### 39. Difference between GenerationType.IDENTITY and AUTO?
**Answer:**
*   `IDENTITY`: Uses database auto-increment column (MySQL, PG).
*   `AUTO`: JPA chooses appropriate strategy (often SEQUENCE or TABLE).

### 40. What is @Column used for?
**Answer:**
Customize column mapping: name, length, nullable, unique.
*   `@Column(name="user_email", unique=true)`


### 41. Difference between @OneToMany and @ManyToOne?
**Answer:**
Relationship mapping.
*   `@ManyToOne`: Child side (owns foreign key). E.g., Employee -> Department.
*   `@OneToMany`: Parent side (collection of children). E.g., Department -> List<Employee>.
*   Best practice: Use Bidirectional mapping with `mappedBy` on OneToMany side.

### 42. Lazy vs Eager fetching?
**Answer:**
*   **Eager:** Loads related entities immediately with the parent. (Default for `@ManyToOne`, `@OneToOne`).
*   **Lazy:** Loads related entities only when accessed (proxy). (Default for `@OneToMany`, `@ManyToMany`). Better for performance.

### 43. Difference between CrudRepository and JpaRepository?
**Answer:**
*   **CrudRepository:** Basic CRUD operations (save, find, delete).
*   **PagingAndSortingRepository:** Adds pagination and sorting.
*   **JpaRepository:** Extends both. Adds JPA specific methods (`flush`, `saveAndFlush`, batch delete).

### 44. How to write custom queries with @Query?
**Answer:**
Used when method name derivation is insufficient.
*   JPQL: `@Query("SELECT u FROM User u WHERE u.status = 1")`
*   Native: `@Query(value = "SELECT * FROM users u WHERE u.status = 1", nativeQuery = true)`

### 45. What is @Transactional?
**Answer:**
Defines the scope of a database transaction.
*   If method succeeds, transaction commits.
*   If RuntimeException occurs, transaction rolls back.
*   Parameters: `propagation`, `isolation`, `readOnly`, `rollbackFor`.

### 46. Transaction Propagation types?
**Answer:**
*   `REQUIRED` (Default): Use existing transaction or create new one.
*   `REQUIRES_NEW`: Suspend existing and create new independent transaction.
*   `SUPPORTS`, `MANDATORY`, `NEVER`, `NOT_SUPPORTED`.

### 47. Transaction Isolation levels?
**Answer:**
Defines visibility of changes to other transactions.
*   `READ_UNCOMMITTED` (Dirty reads).
*   `READ_COMMITTED` (No dirty reads).
*   `REPEATABLE_READ` (No non-repeatable reads).
*   `SERIALIZABLE` (Highest, slowest).

### 48. What is Spring Security?
**Answer:**
Framework for Authentication (Who are you?) and Authorization (What can you do?).
*   Based on Servlet Filters (FilterChain).
*   Key components: `SecurityContextHolder`, `AuthenticationManager`, `UserDetailsService`.

### 49. Difference between Authentication and Authorization?
**Answer:**
*   **Authentication (AuthN):** Verifying identity (Login).
*   **Authorization (AuthZ):** Verifying access rights (Permissions/Roles).

### 50. How to implement JWT authentication?
**Answer:**
1.  **Login:** Validate credentials -> Generate JWT (Header + Payload + Signature) -> Send to client.
2.  **Request:** Client sends JWT in `Authorization: Bearer` header.
3.  **Filter:** Intercept request -> Validate JWT signature -> Extract User -> Set `Authentication` in `SecurityContext`.

### 51. What is UserDetailsService?
**Answer:**
Core interface to load user-specific data.
*   Method: `loadUserByUsername(String username)`.
*   Returns: `UserDetails` (password, authorities).

### 52. What is BCryptPasswordEncoder?
**Answer:**
Implementation of PasswordEncoder that uses the BCrypt strong hashing function. Recommended for password storage (includes salt automatically).

### 53. How to protect endpoints with roles?
**Answer:**
1.  **Config:** `.requestMatchers("/admin/**").hasRole("ADMIN")`
2.  **Annotation:** `@PreAuthorize("hasRole('ADMIN')")` (Requires `@EnableMethodSecurity`).

### 54. What is CSRF and how to handle it?
**Answer:**
**Cross-Site Request Forgery.** Attack forcing user to execute unwanted actions.
*   Spring Security enables CSRF protection by default (Sync Token pattern).
*   **REST APIs:** Usually disabled (`csrf.disable()`) because they are stateless and use tokens (JWT) instead of session cookies.

### 55. What is Spring Boot Logging?
**Answer:**
*   Default: SLF4J + Logback.
*   Config: `application.properties` (`logging.level.root=WARN`, `logging.file.name=app.log`).

### 56. Difference between YAML and Properties?
**Answer:**
*   **Properties:** Key-value pairs. Flat. `a.b.c=d`.
*   **YAML:** Hierarchical, readable. Supports lists and maps natively.

### 57. What is embedded Tomcat, Jetty, or Undertow?
**Answer:**
Servlets containers packaged inside the app JAR.
*   **Tomcat:** Default. Robust.
*   **Jetty:** Lightweight. Good for WebSockets.
*   **Undertow:** Non-blocking (NIO), high performance.

### 58. How to create a REST API with pagination and sorting?
**Answer:**
Use `Pageable` interface in Controller and Repository.
```java
@GetMapping
public Page<User> getUsers(Pageable pageable) {
    return userRepository.findAll(pageable);
}
// Request: /users?page=0&size=10&sort=name,desc
```

### 59. How to deploy Spring Boot as standalone JAR or WAR?
**Answer:**
*   **JAR (Default):** Includes embedded server. Run with `java -jar app.jar`.
*   **WAR:** Exclude embedded tomcat (`provided` scope), extend `SpringBootServletInitializer`. Deploy to external Tomcat/Wildfly.

### 60. What is Spring WebFlux?
**Answer:**
Reactive-stack web framework (non-blocking).
*   Based on Project Reactor (Mono/Flux).
*   Runs on Netty (default).
*   Handles high concurrency with fewer threads.

### 61. Difference between Mono and Flux?
**Answer:**
*   **Mono:** Publisher that emits 0 or 1 item.
*   **Flux:** Publisher that emits 0 to N items.


---

## Advanced Spring Boot

### 62. What is Reactive Programming?
**Answer:**
A programming paradigm oriented around data flows and the propagation of change.
*   **Non-blocking:** Threads don't wait for I/O.
*   **Asynchronous:** Events are pushed to consumers.
*   **Backpressure:** Consumers can signal producers to slow down.

### 63. How to handle Backpressure in reactive streams?
**Answer:**
Backpressure allows a consumer to signal the producer how much data it can handle.
*   `onBackpressureBuffer()`: Buffer overflow events.
*   `onBackpressureDrop()`: Drop events if consumer is busy.
*   `onBackpressureLatest()`: Keep only the latest.

### 64. Difference between RestTemplate and WebClient?
**Answer:**
*   **RestTemplate:** Blocking, Synchronous. Deprecated (maintenance mode).
*   **WebClient:** Non-blocking, Asynchronous. Supports Reactive Streams. Recommended for new development.

### 65. What is Spring Security Filter Chain?
**Answer:**
Spring Security maintains a chain of filters that intercept every request.
*   Examples: `CorsFilter`, `CsrfFilter`, `UsernamePasswordAuthenticationFilter`, `BearerTokenAuthenticationFilter`.
*   The request must pass through all filters to reach the servlet.

### 66. How to implement OAuth2 Client Credentials Flow?
**Answer:**
Used for service-to-service communication (no user).
*   Config: `spring.security.oauth2.client.registration.my-client.authorization-grant-type=client_credentials`.
*   Usage: Inject `OAuth2AuthorizedClientManager` to fetch tokens automatically.

### 67. How to customize Swagger UI?
**Answer:**
Using `application.properties` or `@OpenAPIDefinition`.
*   `springdoc.swagger-ui.path=/api-docs`
*   `springdoc.swagger-ui.operations-sorter=method`

### 68. What are Liveness and Readiness Probes?
**Answer:**
Health checks used by Kubernetes.
*   **Liveness:** "Is the app running?" If fails, K8s restarts the pod. (e.g., Deadlock).
*   **Readiness:** "Is the app ready to accept traffic?" If fails, K8s stops sending traffic. (e.g., Database connecting, Cache warming).
*   Enabled via `management.endpoint.health.probes.enabled=true`.

### 69. How to use Micrometer?
**Answer:**
Vendor-neutral application metrics facade.
*   Spring Boot autoconfigures it.
*   Supports backends like Prometheus, Datadog, New Relic.
*   `MeterRegistry.counter("my.counter").increment();`

### 70. How to implement structured logging?
**Answer:**
Outputting logs in JSON format for easier parsing by ELK/Splunk.
*   Add `logstash-logback-encoder` dependency.
*   Configure `logback-spring.xml` to use `LogstashEncoder`.

### 71. What is Spring Boot with Docker?
**Answer:**
Containerizing the application.
*   **Dockerfile:**
    ```dockerfile
    FROM openjdk:17-alpine
    COPY target/app.jar app.jar
    ENTRYPOINT ["java", "-jar", "/app.jar"]
    ```
*   **Buildpacks:** `mvn spring-boot:build-image` (No Dockerfile needed).

### 72. How to handle Distributed Tracing with Zipkin?
**Answer:**
*   Add `spring-cloud-starter-sleuth` (or Micrometer Tracing in Boot 3) and `spring-cloud-sleuth-zipkin`.
*   Config: `spring.zipkin.baseUrl=http://localhost:9411`.
*   Sleuth adds TraceID and SpanID to logs. Zipkin collects and visualizes them.

### 73. What is @Async annotation?
**Answer:**
Runs a method in a separate thread.
*   Requires `@EnableAsync`.
*   Method must be `public` and return `void` or `Future`.
*   `@Async` methods are proxied; calling them from within the same class won't work.

### 74. How to implement Scheduled Tasks?
**Answer:**
*   Enable: `@EnableScheduling`.
*   Annotate: `@Scheduled(fixedRate = 1000)` or `@Scheduled(cron = "0 * * * * *")`.

### 75. How to implement Batch Processing (Spring Batch)?
**Answer:**
Framework for robust execution of batch jobs.
*   **Job:** The entire batch process.
*   **Step:** A phase in the job.
*   **ItemReader:** Reads data.
*   **ItemProcessor:** Transforms data.
*   **ItemWriter:** Writes data.
*   **Chunk-oriented:** Reads/Processes/Writes in chunks (transactions).

### 76. What is Spring Boot DevTools?
**Answer:**
Provides developer-friendly features:
*   Automatic restart on code changes.
*   LiveReload.
*   Disables template caching.
*   H2 Console enabled by default.


### 77. How to implement Spring Boot with Kubernetes?
**Answer:**
*   **Containerize:** Build Docker image (`mvn spring-boot:build-image`).
*   **Deployment:** Create `deployment.yaml` (replicas, image, ports).
*   **Service:** Create `service.yaml` (LoadBalancer/ClusterIP) to expose the app.
*   **Config:** Use `ConfigMap` for `application.properties` and `Secret` for passwords.

### 78. How to handle Configuration Management in Kubernetes?
**Answer:**
Use `ConfigMap`.
1.  Create ConfigMap: `kubectl create configmap my-config --from-file=application.properties`.
2.  Mount in Pod:
    ```yaml
    envFrom:
      - configMapRef:
          name: my-config
    ```
*   Spring Cloud Kubernetes can also read ConfigMaps directly via API.

### 79. How to implement Distributed Caching with Redis?
**Answer:**
1.  Dependency: `spring-boot-starter-data-redis`.
2.  Config: `spring.data.redis.host=localhost`.
3.  Usage:
    *   `@EnableCaching`.
    *   `@Cacheable(value = "users", key = "#id")`.
    *   Spring automatically serializes objects to Redis.

### 80. How to implement Messaging with Kafka in Spring Boot?
**Answer:**
1.  Dependency: `spring-kafka`.
2.  **Producer:** `KafkaTemplate.send("topic", "message")`.
3.  **Consumer:**
    ```java
    @KafkaListener(topics = "topic", groupId = "group_id")
    public void listen(String message) { ... }
    ```
4.  Config: `spring.kafka.bootstrap-servers=localhost:9092`.

### 81. How to implement Exception Handling in Spring Batch?
**Answer:**
*   **Skip Logic:** Configure step to skip specific exceptions (e.g., `FlatFileParseException`) up to a limit.
    `.faultTolerant().skip(Exception.class).skipLimit(10)`
*   **Retry Logic:** Retry failed items.
    `.retry(ConnectException.class).retryLimit(3)`

### 82. What is @SpringBootTest?
**Answer:**
Annotation for integration testing. It starts the full application context (including server if defined).
*   `@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)`
*   Inject `TestRestTemplate` or `WebTestClient` to make actual HTTP calls.

### 83. How to mock dependencies in tests?
**Answer:**
Use `@MockBean`.
*   Replaces a bean in the Spring Context with a Mockito mock.
*   `given(userRepository.findById(1)).willReturn(Optional.of(user));`

### 84. What is Slice Testing?
**Answer:**
Testing only a specific layer of the application, mocking the rest. Faster than `@SpringBootTest`.
*   `@WebMvcTest`: Tests Controllers (mocks Service/Repo).
*   `@DataJpaTest`: Tests Repositories (uses embedded DB).
*   `@JsonTest`: Tests JSON serialization.

### 85. How to handle File Uploads?
**Answer:**
1.  Controller takes `MultipartFile`.
2.  `file.transferTo(path)`.
3.  Config: `spring.servlet.multipart.max-file-size=10MB`.

