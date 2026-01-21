# Design Patterns Interview Questions & Answers (In-Depth Guide)

This document covers GOF (Gang of Four) design patterns and their real-world applications in Java and Spring Framework.

## Table of Contents
1.  [Creational Patterns](#creational)
2.  [Structural Patterns](#structural)
3.  [Behavioral Patterns](#behavioral)
4.  [Microservices Patterns](#cloud)

---

## <a name="creational"></a>1. Creational Patterns

### Q1: Singleton Pattern - How to break it?
**Implementation:** Enum Singleton (Best).
```java
public enum Singleton {
    INSTANCE;
    public void doSomething() { ... }
}
```
**Breaking Singleton:**
1.  **Reflection:** Change constructor visibility. (Enum handles this).
2.  **Serialization:** Deserializing creates new object. (Enum handles this).
3.  **Cloning:** Override `clone()`.
4.  **ClassLoaders:** Two classloaders loading the same class creates two instances.

### Q2: Factory Pattern vs Abstract Factory?
*   **Factory Method:** Creates **one** type of product (e.g., `Button`). Subclasses decide which class to instantiate.
*   **Abstract Factory:** Creates **families** of related products (e.g., `Button` + `Checkbox` + `Window` for MacOS vs Windows).

### Q3: Builder Pattern (Fluent API)
**Use Case:** Constructing complex objects with many optional parameters.
**Java Example:** `StringBuilder`.
**Lombok:** `@Builder` generates the inner static builder class automatically.
```java
User user = User.builder()
    .firstName("John")
    .age(30)
    .build();
```

### Q4: Prototype Pattern
**Concept:** Creating a new object by copying an existing one (cloning).
**Use Case:** When object creation is expensive (DB calls).
**Java:** `implements Cloneable`.
**Spring:** `scope="prototype"`.

---

## <a name="structural"></a>2. Structural Patterns

### Q5: Adapter Pattern
**Concept:** Makes incompatible interfaces work together.
**Real World:**
*   `Arrays.asList()`: Adapts Array to List interface.
*   `InputStreamReader`: Adapts Stream to Reader.
*   **Spring:** `HandlerAdapter` adapts different Controller types to DispatcherServlet.

### Q6: Decorator Pattern
**Concept:** Dynamically adds behavior to an object without altering its structure.
**Real World:**
*   `java.io`: `new BufferedReader(new FileReader(file))`.
*   **Spring:** `BeanWrapper`.

### Q7: Proxy Pattern
**Concept:** Controls access to an object.
**Types:**
*   **Remote Proxy:** RMI / gRPC stub.
*   **Virtual Proxy:** Lazy loading (Hibernate).
*   **Protection Proxy:** Security checks.
**Spring AOP:** Uses Dynamic Proxies to add Transaction/Security logic.

### Q8: Facade Pattern
**Concept:** Simple interface to a complex subsystem.
**Real World:**
*   **API Gateway:** Facade for Microservices.
*   **SLF4J:** Facade for logging implementations.

### Q9: Bridge Pattern
**Concept:** Decouples abstraction from implementation so they vary independently.
**Example:** `JDBC`.
*   Abstraction: `Connection`, `Driver` interfaces.
*   Implementation: `MySQLDriver`, `OracleDriver`.

---

## <a name="behavioral"></a>3. Behavioral Patterns

### Q10: Strategy Pattern
**Concept:** Defines a family of algorithms and makes them interchangeable.
**Use Case:** Payment Methods.
```java
public interface PaymentStrategy { void pay(int amount); }
public class CreditCard implements PaymentStrategy { ... }
public class PayPal implements PaymentStrategy { ... }
```
**Spring:** injecting `PaymentStrategy` based on config.

### Q11: Observer Pattern
**Concept:** Pub/Sub. One changes, others notified.
**Java:** `java.util.Observer` (Deprecated). `PropertyChangeListener`.
**Spring:** `ApplicationEventPublisher` and `@EventListener`.

### Q12: Template Method Pattern
**Concept:** Defines skeleton of algorithm. Subclasses override steps.
**Spring:** `JdbcTemplate`, `RestTemplate`, `JmsTemplate`.
*   They handle the boilerplate (Open connection, handle exception, close connection).
*   You provide the specific logic (RowMapper).

### Q13: Chain of Responsibility
**Concept:** Pass request along a chain of handlers.
**Real World:**
*   **Servlet Filters:** `doFilter()`.
*   **Spring Security:** Filter Chain.
*   **Logback:** Appenders.

### Q14: Command Pattern
**Concept:** Encapsulate request as object.
**Use Case:** `Runnable` (Thread pool), Undo/Redo operations.

### Q15: Iterator Pattern
**Concept:** Access elements without exposing representation.
**Java:** `java.util.Iterator`, `foreach` loop.

---

## <a name="cloud"></a>4. Microservices / Cloud Patterns

### Q16: Sidecar Pattern
**(Covered in Microservices File)**. Helper container.

### Q17: Ambassador Pattern
Helper container handling outgoing network connections (Proxy).

### Q18: Anti-Corruption Layer (ACL)
**Concept:**
When migrating Legacy -> Microservices.
Don't let Legacy's bad domain model pollute the new Microservice.
**Impl:**
Create an adapter layer that translates Legacy Model <-> New Model.

### Q19: Backend For Frontend (BFF)
**Concept:**
Instead of one general API Gateway, create specific Gateways for specific Clients.
*   `Mobile BFF`: Returns minimal JSON.
*   `Web BFF`: Returns rich data.
*   `3rd Party BFF`: Strict rate limits.

### Q20: Retry & Circuit Breaker
**(Covered in Microservices File)**.

