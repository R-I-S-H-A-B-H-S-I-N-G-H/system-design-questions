# Spring Boot Interview Questions & Answers (In-Depth Guide)

This document provides comprehensive, deep-dive answers to Spring Boot and Spring Framework interview questions. It focuses on internal mechanisms, architectural decisions, and production-grade implementations.

## Table of Contents
1.  [Spring Core & Dependency Injection](#spring-core)
2.  [Spring Boot Internals & Auto-Configuration](#boot-internals)
3.  [Spring MVC & REST APIs (Deep Dive)](#spring-mvc)
4.  [Data Access (JPA, Transaction Management)](#data-access)
5.  [Spring Security (Architecture)](#security)

---

## <a name="spring-core"></a>1. Spring Core & Dependency Injection

### Q1: What is Inversion of Control (IoC) and Dependency Injection (DI)?
**Deep Dive:**
*   **IoC (Principle):** It is a software design principle where the control flow of a program is inverted. Instead of the programmer manually creating objects (`new Service()`) and managing their lifecycle, a container (Spring) takes charge.
*   **DI (Pattern):** Dependency Injection is a specific implementation of IoC. It is the process where the container "injects" the required dependencies into a bean at runtime.

**Why use it?**
1.  **Decoupling:** `OrderService` doesn't need to know how to construct `PaymentRepository`. It just asks for an interface.
2.  **Testing:** You can easily swap the real `PaymentRepository` with a `MockPaymentRepository` during Unit Testing.

**Types of DI:**
1.  **Constructor Injection (Best Practice):**
    *   Dependencies are passed via constructor.
    *   **Pros:** Ensures immutability (fields can be `final`). Ensures the object is fully initialized and valid before use.
2.  **Setter Injection:**
    *   Dependencies passed via setter methods.
    *   **Pros:** Useful for optional dependencies or breaking circular dependencies.
3.  **Field Injection (`@Autowired` on field):**
    *   **Cons:** Not recommended. Makes testing hard (cannot pass mock in constructor). Hides dependencies.

---

### Q2: What is the Spring Bean Lifecycle?
**Detailed Workflow:**
When the container starts, a Bean goes through a complex lifecycle:

1.  **Instantiation:** Spring creates the object instance (like `new Class()`).
2.  **Populate Properties:** Spring injects dependencies (DI).
3.  **BeanNameAware / BeanFactoryAware:** If the bean implements these interfaces, Spring passes the bean's ID or the Factory instance.
4.  **Pre-Initialization (BeanPostProcessor - Before):** Custom logic before init.
5.  **Initialization:**
    *   Methods annotated with `@PostConstruct` run.
    *   `InitializingBean.afterPropertiesSet()` runs.
    *   Custom `init-method` (defined in XML) runs.
6.  **Post-Initialization (BeanPostProcessor - After):** This is where AOP Proxies are often created (wrapping the bean).
7.  **Ready for Use:** The bean is in the application context.
8.  **Destruction (Shutdown):**
    *   `@PreDestroy` runs.
    *   `DisposableBean.destroy()` runs.

**Interview Use Case:** "Where would you place logic to validate that a bean was configured correctly?"
**Answer:** In `@PostConstruct`, because at that point all dependencies have been injected.

---

### Q3: How does `@Transactional` work internally?
**Mechanism: AOP & Proxies**
When you annotate a method with `@Transactional`, Spring does not change your class code. Instead, it creates a **Proxy** (a wrapper object) around your bean.

**Workflow:**
1.  **Caller** calls `service.doPayment()`.
2.  **Proxy Intercepts:** The proxy intercepts the call.
3.  **Transaction Advisor:**
    *   Asks `TransactionManager` to **start** a new transaction (or join existing).
    *   `connection.setAutoCommit(false)`.
4.  **Actual Method:** The proxy calls your real `doPayment()` method.
5.  **Success:** If method returns, Proxy asks TransactionManager to **commit**.
6.  **Failure:** If method throws `RuntimeException` (unchecked), Proxy asks TransactionManager to **rollback**.
    *   *Note:* By default, Checked Exceptions (`IOException`) do **NOT** trigger rollback. You must configure `rollbackFor = Exception.class`.

**The "Self-Invocation" Problem:**
If `methodA()` calls `methodB()` within the *same class*, and `methodB` is `@Transactional`, the transaction logic for B is **ignored**.
**Why?** Because `this.methodB()` calls the internal method directly, bypassing the Proxy.

---

## <a name="boot-internals"></a>2. Spring Boot Internals

### Q4: How does Spring Boot Auto-Configuration work? (`@EnableAutoConfiguration`)
**The Magic:**
Spring Boot looks at your classpath (libraries you added) and automatically configures beans.

**Process:**
1.  **`@SpringBootApplication`** includes `@EnableAutoConfiguration`.
2.  This imports `AutoConfigurationImportSelector`.
3.  It scans `META-INF/spring.factories` (in standard jars) or `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` (in newer versions).
4.  It loads a huge list of "AutoConfiguration" classes (e.g., `DataSourceAutoConfiguration`, `JacksonAutoConfiguration`).
5.  **Conditional Filtering:** It checks conditions for each class using `@Conditional`:
    *   `@ConditionalOnClass(DataSource.class)`: Do we have H2/MySQL driver on classpath?
    *   `@ConditionalOnMissingBean`: Did the user define their own `DataSource`? If yes, skip auto-config.
    *   `@ConditionalOnProperty`: Is this feature enabled in `application.properties`?

**Example:**
If `h2.jar` is on classpath -> `DataSourceAutoConfiguration` creates an in-memory `DataSource`.
If you manually define a `@Bean DataSource`, the auto-config backs off (`@ConditionalOnMissingBean`).

---

### Q5: What is the difference between `@Controller` and `@RestController`?
**Internals:**
*   **`@Controller`:** Used for traditional MVC.
    *   The method usually returns a `String` (view name, e.g., "index").
    *   A `ViewResolver` converts "index" -> `index.jsp` or `index.html`.
*   **`@RestController`:**
    *   It is a composite annotation: `@Controller` + `@ResponseBody`.
    *   `@ResponseBody` tells Spring: "Do not use ViewResolver. Write the return value directly to the HTTP Response body."
    *   **Message Conversion:** Spring uses `HttpMessageConverter` (usually Jackson) to serialize the returned object (e.g., `User`) into JSON.

---

## <a name="data-access"></a>3. Data Access (JPA)

### Q6: What is the N+1 Problem in Hibernate/JPA? How to fix it?
**Scenario:**
You have `User` (1) ----> `Orders` (Many).
You fetch 10 users: `List<User> users = repo.findAll();` (1 Query).
Then you loop and print orders: `user.getOrders().size()`.

**The Problem:**
Since `orders` is lazy-loaded by default, Hibernate executes a *separate query* for each user to fetch their orders.
*   1 Query (fetch users) + N Queries (fetch orders for N users).
*   Total N+1 queries. For 1000 users, that's 1001 DB calls. Disaster for performance.

**Solutions:**
1.  **Join Fetch (JPQL):**
    `@Query("SELECT u FROM User u JOIN FETCH u.orders")`
    This forces a SQL JOIN, fetching users and orders in a **single query**.
2.  **EntityGraph:** Define which attributes to load eagerly in the repository method.
3.  **Batch Size:** `@BatchSize(size=10)`. Hibernate will fetch orders for 10 users at a time using `IN` clause.

---

### Q7: Explain Hibernate First-Level vs Second-Level Cache.
**L1 Cache (Session Scope):**
*   Enabled by default. Mandatory.
*   Associated with the `EntityManager` (Session).
*   If you request `findById(1)` twice in the *same transaction*, Hibernate returns the object from L1 cache (no 2nd DB call).
*   Cleared when transaction ends.

**L2 Cache (SessionFactory Scope):**
*   Disabled by default. Shared across all sessions/transactions.
*   Uses external providers (EhCache, Redis, Hazelcast).
*   Useful for data that is read frequently but changes rarely (e.g., Reference Data, Countries, States).
*   **Risks:** Stale data if external systems modify the DB directly.

---

## <a name="security"></a>4. Spring Security Architecture

### Q8: How does Spring Security work internally (Filter Chain)?
**Architecture:**
Spring Security is essentially a chain of standard Servlet Filters. It wraps the standard `HttpServletRequest`.

**DelegatingFilterProxy:**
The web container (Tomcat) doesn't know about Spring Beans. It knows standard filters.
Spring provides `DelegatingFilterProxy`, a standard filter that delegates all work to a Spring Bean (`FilterChainProxy`).

**The Chain:**
1.  **`SecurityContextPersistenceFilter`:** Loads `SecurityContext` (User details) from Session (or empty for stateless).
2.  **`LogoutFilter`:** Checks if URL is /logout.
3.  **`UsernamePasswordAuthenticationFilter`:** Checks if URL is /login. Extracts username/password. Attempts Authentication.
4.  **`BearerTokenAuthenticationFilter` (Resource Server):** Checks for "Bearer" token.
5.  **`ExceptionTranslationFilter`:** Catches security exceptions (AccessDenied) and returns 401/403.
6.  **`FilterSecurityInterceptor`:** Final gatekeeper. Checks `@PreAuthorize` or URL configs (`.requestMatchers("/admin").hasRole("ADMIN")`).

**Authentication Manager:**
The filters don't authenticate. They delegate to `AuthenticationManager`, which delegates to `AuthenticationProvider` (e.g., DaoAuthenticationProvider calls DB, LdapAuthenticationProvider calls LDAP).


---

## <a name="spring-mvc"></a>3. Spring MVC & REST APIs (Deep Dive)

### Q9: Explain the internal workflow of `DispatcherServlet`.
**Architecture:**
The `DispatcherServlet` is the "Front Controller" of Spring MVC. It receives all incoming HTTP requests and delegates them.

**Workflow Steps:**
1.  **Request:** HTTP Request lands on `DispatcherServlet`.
2.  **Handler Mapping:** Servlet asks `HandlerMapping` (e.g., `RequestMappingHandlerMapping`) to find which Controller method maps to this URL (`/api/users`). It returns a `HandlerExecutionChain` (Interceptors + Handler).
3.  **Handler Adapter:** Servlet asks `HandlerAdapter` to execute the method. (Needed because Controllers can be of different types).
4.  **Execution:** Adapter calls your Controller method (`getUser()`).
5.  **Return Value:**
    *   **MVC:** Controller returns a View Name ("profile").
    *   **REST:** Controller returns a Data Object (`User`).
6.  **Response Handling:**
    *   **MVC:** `ViewResolver` resolves "profile" -> `profile.jsp`. View is rendered.
    *   **REST:** `HandlerAdapter` uses `HttpMessageConverter` (Jackson) to write JSON to response stream.
7.  **Response:** HTTP Response sent back to client.

---

### Q10: How does Spring Security authentication work? (`AuthenticationManager`)
**Deep Dive:**
The central interface is `AuthenticationManager`. The default implementation is `ProviderManager`.

**Workflow:**
1.  **Filter:** `UsernamePasswordAuthenticationFilter` creates an `Authentication` object (Unauthenticated token) containing username/password.
2.  **Manager:** Passes token to `ProviderManager`.
3.  **Providers:** `ProviderManager` iterates through a list of `AuthenticationProvider`s.
    *   `DaoAuthenticationProvider`: Checks DB via `UserDetailsService`.
    *   `LdapAuthenticationProvider`: Checks LDAP.
4.  **Verification:** The Provider checks password (using `PasswordEncoder`).
5.  **Success:** Returns a fully populated `Authentication` object (Authenticated + Authorities/Roles).
6.  **Context:** The Filter stores this object in `SecurityContextHolder` (ThreadLocal).

**Interview Tip:** "How do I implement custom auth?" -> Implement `AuthenticationProvider` and add it to the manager.

---

### Q11: What is the difference between `@Component`, `@Repository`, `@Service`, and `@Controller`?
**Internals:**
All of them are meta-annotated with `@Component`. Spring treats them almost identically (creates beans).
**Differences:**
1.  **`@Component`:** Generic.
2.  **`@Controller`:** Registers handler methods in `RequestMappingHandlerMapping`.
3.  **`@Service`:** Semantic only (indicates business logic). No special processing yet.
4.  **`@Repository`:**
    *   **Exception Translation:** It enables `PersistenceExceptionTranslationPostProcessor`.
    *   It catches platform-specific exceptions (Hibernate `ConstraintViolationException`, JDBC `SQLException`) and re-throws them as Spring's unified **`DataAccessException`** hierarchy (Unchecked).

---

### Q12: How does Spring Boot handle configuration properties? (`@ConfigurationProperties`)
**Type-Safe Configuration:**
Instead of injecting `@Value("${app.host}")` 50 times, we use a POJO.

```java
@ConfigurationProperties(prefix = "app")
@Component
public class AppConfig {
    private String host;
    private int port;
    private List<String> servers;
    // getters/setters
}
```
**Benefits:**
*   **Validation:** Can use JSR-303 annotations (`@NotNull`, `@Min`).
*   **IDE Support:** Auto-completion in `application.yml`.
*   **Lists/Maps:** Easy binding of complex structures.

---

### Q13: What are Spring Boot Actuator Endpoints? How to secure them?
**Purpose:** Production monitoring.
*   `/health`: Up/Down status. Checks DB, Disk, Redis.
*   `/metrics`: CPU, RAM, GC stats, HTTP counters.
*   `/loggers`: View and **change** log levels at runtime (POST request).
*   `/threaddump`: Real-time thread dump.

**Security Risk:**
Exposing `/env` or `/heapdump` is dangerous (can reveal passwords).
**Securing:**
1.  Disable sensitive endpoints by default.
2.  Expose only needed: `management.endpoints.web.exposure.include=health,info`.
3.  Run on separate port: `management.server.port=8081` (Internal firewall).
4.  Apply Spring Security: `.requestMatchers("/actuator/**").hasRole("ADMIN")`.

---

### Q14: Explain Spring Bean Scopes.
**1. Singleton (Default):**
*   One instance per container.
*   **Thread Safety:** The bean must be stateless (or synchronize access) because multiple threads access it simultaneously.

**2. Prototype:**
*   New instance every time requested.
*   **Use Case:** Stateful beans (rare).

**3. Web Scopes (Only in WebContext):**
*   **Request:** One per HTTP Request.
*   **Session:** One per HTTP Session.
*   **Application:** One per ServletContext.
*   **WebSocket:** One per WebSocket session.

**Proxying Scoped Beans:**
If you inject a `Session` bean into a `Singleton` bean, Spring injects a **Proxy**.
Why? The Singleton is created once. The Session bean changes per user. The Proxy checks the current thread/request and delegates to the correct instance.

---

### Q15: What is Spring AOP? How is it different from AspectJ?
**Spring AOP:**
*   **Runtime Weaving:** Uses Dynamic Proxies (JDK Interface or CGLIB Class).
*   **Limitations:** Can only advise public methods of Spring Beans. Cannot advise self-invocations.
*   **Simplicity:** No special compiler needed.

**AspectJ:**
*   **Compile-Time Weaving:** Modifies bytecode during compilation (or load-time).
*   **Power:** Can advise private methods, constructors, static methods, final classes.
*   **Performance:** Faster runtime (no proxy overhead), slower build time.

---

### Q16: How to handle Exceptions globally in Spring Boot?
**Evolution:**
1.  **Legacy:** `SimpleMappingExceptionResolver` (XML).
2.  **Annotation:** `@ExceptionHandler` in Controller (Local).
3.  **Global (Best Practice):** `@ControllerAdvice` + `@ExceptionHandler`.

**Implementation:**
```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(UserNotFoundException.class)
    public ProblemDetail handleNotFound(Exception e) {
        return ProblemDetail.forStatusAndDetail(HttpStatus.NOT_FOUND, e.getMessage());
    }
}
```
**`ProblemDetail` (Spring 6):** Standard RFC 7807 JSON format for errors.

---

### Q17: What is Spring Profiles?
**Environment Isolation:**
Allows registering beans or properties conditionally.
*   `@Profile("dev")`: Bean only created if active profile is dev.
*   `application-dev.yml`: Properties loaded only for dev.

**Activation:**
*   `-Dspring.profiles.active=prod`
*   `export SPRING_PROFILES_ACTIVE=prod`

---

### Q18: Difference between `Filter` and `Interceptor`?
**Filter (Servlet Standard):**
*   Runs **before** DispatcherServlet.
*   Access to raw `ServletRequest` / `ServletResponse`.
*   Good for: Security, Compression, Encoding, logging raw bytes.

**HandlerInterceptor (Spring Specific):**
*   Runs **inside** DispatcherServlet.
*   Access to the `Handler` (Controller method) being called.
*   Good for: AuthZ checks based on handler annotations, adding model attributes, measuring execution time.


---

## <a name="security-advanced"></a>Spring Security (Advanced)

### Q19: Explain the difference between `@PreAuthorize` and `@Secured`.
**Deep Dive:**
1.  **`@Secured`:** Legacy (Java 5). Supports only simple roles (`ROLE_ADMIN`). AND/OR logic is hard.
2.  **`@PreAuthorize`:** Uses **SpEL** (Spring Expression Language).
    *   Powerful expressions: `@PreAuthorize("hasRole('ADMIN') or #user.name == authentication.name")`.
    *   Can access method arguments (`#user`) and the current principal.
    *   **Recommendation:** Always use `@PreAuthorize`.

### Q20: How does `MethodSecurity` work? (`@EnableMethodSecurity`)
**AOP Proxy:**
When you enable method security, Spring creates an AOP proxy around your Service/Controller beans.
Before the method is executed, the proxy delegates to `MethodSecurityInterceptor`, which calls `AccessDecisionManager`. If access is denied, it throws `AccessDeniedException`.

---

## <a name="boot-testing"></a>Testing in Spring Boot

### Q21: What is the difference between `@MockBean` and `@Mock`?
**Internals:**
1.  **`@Mock` (Mockito):** Creates a mock object. It is NOT part of the Spring Context. Used in simple unit tests (no Spring).
2.  **`@MockBean` (Spring Boot):**
    *   It creates a Mockito mock.
    *   **Crucial:** It *replaces* the existing bean of the same type in the Spring ApplicationContext.
    *   If the bean doesn't exist, it adds it.
    *   **Use Case:** Integration tests (`@SpringBootTest`) where you want to load the full context but mock out *just* the external payment gateway.

### Q22: How to test a Controller (`@WebMvcTest`)?
**Slice Test:**
`@WebMvcTest(UserController.class)`
1.  Loads only the web layer (Controller, ControllerAdvice, Filters).
2.  **Does NOT** load Service/Repository beans.
3.  You must `@MockBean` the dependencies (e.g., `UserService`).
4.  Use `MockMvc` to perform requests:
    ```java
    mockMvc.perform(get("/users/1"))
           .andExpect(status().isOk())
           .andExpect(jsonPath("$.name").value("John"));
    ```

---

## <a name="data-advanced"></a>Data Access (Advanced)

### Q23: What is Optimistic Locking? How to implement it in JPA?
**Concept:**
Instead of locking the DB row (Pessimistic), we assume no conflict will occur.
**Implementation:**
1.  Add a version field: `@Version private Long version;`.
2.  **Read:** Entity loaded with version `1`.
3.  **Write:** Update SQL includes version check: `UPDATE table SET name='X', version=2 WHERE id=1 AND version=1`.
4.  **Conflict:** If another thread already updated version to `2`, the row count is 0. Hibernate throws `OptimisticLockException`.

### Q24: What is the purpose of `EntityManager`?
**Role:**
It is the primary interface for interacting with the Persistence Context.
*   `persist()`: Make transient instance managed.
*   `merge()`: Update detached instance.
*   `remove()`: Delete.
*   `find()`: Get by ID.
`JpaRepository` is just a wrapper around `EntityManager`.

### Q25: Difference between `save()` and `saveAndFlush()`?
**Internals:**
*   **`save()`:** Temporarily stores changes in the Persistence Context (First-level cache). The actual SQL `INSERT/UPDATE` is deferred until `commit` or needed by a query.
*   **`saveAndFlush()`:** Calls `save()` and immediately triggers `entityManager.flush()`, forcing the SQL to be sent to the DB. Useful if you need to catch DB constraints (like Unique violation) inside a try-catch block immediately.

---

## <a name="misc-boot"></a>Miscellaneous Spring Boot

### Q26: How to create a Custom Starter?
**Steps:**
1.  **AutoConfiguration Module:** Create a maven project.
2.  **Code:** Write the logic (e.g., `MyService`).
3.  **Configuration:** Create `MyServiceAutoConfiguration` class.
    *   Annotate with `@Configuration`.
    *   Use `@Bean` to create `MyService`.
    *   Use `@ConditionalOnClass` / `@ConditionalOnProperty` to control when it activates.
4.  **Registration:** Add file `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` listing your config class.
5.  **Use:** Other apps just include your jar dependency.

### Q27: What is Spring Shell?
**Concept:**
Framework to build CLI (Command Line Interface) applications.
Allows you to define commands:
```java
@ShellMethod("Add two numbers")
public int add(int a, int b) { return a + b; }
```
You can run the jar and get an interactive prompt: `shell:> add 1 2`.

### Q28: How does Spring handle Circular Dependencies?
**Scenario:** A needs B, B needs A.
**Constructor Injection:** Fails immediately (`BeanCurrentlyInCreationException`). Impossible to resolve.
**Setter/Field Injection:**
1.  Spring creates A (raw object).
2.  Spring creates B.
3.  Injects A (partially created) into B.
4.  Injects B into A.
**Resolution:** It works because of the 3-level cache in Spring Singleton registry.
**Best Practice:** Avoid circular deps. Use `@Lazy` on one side if unavoidable.

### Q29: What is `CommandLineRunner` vs `ApplicationRunner`?
**Purpose:** Run code *once* after the application starts.
**Difference:**
*   **`CommandLineRunner`:** Receives raw `String... args`. (Hard to parse).
*   **`ApplicationRunner`:** Receives `ApplicationArguments` object (parsed flags: `--option=value` becomes `getOptionValues("option")`).

### Q30: How to exclude an Auto-Configuration class?
**Why?** Sometimes Spring auto-configures something you don't want (e.g., `DataSourceAutoConfiguration` when you don't want a DB).
**Ways:**
1.  `@SpringBootApplication(exclude = DataSourceAutoConfiguration.class)`
2.  `spring.autoconfigure.exclude=org.spring...DataSourceAutoConfiguration`


---

## <a name="configuration-advanced"></a>Advanced Configuration (Q31-60)

### Q31: How does Spring Boot Logging work internally? (SLF4J & Logback)
**Architecture:**
*   **Interface:** `SLF4J` (Simple Logging Facade for Java).
*   **Implementation:** `Logback` (Default in Spring Boot).
*   **Bridge:** Even if you use `log4j` or `java.util.logging` classes, Spring Boot bridges them to SLF4J -> Logback.

**Customization:**
1.  **Properties:** `logging.level.com.myapp=DEBUG`.
2.  **XML:** `logback-spring.xml` (Recommended for complex setups).
    *   Allows environment-specific logs: `<springProfile name="dev">`.
    *   Allows complex Appenders (RollingFileAppender, JSON Layout).

### Q32: What is Mapped Diagnostic Context (MDC)?
**Tracing:**
MDC is a ThreadLocal map provided by SLF4J.
**Usage:**
In a Filter/Interceptor, you can put the `requestId` or `userId` into MDC.
`MDC.put("requestId", uuid);`
**Benefit:**
Every log line printed by that thread (even deep inside 10 service layers) will automatically include `[requestId=...]`.
**Cleanup:** Must call `MDC.clear()` in `finally` block to prevent data leaking to reused threads.

### Q33: How to reload configuration at runtime without restart?
**Mechanism:**
1.  **Bean Scope:** `@RefreshScope`.
2.  **Trigger:** `/actuator/refresh` endpoint (POST).
3.  **Process:**
    *   Spring destroys the existing bean.
    *   Reloads properties from the source (e.g., Config Server).
    *   Re-creates the bean on the next request.
**Use Case:** Changing Log Levels, Feature Flags.

### Q34: What is the loading order of Configuration Files?
**Hierarchy (Highest Priority First):**
1.  DevTools global settings (`~/.spring-boot-devtools.properties`).
2.  `@TestPropertySource` on tests.
3.  Command Line Arguments (`--server.port=9090`).
4.  `SPRING_APPLICATION_JSON`.
5.  OS Environment Variables.
6.  `application-{profile}.properties` (External).
7.  `application.properties` (External).
8.  `application-{profile}.properties` (Internal/Classpath).
9.  `application.properties` (Internal/Classpath).

### Q35: What is `SpringApplication.run()` internally?
**Workflow:**
1.  Create `ApplicationContext` (AnnotationConfigServletWebServerApplicationContext).
2.  Register a `ShutdownHook`.
3.  Trigger `ApplicationStartedEvent`.
4.  Perform Auto-Configuration.
5.  Start Embedded Web Server (Tomcat).
6.  Call `CommandLineRunner` beans.
7.  Trigger `ApplicationReadyEvent`.

---

## <a name="actuator-deep"></a>Actuator Deep Dive (Q61-90)

### Q61: How to create a Custom Actuator Endpoint?
**Annotation:** `@Endpoint(id = "my-endpoint")`.
**Methods:**
*   `@ReadOperation`: Maps to HTTP GET.
*   `@WriteOperation`: Maps to HTTP POST.
*   `@DeleteOperation`: Maps to HTTP DELETE.
**Example:**
```java
@Component
@Endpoint(id = "features")
public class FeatureEndpoint {
    @ReadOperation
    public Map<String, Boolean> features() { ... }
}
```

### Q62: Explain Prometheus integration with Spring Boot.
**Architecture:**
Prometheus uses a **Pull Model**.
1.  Add `micrometer-registry-prometheus`.
2.  Spring Boot exposes `/actuator/prometheus`.
3.  This endpoint outputs metrics in a text format Prometheus understands.
4.  Prometheus Server scrapes this URL every X seconds.

### Q63: What is `Micrometer`?
**Facade:**
It is "SLF4J for Metrics".
It provides a vendor-neutral API to collect metrics (Timers, Gauges, Counters).
You code against Micrometer API.
At runtime, you plug in a registry (Datadog, New Relic, Prometheus) to export the data.

### Q64: How to secure Actuator endpoints?
**Best Practice:**
1.  **Network Isolation:** Run on a different port.
    `management.server.port=8081`.
    Block port 8081 at the firewall/Load Balancer for public access.
2.  **Spring Security:**
    ```java
    http.requestMatchers(EndpointRequest.toAnyEndpoint())
        .hasRole("ADMIN");
    ```

---

## <a name="reactive-stack"></a>Reactive Stack (WebFlux) (Q91-120)

### Q91: What is the difference between `Tomcat` and `Netty`?
**Architecture:**
*   **Tomcat:** Servlet Container. Blocking I/O (Thread-per-Request).
    *   If you have 200 threads, you can handle 200 concurrent DB requests. 201st waits.
*   **Netty:** Event Loop. Non-Blocking I/O (NIO).
    *   Uses a small number of threads (CPU Cores * 2).
    *   Can handle 10,000+ concurrent connections because threads don't wait for DB. They register a callback and move to the next request.

### Q92: What is `Mono` vs `Flux`?
**Publisher Types (Project Reactor):**
*   **`Mono<T>`:** 0 or 1 item.
    *   Use for: `findById`, `save` (returns saved entity).
*   **`Flux<T>`:** 0 to N items.
    *   Use for: `findAll`, Data Streaming.
**Operators:** `map`, `flatMap`, `filter`, `zip` (combine), `merge`.

### Q93: Explain Backpressure in Reactive Streams.
**Problem:** Fast Producer -> Slow Consumer. Consumer gets overwhelmed (OOM).
**Mechanism:**
The Subscriber sends a **subscription.request(n)** signal to Publisher.
"I can handle 10 items". Publisher sends 10.
**Strategies:**
1.  **Buffer:** Store extra items in memory (Risk: OOM).
2.  **Drop:** Discard extra items.
3.  **Error:** Throw exception if buffer full.
4.  **Latest:** Keep only the most recent item.

### Q94: Difference between `.map()` and `.flatMap()` in Reactor?
*   **`map`:** Synchronous transformation. `String -> Integer`.
*   **`flatMap`:** Asynchronous transformation. `String -> Mono<Integer>`.
    *   Crucial for non-blocking DB calls. If you use `map` for a DB call, you get `Mono<Mono<User>>`. `flatMap` unwraps it to `Mono<User>`.

### Q95: Can I use JDBC with WebFlux?
**No.** JDBC is a blocking API. If you use it in WebFlux, you block the Event Loop threads, killing the entire application's scalability.
**Solution:** Use **R2DBC** (Reactive Relational Database Connectivity). It is the non-blocking alternative to JDBC.

---

## <a name="security-internals"></a>Advanced Security (Q121-140)

### Q121: How does OAuth2 "Authorization Code Flow" work?
**Steps:**
1.  **User** clicks "Login with Google".
2.  **Redirect:** App redirects user to Google Auth Server.
3.  **Consent:** User logs in and grants permission.
4.  **Code:** Google redirects back to App (`/login/oauth2/code/google`) with a temporary `code`.
5.  **Exchange:** App (Backend) sends `code` + `client_secret` to Google (Direct channel).
6.  **Token:** Google responds with `access_token` and `id_token`.
**Why Code?** To ensure the `access_token` is never exposed in the Browser URL.

### Q122: What is `FilterChainProxy`?
**Internals:**
It is the heart of Spring Security.
It is a special Filter that contains a list of **Security Filter Chains**.
Depending on the URL (`/api/**` vs `/login/**`), it decides which Chain to execute.
This allows you to have different security rules for different parts of the app (e.g., Basic Auth for API, Form Login for Admin).

### Q123: Difference between `@AuthenticationPrincipal` and `SecurityContextHolder`?
**Accessing User:**
1.  **`SecurityContextHolder`:** Static access.
    `Authentication auth = SecurityContextHolder.getContext().getAuthentication();`
    Works anywhere.
2.  **`@AuthenticationPrincipal`:** Method Argument Resolver.
    `public void method(@AuthenticationPrincipal User user)`.
    Cleaner, creates no dependency on static context, easier to test.

---

## <a name="deployment-advanced"></a>Deployment & Kubernetes (Q141-150)

### Q141: How to implement Graceful Shutdown in Spring Boot?
**Configuration:**
`server.shutdown=graceful`
**Mechanism:**
1.  Spring stops accepting *new* HTTP requests.
2.  It waits for *active* requests to complete (default 30s timeout).
3.  Then it shuts down beans and context.
**Kubernetes:** Essential for zero-downtime deployments. K8s sends SIGTERM, Spring finishes work, then K8s kills Pod.

### Q142: How to optimize Docker Image for Spring Boot? (Layering)
**Layered Jar:**
Spring Boot jars have dependencies (50MB) and app code (1MB).
Dependencies rarely change. App code changes often.
**Dockerfile Optimization:**
Use `layertools` to extract layers.
```dockerfile
COPY --from=builder application/dependencies/ ./
COPY --from=builder application/spring-boot-loader/ ./
COPY --from=builder application/snapshot-dependencies/ ./
COPY --from=builder application/application/ ./
```
**Benefit:** Docker caches the dependency layers. Rebuilds are super fast.

### Q143: What is Spring Native (GraalVM)?
**Concept:**
Compiles Java application into a **Native Binary** (not bytecode) ahead-of-time (AOT).
**Pros:**
*   Instant Startup (ms).
*   Low Memory footprint.
**Cons:**
*   Slow Build time.
*   Reflection/Proxies require complex configuration (Hints).

