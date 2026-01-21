
# Java/Spring Boot Interview Questions and Answers

## Page 1

### 1. What is Spring Framework?
The Spring Framework is a comprehensive programming and configuration model for modern Java-based enterprise applications. It provides a lightweight container that manages the lifecycle of objects (beans) and their dependencies. Spring's core features can be used by any Java application, but there are extensions for building web applications on top of the Java EE platform.

### 2. What problems does Spring solve?
Spring solves several problems in Java development:
- **Reduces boilerplate code:** Simplifies database transactions, remote procedure calls, and JMS.
- **Promotes loose coupling:** Through Dependency Injection (DI) and Inversion of Control (IoC), it makes code easier to test and maintain.
- **Simplifies integration:** Provides seamless integration with various Java EE technologies and other popular frameworks.
- **Declarative programming:** Enables developers to focus on business logic by handling cross-cutting concerns like security, transactions, and logging declaratively.

### 3. What is Dependency Injection (DI)?
Dependency Injection is a design pattern in which an object receives other objects that it depends on. These dependencies are "injected" into the object by an external entity (the Spring IoC container) rather than being created by the object itself.

### 4. What are Spring Beans?
Spring Beans are the objects that form the backbone of a Spring application. They are instantiated, assembled, and managed by the Spring IoC container. Beans are created from a configuration metadata that is supplied to the container (e.g., in the form of XML definitions or Java annotations).

### 5. Difference between @Component, @Service, @Repository?
- **@Component:** A generic stereotype for any Spring-managed component. It is the base annotation for other stereotypes.
- **@Service:** A specialization of the component annotation. It is used to mark a class as a service provider in the business layer.
- **@Repository:** A specialization of the component annotation. It is used to mark a class as a data access object (DAO) in the persistence layer. It also enables the translation of persistence-related exceptions into Spring's unified DataAccessException hierarchy.

### 6. What is @Autowired?
The `@Autowired` annotation is used for automatic dependency injection. It allows Spring to resolve and inject collaborating beans into a bean. It can be used on constructors, fields, and setter methods.

### 7. What is Spring Boot? How is it different from Spring?
Spring Boot is an extension of the Spring Framework that simplifies the development of stand-alone, production-grade Spring-based applications. It takes an opinionated view of the Spring platform and third-party libraries, allowing developers to get started with minimum fuss.

**Differences:**
- **Auto-configuration:** Spring Boot automatically configures the Spring application based on the JAR dependencies on the classpath.
- **Embedded servers:** Spring Boot includes embedded servers (like Tomcat, Jetty, or Undertow) to run applications without needing to deploy them as WAR files.
- **Starter dependencies:** Spring Boot provides "starter" dependencies to simplify build configuration.

### 8. What is application.properties/application.yml used for?
These files are used to configure a Spring Boot application. They can be used to specify properties like server port, database connection details, logging levels, and other application settings. `application.yml` uses YAML syntax, which is more hierarchical and readable, while `application.properties` uses a simple key-value format.

### 9. What is an embedded server in Spring Boot?
An embedded server is a server that is packaged as part of the application. Spring Boot includes an embedded Tomcat server by default, but it can be replaced with Jetty or Undertow. This allows the application to be run as a standalone JAR file, without needing to be deployed to an external web server.

### 10. How do you create a REST API in Spring Boot?
To create a REST API in Spring Boot, you need to:
1.  Use the `@RestController` annotation on a class to mark it as a request handler.
2.  Use annotations like `@RequestMapping`, `@GetMapping`, `@PostMapping`, etc., to map HTTP requests to specific methods.
3.  The methods can return objects, and Spring Boot will automatically serialize them to JSON or XML.

### 11. What is @RestController and how is it different from @Controller?
- **@Controller:** A stereotype annotation that marks a class as a Spring MVC controller. It is typically used for traditional web applications that return views (e.g., HTML pages).
- **@RestController:** A convenience annotation that combines `@Controller` and `@ResponseBody`. It is used for creating RESTful web services that return JSON or XML data. The `@ResponseBody` annotation tells Spring to serialize the return value of a method into the response body.

### 12. What are Spring Boot starters?
Spring Boot starters are a set of convenient dependency descriptors that you can include in your application. They provide a one-stop-shop for all the Spring and related technologies that you need, without having to hunt down and configure all the dependencies yourself. For example, `spring-boot-starter-web` includes everything needed to build a web application with Spring MVC.

### 13. What is Spring Boot auto-configuration?
Spring Boot auto-configuration is a feature that automatically configures a Spring application based on the JAR dependencies that are on the classpath. For example, if `spring-boot-starter-web` is on the classpath, Spring Boot will automatically configure a DispatcherServlet, a default error page, and other web-related components.

### 14. What are Spring Boot profiles?
Spring Boot profiles are a way to segregate parts of the application configuration and make it available only in certain environments. For example, you can have a `dev` profile for development, a `test` profile for testing, and a `prod` profile for production. You can use different properties files for each profile (e.g., `application-dev.properties`, `application-prod.properties`).

### 15. How do you configure different environments in Spring Boot?
You can configure different environments in Spring Boot by using profiles. You can create different `application-{profile}.properties` or `application-{profile}.yml` files for each environment. The active profile can be set using the `spring.profiles.active` property in `application.properties` or as a command-line argument.

### 16. What is Spring ApplicationContext?
The `ApplicationContext` is the central interface in a Spring application for providing configuration information to the application. It is a more advanced version of the `BeanFactory` and provides additional features like internationalization, event propagation, and application-layer specific contexts.

### 17. Difference between BeanFactory and ApplicationContext?
- **BeanFactory:** The root interface for accessing a Spring bean container. It provides the basic functionality of managing beans.
- **ApplicationContext:** A sub-interface of `BeanFactory`. It adds more enterprise-specific functionality, such as the ability to resolve textual messages from a properties file and the ability to publish application events to interested event listeners. It also eagerly instantiates singleton beans by default.

### 18. What is component scanning in Spring?
Component scanning is a feature of Spring that allows it to automatically discover and register beans from the classpath. Spring can be configured to scan a package for classes that are annotated with stereotypes like `@Component`, `@Service`, `@Repository`, and `@Controller`.

### 19. What is the purpose of @Configuration annotation?
The `@Configuration` annotation is used to mark a class as a source of bean definitions. It is a central part of Java-based configuration in Spring. Methods within a `@Configuration` class can be annotated with `@Bean` to create and configure beans.

### 20. How to define beans using Java config?
To define beans using Java config, you need to:
1.  Create a class and annotate it with `@Configuration`.
2.  Within the class, create methods and annotate them with `@Bean`.
3.  The return value of the `@Bean` method will be registered as a bean in the Spring application context.

### 21. How to define beans using XML config?
To define beans using XML config, you create an XML file with a `<beans>` root element. Each bean is defined using a `<bean>` element, with attributes like `id` and `class`. Dependencies can be injected using the `<property>` or `<constructor-arg>` elements.

### 22. What is @PostConstruct and @PreDestroy?
- **@PostConstruct:** This annotation is used on a method that needs to be executed after dependency injection is done to perform any initialization.
- **@PreDestroy:** This annotation is used on a method that is called when the bean is about to be removed from the container. It is used to perform any cleanup tasks.

### 23. What is the purpose of @Scope annotation?
The `@Scope` annotation is used to define the scope of a bean. The scope determines the lifecycle of the bean and how it is shared within the application. The default scope is `singleton`.

### 24. Difference between singleton and prototype bean scope?
- **singleton:** (Default) Only one instance of the bean is created for the entire application context.
- **prototype:** A new instance of the bean is created every time it is requested.

### 25. What is @Value annotation?
The `@Value` annotation is used to inject values into fields in Spring-managed beans. It can be used to inject values from properties files, system properties, or even the results of SpEL (Spring Expression Language) expressions.

### 26. What are Spring events?
Spring events are part of the ApplicationContext and allow for a loosely coupled way of communication between different parts of an application. An application can create custom events that extend the `ApplicationEvent` class and publish them using an `ApplicationEventPublisher`. Other components can listen for these events by implementing the `ApplicationListener` interface or by using the `@EventListener` annotation.

### 27. What is ApplicationEventPublisher?
The `ApplicationEventPublisher` is an interface that allows an application to publish events. When an event is published, all registered `ApplicationListener`s for that event type are notified.

### 28. Difference between dependency injection via constructor vs setter?
- **Constructor Injection:** Dependencies are provided as arguments to the constructor. This is the recommended approach for mandatory dependencies, as it ensures that the object is in a valid state upon creation.
- **Setter Injection:** Dependencies are provided through setter methods. This is suitable for optional dependencies that can be set after the object has been created.

### 29. How to handle exceptions in Spring MVC?
Spring MVC provides several ways to handle exceptions:
- **@ExceptionHandler:** An annotation that can be used on methods within a controller to handle exceptions thrown by request handler methods in the same controller.
- **@ControllerAdvice:** A specialization of the `@Component` annotation that allows you to handle exceptions across the whole application in one global component.
- **HandlerExceptionResolver:** An interface that can be implemented to create a global exception handler.

### 30. What is @ControllerAdvice?
`@ControllerAdvice` is an annotation that is used to define a class that can handle exceptions globally across all controllers. It can also be used to add global `@ModelAttribute` and `@InitBinder` methods.

### 31. What is @ExceptionHandler?
`@ExceptionHandler` is an annotation that is used to handle specific exceptions in a controller. It can be used on methods within a controller or a `@ControllerAdvice` class.

### 32. What is @RestControllerAdvice?
`@RestControllerAdvice` is a convenience annotation that combines `@ControllerAdvice` and `@ResponseBody`. It is used for creating global exception handlers for RESTful web services that return JSON or XML data.

### 33. How do you validate request parameters in Spring Boot?
Spring Boot uses the Bean Validation API (JSR 303) for validating request parameters. You can add validation annotations (like `@NotNull`, `@Size`, etc.) to the fields of a request body object and then use the `@Valid` annotation on the method parameter to trigger the validation.

### 34. What is @Valid and @Validated?
- **@Valid:** A standard JSR-303 annotation that is used to enable validation on a method parameter.
- **@Validated:** A Spring-specific annotation that is a specialization of `@Valid`. It allows for the use of validation groups.

### 35. What is Bean Validation (JSR 303)?
Bean Validation is a Java specification that defines a metadata model and API for entity and method validation. It provides a set of standard validation annotations (e.g., `@NotNull`, `@Size`, `@Min`, `@Max`) and allows for the creation of custom validators.

### 36. How do you customize validation messages in Spring Boot?
You can customize validation messages in Spring Boot by creating a `ValidationMessages.properties` file in the `src/main/resources` directory. The keys in this file should correspond to the validation annotation and the field name.

### 37. What is @PathVariable?
The `@PathVariable` annotation is used to bind a method parameter to a URI template variable. For example, if the URI is `/users/{id}`, you can use `@PathVariable("id")` to bind the value of `id` to a method parameter.

### 38. What is @RequestParam?
The `@RequestParam` annotation is used to bind a method parameter to a request parameter. For example, if the URL is `/users?name=John`, you can use `@RequestParam("name")` to bind the value of `name` to a method parameter.

### 39. Difference between @RequestBody and @ModelAttribute?
- **@RequestBody:** This annotation is used to bind the body of an HTTP request to a method parameter. Spring will deserialize the request body (e.g., from JSON) into an object.
- **@ModelAttribute:** This annotation is used to bind a method parameter or a method return value to a named model attribute and then exposes it to a web view.

### 40. What is @RequestMapping and its HTTP method shortcuts?
`@RequestMapping` is a generic annotation that can be used to map HTTP requests to handler methods. It can be configured with a path, HTTP method, and other attributes.
Spring provides several HTTP method-specific shortcuts for `@RequestMapping`:
- `@GetMapping`
- `@PostMapping`
- `@PutMapping`
- `@DeleteMapping`
- `@PatchMapping`

### 41. What is @GetMapping, @PostMapping, @PutMapping, @DeleteMapping?
These are shortcut annotations for `@RequestMapping` that are used to map HTTP GET, POST, PUT, and DELETE requests, respectively.

### 42. What is ResponseEntity?
`ResponseEntity` represents the entire HTTP response, including the status code, headers, and body. It is a generic type, so you can specify the type of the response body.

### 43. How do you handle CORS in Spring Boot?
CORS (Cross-Origin Resource Sharing) can be handled in Spring Boot in several ways:
- **@CrossOrigin annotation:** This annotation can be used on a controller method or a whole controller to enable CORS for that specific resource.
- **Global CORS configuration:** You can configure CORS globally by defining a `WebMvcConfigurer` bean and overriding the `addCorsMappings` method.

### 44. How to enable Swagger / OpenAPI documentation?
To enable Swagger/OpenAPI documentation in a Spring Boot project, you need to:
1.  Add the `springdoc-openapi-ui` dependency to your `pom.xml`.
2.  The API documentation will then be available at `/v3/api-docs` and the Swagger UI at `/swagger-ui.html`.

### 45. What is Spring Boot actuator?
Spring Boot Actuator is a sub-project of Spring Boot that provides production-ready features for monitoring and managing an application. It includes several built-in endpoints for health checks, metrics, environment information, and more.

### 46. How to monitor application metrics with Actuator?
Actuator provides several endpoints for monitoring application metrics, such as `/metrics` and `/health`. The `/metrics` endpoint provides a detailed breakdown of various metrics, including memory usage, garbage collection, and HTTP request statistics.

### 47. What is /health endpoint in Spring Boot?
The `/health` endpoint is provided by Spring Boot Actuator and is used to check the health of the application. It returns a simple status of "UP" or "DOWN". It can also be configured to show more detailed health information for various components, like the database and disk space.

### 48. How to customize Actuator endpoints?
Actuator endpoints can be customized in several ways:
- **Enabling/disabling endpoints:** You can enable or disable specific endpoints using the `management.endpoints.web.exposure.include` and `management.endpoints.web.exposure.exclude` properties.
- **Securing endpoints:** You can secure the endpoints using Spring Security.
- **Creating custom endpoints:** You can create your own custom endpoints by implementing the `@Endpoint` annotation.

### 49. What is spring-boot-devtools used for?
`spring-boot-devtools` is a set of tools that can make the application development process easier. It provides features like:
- **Automatic restart:** Automatically restarts the application when files on the classpath change.
- **LiveReload:** Automatically reloads the browser when resources change.
- **Property defaults:** Disables caching for template engines to see changes without restarting.

### 50. How to enable hot reload in Spring Boot?
Hot reload is enabled by default when you include the `spring-boot-devtools` dependency in your project. This will automatically restart the application whenever files on the classpath are modified.

## Page 2

### 51. What is Spring Data JPA?
Spring Data JPA is a sub-project of Spring Data that makes it easier to implement JPA-based repositories. It provides a set of interfaces that can be extended to create repositories for your entities. Spring Data JPA will automatically provide implementations for the methods defined in the interfaces.

### 52. How to define entities in JPA?
To define an entity in JPA, you need to:
1.  Create a POJO (Plain Old Java Object) class.
2.  Annotate the class with `@Entity`.
3.  Annotate a field in the class with `@Id` to specify the primary key.

### 53. Difference between @Entity and @Table?
- **@Entity:** This annotation is used to mark a class as a JPA entity. It is a required annotation for any class that is to be persisted to a database.
- **@Table:** This annotation is used to specify the details of the table that will be used to persist the entity in the database. It is an optional annotation, and if it is not specified, the table name will be the same as the entity name.

### 54. What is @Id and @GeneratedValue?
- **@Id:** This annotation is used to specify the primary key of an entity.
- **@GeneratedValue:** This annotation is used to specify how the primary key of an entity is generated. It can be used with different generation strategies, such as `AUTO`, `IDENTITY`, `SEQUENCE`, and `TABLE`.

### 55. Difference between GenerationType.IDENTITY and GenerationType.AUTO?
- **GenerationType.IDENTITY:** This strategy relies on the database to generate the primary key. It is typically used with auto-increment columns in MySQL.
- **GenerationType.AUTO:** (Default) This strategy allows the persistence provider to choose the generation strategy. It is the most portable option.

### 56. What is @Column used for?
The `@Column` annotation is used to specify the details of the column that a field will be mapped to in the database. It can be used to specify the column name, length, and other attributes.

### 57. What is @ManyToOne VS @OneToMany?
- **@ManyToOne:** This annotation is used to define a many-to-one relationship between two entities. For example, many `Employee` entities can be associated with one `Department` entity.
- **@OneToMany:** This annotation is used to define a one-to-many relationship between two entities. For example, one `Department` entity can be associated with many `Employee` entities.

### 58. What is @OneToOne VS @ManyToMany?
- **@OneToOne:** This annotation is used to define a one-to-one relationship between two entities. For example, one `User` entity can be associated with one `Address` entity.
- **@ManyToMany:** This annotation is used to define a many-to-many relationship between two entities. For example, many `Student` entities can be associated with many `Course` entities.

### 59. What is lazy vs eager fetching?
- **Lazy Fetching:** The data is not fetched from the database until it is actually needed. This is the default for `@OneToMany` and `@ManyToMany` relationships.
- **Eager Fetching:** The data is fetched from the database immediately, along with the main entity. This is the default for `@ManyToOne` and `@OneToOne` relationships.

### 60. What is EntityManager?
The `EntityManager` is a JPA interface that is used to interact with the persistence context. It provides methods for creating, reading, updating, and deleting entities.

### 61. What is CrudRepository?
`CrudRepository` is a Spring Data interface that provides basic CRUD (Create, Read, Update, Delete) operations for an entity.

### 62. What is JpaRepository?
`JpaRepository` is a Spring Data interface that extends `PagingAndSortingRepository` (which in turn extends `CrudRepository`). It provides JPA-specific methods, such as flushing the persistence context and deleting records in a batch.

### 63. What is PagingAndSortingRepository?
`PagingAndSortingRepository` is a Spring Data interface that extends `CrudRepository`. It provides methods for retrieving entities in a paginated and sorted manner.

### 64. How to write custom queries with @Query?
The `@Query` annotation can be used on a repository method to define a custom query. The query can be written in JPQL (Java Persistence Query Language) or native SQL.

### 65. What is query derivation in Spring Data JPA?
Query derivation is a feature of Spring Data JPA that allows you to create queries by simply defining a method name in a repository interface. Spring Data will parse the method name and create a query based on the property names and keywords used.

### 66. What is transaction management in Spring?
Transaction management is a mechanism that allows you to ensure the integrity of your data. In Spring, transaction management can be done declaratively using the `@Transactional` annotation or programmatically using the `TransactionTemplate` or `PlatformTransactionManager`.

### 67. Difference between programmatic and declarative transactions?
- **Programmatic Transaction Management:** The transaction boundaries are defined explicitly in the code. This gives you more control, but it is also more verbose.
- **Declarative Transaction Management:** The transaction boundaries are defined using annotations or XML configuration. This is less invasive and is the recommended approach in most cases.

### 68. What is @Transactional annotation?
The `@Transactional` annotation is used to mark a method as transactional. When a method is annotated with `@Transactional`, Spring will create a new transaction before the method is executed and commit the transaction after the method completes. If the method throws an exception, the transaction will be rolled back.

### 69. How to handle propagation in transactions?
Transaction propagation determines how a transactional method behaves when it is called from another transactional method. The propagation level can be configured using the `propagation` attribute of the `@Transactional` annotation. The default propagation level is `REQUIRED`.

### 70. How to handle isolation levels in transactions?
The isolation level of a transaction determines how it is isolated from other concurrent transactions. The isolation level can be configured using the `isolation` attribute of the `@Transactional` annotation. The default isolation level is `DEFAULT`.

### 71. How to rollback transactions manually?
Transactions can be rolled back manually by calling the `setRollbackOnly()` method on the `TransactionAspectSupport` class.

### 72. How to handle exceptions in transactions?
By default, Spring will only roll back a transaction for unchecked exceptions. To roll back a transaction for a checked exception, you need to specify the exception type in the `rollbackFor` attribute of the `@Transactional` annotation.

### 73. What is Spring Boot Security?
Spring Boot Security is a sub-project of Spring Boot that provides security features for an application. It provides authentication, authorization, and protection against common attacks.

### 74. How to secure REST APIs with Spring Security?
REST APIs can be secured with Spring Security by:
1.  Adding the `spring-boot-starter-security` dependency.
2.  Configuring a `SecurityFilterChain` bean to define the security rules.
3.  You can use different authentication mechanisms, such as basic authentication, JWT, or OAuth2.

### 75. Difference between authentication and authorization?
- **Authentication:** The process of verifying the identity of a user.
- **Authorization:** The process of determining whether a user has the permission to access a particular resource.

### 76. What is OAuth2?
OAuth2 is an authorization framework that enables a third-party application to obtain limited access to an HTTP service, either on behalf of a resource owner by orchestrating an approval interaction between the resource owner and the HTTP service, or by allowing the third-party application to obtain access on its own behalf.

### 77. How to implement JWT authentication in Spring Boot?
To implement JWT (JSON Web Token) authentication in Spring Boot, you need to:
1.  Add the necessary dependencies for Spring Security and JWT.
2.  Create a class that generates and validates JWTs.
3.  Create a filter that intercepts incoming requests, extracts the JWT, and validates it.
4.  Configure Spring Security to use the JWT filter.

### 78. How to configure in-memory user authentication?
In-memory user authentication can be configured in Spring Security by creating a `InMemoryUserDetailsManager` bean and adding users with their credentials and roles.

### 79. How to use UserDetailsService?
The `UserDetailsService` interface is used to load user-specific data. It has a single method, `loadUserByUsername()`, that returns a `UserDetails` object. You can provide your own implementation of this interface to load user data from a database or any other source.

### 80. What is BCryptPasswordEncoder?
`BCryptPasswordEncoder` is an implementation of the `PasswordEncoder` interface that uses the BCrypt hashing algorithm to encode passwords. It is the recommended password encoder for most applications.

### 81. How to protect endpoints with roles?
Endpoints can be protected with roles in Spring Security by using the `hasRole()` or `hasAuthority()` methods in the security configuration. You can also use method-level security with annotations like `@PreAuthorize`.

### 82. How to configure HTTP Basic and Form login?
HTTP Basic and Form login can be configured in Spring Security by using the `httpBasic()` and `formLogin()` methods in the security configuration.

### 83. What is CSRF and how to handle it?
CSRF (Cross-Site Request Forgery) is an attack that tricks a user into submitting a malicious request. Spring Security provides built-in protection against CSRF attacks, which is enabled by default.

### 84. How to implement method-level security (@PreAuthorize)?
Method-level security can be implemented in Spring Security by using the `@EnableGlobalMethodSecurity` annotation and then using annotations like `@PreAuthorize`, `@PostAuthorize`, `@Secured`, and `@RolesAllowed` on methods.

### 85. What is Spring Boot logging?
Spring Boot uses Commons Logging for all internal logging but leaves the underlying log implementation open. Default configurations are provided for Java Util Logging, Log4j2, and Logback.

### 86. How to configure logging with Logback?
Logging with Logback can be configured by creating a `logback-spring.xml` file in the `src/main/resources` directory. This file can be used to configure log levels, appenders, and other logging settings.

### 87. How to create custom log patterns?
Custom log patterns can be created by setting the `logging.pattern.console` or `logging.pattern.file` properties in `application.properties`.

### 88. What is Spring Boot Actuator metrics?
Spring Boot Actuator metrics provide detailed information about the performance of an application. The `/metrics` endpoint provides a wide range of metrics, including memory usage, garbage collection, and HTTP request statistics.

### 89. How to monitor database connections in Actuator?
Actuator can be used to monitor database connections by using the `/health` and `/metrics` endpoints. The `/health` endpoint will show the status of the database connection, and the `/metrics` endpoint will show detailed metrics about the connection pool.

### 90. What is Spring Boot CLI?
The Spring Boot CLI (Command Line Interface) is a command-line tool that can be used to quickly create, run, and test Spring Boot applications.

### 91. How to create Spring Boot starter project?
A Spring Boot starter project can be created using the Spring Initializr (https://start.spring.io/), which is a web-based tool that allows you to generate a Spring Boot project with your desired dependencies.

### 92. How to configure YAML vs properties?
YAML and properties files are two different ways to configure a Spring Boot application. YAML is more hierarchical and readable, while properties files use a simple key-value format. You can choose to use either one, or both.

### 93. What are Spring Boot dependencies management?
Spring Boot provides a curated list of dependencies that are known to work well together. This is managed by the `spring-boot-dependencies` BOM (Bill of Materials), which is included in the `spring-boot-starter-parent`.

### 94. What is embedded Tomcat, Jetty, or Undertow?
These are embedded web servers that are included in Spring Boot. Tomcat is the default, but it can be replaced with Jetty or Undertow.

### 95. How to customize server port and context path?
The server port and context path can be customized in `application.properties` by setting the `server.port` and `server.servlet.context-path` properties.

### 96. What is Spring Boot profiles for dev, test, prod?
Spring Boot profiles are used to provide different configurations for different environments. You can have a `dev` profile for development, a `test` profile for testing, and a `prod` profile for production.

### 97. How to configure environment-specific beans?
Environment-specific beans can be configured by using the `@Profile` annotation. This annotation can be used on a `@Bean` method or a `@Configuration` class to indicate that it should only be active for a specific profile.

### 98. How to create a REST API with pagination and sorting?
A REST API with pagination and sorting can be created by using the `PagingAndSortingRepository` interface. This interface provides methods for retrieving entities in a paginated and sorted manner.

### 99. How to implement exception handling globally?
Global exception handling can be implemented in Spring Boot by using the `@ControllerAdvice` annotation. This annotation can be used to define a class that can handle exceptions across the whole application.

### 100. How to deploy Spring Boot as a standalone JAR or WAR?
A Spring Boot application can be deployed as a standalone JAR file or as a WAR file. To deploy as a JAR, you can use the `spring-boot-maven-plugin` to create an executable JAR. To deploy as a WAR, you need to extend the `SpringBootServletInitializer` class and configure the packaging in your build file to be `war`.

## Page 6

### 101. What is microservices architecture?
Microservices architecture is an architectural style that structures an application as a collection of small, autonomous services. Each service is self-contained and implements a single business capability.

### 102. Difference between monolith and microservices?
- **Monolith:** A monolithic application is built as a single, unified unit. All the components of the application are interconnected and interdependent.
- **Microservices:** A microservices application is built as a collection of small, independent services. Each service is responsible for a single business capability and can be developed, deployed, and scaled independently.

### 103. What is service discovery?
Service discovery is the process of automatically detecting devices and services on a network. In a microservices architecture, service discovery is used to find the location of a service.

### 104. What is Eureka Server and Eureka Client?
- **Eureka Server:** A service discovery server that allows microservices to register themselves and discover other services.
- **Eureka Client:** A client that registers with a Eureka Server and discovers other services.

### 105. What is Ribbon?
Ribbon is a client-side load balancer that is used to distribute traffic across a group of microservices.

### 106. What is Feign Client?
Feign is a declarative REST client that makes it easier to call RESTful web services. It integrates with Ribbon and Eureka to provide client-side load balancing.

### 107. What is Hystrix circuit breaker?
Hystrix is a latency and fault tolerance library that is used to isolate points of access to remote systems, services, and 3rd party libraries, stop cascading failure, and enable resilience in complex distributed systems where failure is inevitable.

### 108. How to handle fallback methods in microservices?
Fallback methods are used to provide an alternative response when a microservice fails. In Hystrix, fallback methods can be configured using the `@HystrixCommand` annotation.

### 109. What is Spring Cloud Gateway?
Spring Cloud Gateway is a library for building API gateways on top of Spring WebFlux. It provides a way to route requests to multiple microservices and can be used to implement cross-cutting concerns like security, monitoring, and resiliency.

### 110. How to route requests in API gateway?
Requests can be routed in an API gateway by configuring routes that map to different microservices. In Spring Cloud Gateway, routes can be configured using a fluent Java API or by using properties files.

### 111. What is load balancing in microservices?
Load balancing is the process of distributing network traffic across multiple servers. In a microservices architecture, load balancing is used to distribute requests across multiple instances of a service.

### 112. Difference between client-side and server-side load balancing?
- **Client-Side Load Balancing:** The client is responsible for choosing which service instance to send a request to. Ribbon is an example of a client-side load balancer.
- **Server-Side Load Balancing:** A dedicated load balancer (such as Nginx or an AWS Elastic Load Balancer) is used to distribute requests to service instances.

### 113. What is distributed tracing?
Distributed tracing is a technique that is used to monitor and troubleshoot requests that span multiple microservices. It allows you to see the entire lifecycle of a request, from the time it enters the system until it is completed.

### 114. What is Spring Cloud Sleuth?
Spring Cloud Sleuth is a library that provides distributed tracing for Spring Boot applications. It automatically adds trace and span IDs to the logs, which can be used to trace requests across multiple services.

### 115. How to propagate logs across microservices?
Logs can be propagated across microservices by using a centralized logging solution, such as the ELK stack (Elasticsearch, Logstash, and Kibana). Each microservice sends its logs to a central location, where they can be aggregated and analyzed.

### 116. What is Zipkin?
Zipkin is a distributed tracing system that is used to collect and visualize timing data for requests that span multiple microservices.

### 117. How to handle centralized configuration in microservices?
Centralized configuration can be handled in a microservices architecture by using a configuration server. The configuration server provides a central place to manage the configuration for all the microservices.

### 118. What is Spring Cloud Config Server?
Spring Cloud Config Server is a library that provides a centralized configuration server for a microservices architecture. It can be used to manage the configuration for all the microservices in a central place.

### 119. How to implement dynamic configuration refresh?
Dynamic configuration refresh can be implemented by using the Spring Cloud Config Server and the `/actuator/refresh` endpoint. When the configuration is updated in the config server, the `/actuator/refresh` endpoint can be called to refresh the configuration of the microservices.

### 120. What is circuit breaker pattern?
The circuit breaker pattern is a design pattern that is used to prevent a network or service failure from cascading to other services. It works by wrapping a protected function call in a circuit breaker object, which monitors for failures.

### 121. How to implement retry strategy in microservices?
A retry strategy can be implemented in microservices by using a library like Spring Retry. This library provides a simple way to add retry functionality to any method.

### 122. What is bulkhead pattern?
The bulkhead pattern is a design pattern that is used to isolate elements of an application into pools so that if one fails, the others will continue to function.

### 123. What is idempotency in microservices?
Idempotency is the property of an operation that means it can be applied multiple times without changing the result beyond the initial application. In a microservices architecture, idempotency is important for ensuring that requests are not processed multiple times.

### 124. How to implement rate-limiting?
Rate-limiting can be implemented in a microservices architecture by using a library like Resilience4j. This library provides a simple way to add rate-limiting functionality to any method.

### 125. How to implement caching in microservices?
Caching can be implemented in a microservices architecture by using a distributed cache, such as Redis or Memcached.

### 126. What is distributed caching?
Distributed caching is a technique that is used to cache data across multiple servers. This can improve performance and scalability by reducing the number of requests that need to be made to the database.

### 127. How to use Redis in microservices?
Redis can be used in a microservices architecture as a distributed cache, a message broker, or a primary database.

### 128. How to implement messaging with RabbitMQ?
Messaging with RabbitMQ can be implemented in a microservices architecture by using the Spring AMQP library. This library provides a simple way to send and receive messages with RabbitMQ.

### 129. How to implement messaging with Kafka?
Messaging with Kafka can be implemented in a microservices architecture by using the Spring for Apache Kafka library. This library provides a simple way to send and receive messages with Kafka.

### 130. What is event-driven architecture?
Event-driven architecture is a software architecture pattern in which the flow of the application is determined by events. In an event-driven architecture, services communicate with each other by sending and receiving events.



