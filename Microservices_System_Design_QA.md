# Microservices & System Design Interview Questions & Answers

This document contains in-depth answers to Microservices Architecture, Distributed Systems, and System Design scenario-based questions.

## Table of Contents
1. [Microservices Architecture Basics](#architecture)
2. [Service Communication & Discovery](#communication)
3. [Resilience & Fault Tolerance](#resilience)
4. [Distributed Data & Transactions](#distributed-data)
5. [System Design Patterns & Scenarios](#system-design)

---

## Microservices Architecture Basics

### 1. What is Microservices Architecture?
**Answer:**
An architectural style that structures an application as a collection of loosely coupled, independently deployable services.
*   Organized around business capabilities.
*   Owned by small, cross-functional teams.
*   Technology agnostic (polyglot).

### 2. Difference between Monolith and Microservices?
**Answer:**
*   **Monolith:** Single deployment unit, shared database, function calls for communication, simple to develop/test initially, hard to scale/maintain as it grows.
*   **Microservices:** Multiple deployment units, database per service, network calls (REST/gRPC), complex ops/testing, independent scaling, failure isolation.

### 3. What is Service Discovery?
**Answer:**
Mechanism for services to find each other dynamically without hardcoding IP addresses (which change in cloud env).
*   **Server-Side:** Client calls Load Balancer -> LB queries Registry -> LB forwards to Service.
*   **Client-Side:** Client queries Registry -> Gets IP list -> Load Balances internally -> Calls Service. (e.g., Eureka + Ribbon).

### 4. What is Eureka?
**Answer:**
A Service Registry (by Netflix).
*   **Eureka Server:** The registry where services register themselves.
*   **Eureka Client:** Services that register with server and send heartbeats.

### 5. What is API Gateway? Why use it?
**Answer:**
A single entry point for all clients.
*   **Routing:** Forwards requests to appropriate microservice (`/api/user` -> User Service).
*   **Cross-cutting concerns:** Authentication, SSL termination, Rate Limiting, Logging, CORS.
*   **Aggregation:** Combine results from multiple services.
*   Example: Spring Cloud Gateway, Zuul.

### 6. What is Spring Cloud Config?
**Answer:**
Centralized configuration management for distributed systems.
*   Stores config in Git/SVN/Vault.
*   Services fetch config at startup.
*   Supports dynamic refresh (`/actuator/refresh`) without restart.

---

## Communication & Resilience

### 7. What is Feign Client?
**Answer:**
Declarative REST client.
*   Allows writing web service clients by defining an interface with annotations.
*   Integrates with Eureka (Load balancing) and Circuit Breakers.

### 8. What is Circuit Breaker Pattern?
**Answer:**
Prevents cascading failures. If a service is failing/slow, the circuit "opens" and immediately returns an error or fallback response without waiting for timeout.
*   **States:** Closed (Normal), Open (Failing), Half-Open (Testing recovery).
*   Impl: Resilience4j, Hystrix (EOL).

### 9. What is Bulkhead Pattern?
**Answer:**
Isolates resources (threads/pools) for different services. If one service exhausts its thread pool, others are unaffected. (Like ship bulkheads).

### 10. How to implement Rate Limiting?
**Answer:**
Restricts the number of requests a user can make in a given timeframe.
*   **Algorithms:** Token Bucket, Leaky Bucket.
*   **Tools:** Redis (distributed counter), Bucket4j, API Gateway.

### 11. Difference between Client-side and Server-side Load Balancing?
**Answer:**
*   **Client-side:** Client holds the list of servers and picks one (Ribbon). No extra hop.
*   **Server-side:** Traffic goes to a dedicated HW/SW Load Balancer (Nginx, AWS ELB) which distributes traffic. Extra hop.

### 12. What is Distributed Tracing? (Sleuth/Zipkin)
**Answer:**
Tracking a request as it flows across multiple microservices.
*   **Trace ID:** Unique ID for the whole request chain.
*   **Span ID:** Unique ID for a single operation/hop.
*   **Zipkin/Jaeger:** UI to visualize traces and find latency bottlenecks.

---

## Distributed Data & Transactions

### 13. What is Database per Service pattern?
**Answer:**
Each microservice manages its own database. Other services cannot access it directly.
*   **Pros:** Loose coupling, independent scaling.
*   **Cons:** Distributed transactions, complex queries (joins).

### 14. How to handle Distributed Transactions?
**Answer:**
ACID transactions across services are difficult (2PC is slow/blocking). Preferred approach: **Saga Pattern**.
*   **Saga:** Sequence of local transactions. If one fails, execute **Compensating Transactions** to undo changes.
*   **Choreography:** Events trigger next step.
*   **Orchestration:** Central coordinator directs steps.

### 15. What is Two-Phase Commit (2PC)?
**Answer:**
Strong consistency protocol.
1.  **Prepare:** Coordinator asks all participants if they can commit. Lock resources.
2.  **Commit:** If all say Yes -> Commit. If any No -> Rollback.
*   Problem: Blocking, single point of failure.

### 16. What is CQRS (Command Query Responsibility Segregation)?
**Answer:**
Splitting the model into two:
*   **Command Model:** Handles Writes (Create/Update). Complex validation.
*   **Query Model:** Handles Reads. Optimized for display (Denormalized views).
*   Often used with Event Sourcing.

### 17. What is Event Sourcing?
**Answer:**
Storing state as a sequence of events (e.g., `OrderCreated`, `ItemAdded`) rather than current state.
*   Current state is derived by replaying events.
*   Audit trail, time travel debugging.

### 18. How to handle Eventual Consistency?
**Answer:**
Accept that system won't be consistent immediately.
*   Use retries.
*   Idempotent consumers (handle duplicate events safely).
*   Dead Letter Queues (for unprocessable messages).

### 19. What is Idempotency?
**Answer:**
Making multiple identical requests has the same effect as a single request.
*   Crucial for retries in distributed systems.
*   Impl: Track processed Message IDs in DB.

---

## System Design Scenarios

### 20. How to design for High Scalability?
**Answer:**
*   **Horizontal Scaling:** Add more instances. stateless services.
*   **Caching:** Redis/Memcached to offload DB.
*   **Async Processing:** Message Queues (Kafka/RabbitMQ) for non-critical tasks.
*   **Database Sharding:** Partition data.

### 21. How to design a Real-time Chat App?
**Answer:**
*   **Protocol:** WebSockets (duplex) or MQTT.
*   **Scaling:** Stateful connections. Need Sticky Sessions or Pub/Sub (Redis) to broadcast messages across server nodes.
*   **Storage:** Cassandra/HBase for chat history (high write throughput).

### 22. How to implement Notification System?
**Answer:**
*   **Queue:** Decouple trigger from delivery.
*   **Workers:** Consume from queue and send (Email/SMS/Push).
*   **Rate Limiting:** Protect downstream providers.
*   **Prioritization:** OTP (High) vs Marketing (Low).

### 23. How to handle 1 Million Concurrent Users?
**Answer:**
*   **Load Balancer:** Layer 4/7 LB (AWS ALB).
*   **CDN:** Serve static assets.
*   **Caching:** Multi-level (Browser, CDN, App, DB).
*   **DB Optimization:** Read Replicas, Sharding, Indexing.
*   **Asynchronous:** Non-blocking I/O (WebFlux/Node.js).

### 24. What is CAP Theorem?
**Answer:**
In a distributed system, you can only pick 2 out of 3:
*   **Consistency:** Every read receives the most recent write.
*   **Availability:** Every request receives a (non-error) response.
*   **Partition Tolerance:** System continues to operate despite network messages dropping.
*   In Microservices, we usually choose **AP** (Availability + Partition Tolerance) + Eventual Consistency.


---

## Security & Advanced Communication

### 25. How to secure inter-service communication?
**Answer:**
1.  **mTLS (Mutual TLS):** Both client and server authenticate each other using certificates. (Service Mesh like Istio handles this well).
2.  **JWT Propagation:** Pass the user's JWT token in the `Authorization` header downstream.
3.  **Client Credentials Flow:** Services authenticate themselves using Client ID/Secret to get their own token.

### 26. What is OAuth2 Authorization Code Flow?
**Answer:**
Used for user authentication in web apps.
1.  User redirected to Auth Server (Login).
2.  Auth Server redirects back with `code`.
3.  App exchanges `code` for `access_token` and `refresh_token`.
4.  More secure than Implicit Flow as tokens are not exposed in browser URL.

### 27. How to implement Refresh Tokens?
**Answer:**
Access tokens are short-lived (e.g., 15 mins). Refresh tokens are long-lived (days).
*   When access token expires (401), client sends Refresh Token to Auth Server.
*   Auth Server validates it and issues new Access Token + new Refresh Token (Rotation).
*   Allows revoking access by invalidating the refresh token.

### 28. What is gRPC and how is it different from REST?
**Answer:**
*   **Protocol:** gRPC uses HTTP/2 (binary, multiplexing). REST uses HTTP/1.1 (text/JSON).
*   **Format:** gRPC uses Protocol Buffers (Protobuf) - smaller, faster serialization. REST uses JSON.
*   **Contracts:** gRPC requires strict `.proto` contract. REST is flexible (OpenAPI optional).
*   **Use case:** gRPC for internal low-latency microservices. REST for public APIs.

### 29. How to handle partial failures (Graceful Degradation)?
**Answer:**
*   **Fallback:** Return cached data or default value (e.g., "Recommendations unavailable" instead of 500 Error).
*   **Timeouts:** Fail fast if dependency is slow.
*   **Asynchronous:** Decouple non-critical paths via messaging.

---

## Advanced Data Patterns & Caching

### 30. How to implement Distributed Caching?
**Answer:**
Using an external cache cluster (Redis/Memcached) shared by all instances.
*   **Patterns:**
    *   **Cache-Aside:** App reads DB -> writes to Cache.
    *   **Write-Through:** App writes to Cache -> Cache writes to DB.
    *   **Write-Back:** App writes to Cache -> Cache writes to DB asynchronously.

### 31. What is the Thundering Herd problem?
**Answer:**
When many clients request the same expired cache item simultaneously. All miss the cache and hit the DB at once, causing load spike.
*   **Solution:** Use **Distributed Locks** (Mutex) so only one instance updates the cache, or **Probabilistic Early Expiration**.

### 32. How to implement Distributed Locking?
**Answer:**
Ensuring only one process performs a task across the cluster.
*   **Redis (Redlock):** `SET resource_name my_random_value NX PX 30000`.
*   **ZooKeeper/Etcd:** Ephemeral nodes.
*   **Database:** `SELECT ... FOR UPDATE`.

### 33. What is Database Sharding?
**Answer:**
Partitioning data horizontally across multiple database servers.
*   **Shard Key:** Determines which server holds the data (e.g., UserID % 4).
*   **Pros:** Infinite scaling of write throughput.
*   **Cons:** Complex joins (cross-shard), rebalancing data is hard.

### 34. What is the Outbox Pattern?
**Answer:**
Solves the "Dual Write Problem" (Writing to DB and publishing to Kafka atomically).
1.  Save entity AND an "Outbox" event record in the *same* DB transaction.
2.  A background poller (or CDC tool like Debezium) reads the Outbox table and publishes to Kafka.
3.  Ensures At-Least-Once delivery.

### 35. How to handle Message Deduplication (Idempotent Consumer)?
**Answer:**
Message queues (Kafka/RabbitMQ) guarantee "At-Least-Once" delivery, meaning duplicates can happen.
*   **Solution:** Store `message_id` in a unique constraint table (or Redis set). Check before processing.

---

## Deployment & Observability

### 36. What is Blue-Green Deployment?
**Answer:**
Two identical environments: Blue (Live) and Green (Idle).
1.  Deploy new version to Green.
2.  Test Green.
3.  Switch Load Balancer to point to Green.
4.  Blue becomes idle.
*   **Pros:** Instant rollback. **Cons:** Double cost.

### 37. What is Canary Deployment?
**Answer:**
Roll out new version to a small subset of users (e.g., 5%).
*   Monitor metrics (Errors, Latency).
*   If good, gradually increase traffic (10% -> 50% -> 100%).
*   If bad, rollback immediately.

### 38. How to monitor CPU/Memory in Microservices?
**Answer:**
*   **cAdvisor:** Collects container metrics.
*   **Prometheus:** Scrapes metrics.
*   **Grafana:** Visualizes dashboards.
*   **Node Exporter:** Machine-level metrics.

### 39. What is Chaos Engineering?
**Answer:**
Intentionally injecting faults (latency, killing pods, network drops) into the system to test resilience.
*   **Tool:** Chaos Monkey.

---

## Real-Time & High Scale Scenarios

### 40. Design a Stock Market Feed?
**Answer:**
*   **Requirements:** High throughput updates, low latency.
*   **Ingestion:** Kafka for raw tick data.
*   **Processing:** Stream Processing (Kafka Streams/Flink) for aggregations (OHLC candles).
*   **Delivery:** WebSockets to clients.
*   **Optimization:** Binary format (Protobuf), conflation (send only latest price if changing too fast).

### 41. Design a Unique ID Generator (like Snowflake)?
**Answer:**
Generate unique, sortable 64-bit IDs in distributed system.
*   **Structure:** Timestamp (41 bits) + Machine ID (10 bits) + Sequence (12 bits).
*   **No central DB:** Each node generates IDs independently.

### 42. Design a Rate Limiter?
**Answer:**
*   **Distributed:** Use Redis Lua script.
*   **Algorithm:** Sliding Window Log or Token Bucket.
*   **Granularity:** IP-based or User-based.

### 43. Design a Leaderboard?
**Answer:**
*   **Data Structure:** Redis Sorted Sets (`ZADD`, `ZRANGE`).
*   **Operations:** `ZINCRBY` (Update score), `ZREVRANK` (Get rank).
*   **Scale:** Sharding by score range or user ID if specific rank not needed globally.

### 44. How to handle Hot Partitions in Kafka?
**Answer:**
Occurs when one partition gets disproportionate data (e.g., partitioning by UserID and one user is very active).
*   **Solution:** Use a better partition key (add entropy), or "salting" the key for hot keys.

### 45. Difference between Orchestration and Choreography in Sagas?
**Answer:**
*   **Choreography (Decentralized):** Service A emits event -> Service B listens & acts -> Emits event -> Service C listens. No central brain. Hard to track.
*   **Orchestration (Centralized):** A Central Orchestrator Service (state machine) calls A, then B, then C. Handles rollbacks. Easier to manage.

---

## Design Patterns in Java

### 46. Singleton Pattern?
**Answer:**
Ensures a class has only one instance.
*   **Spring:** Default bean scope is Singleton.
*   **Java:** Enum Singleton (Thread-safe, Serialization-safe).

### 47. Factory Pattern?
**Answer:**
Creates objects without specifying the exact class.
*   **Spring:** `BeanFactory`.
*   **Usage:** When implementation depends on configuration (e.g., `PaymentFactory.getPaymentStrategy("PAYPAL")`).

### 48. Strategy Pattern?
**Answer:**
Defines a family of algorithms, encapsulates each one, and makes them interchangeable.
*   **Usage:** Payment methods, Validation rules.
*   **Implementation:** Interface `PaymentStrategy` -> Classes `PayPalStrategy`, `CreditCardStrategy`. Map strategies in a Map<String, Strategy>.

### 49. Observer Pattern?
**Answer:**
One-to-many dependency. When one object changes state, dependents are notified.
*   **Spring:** Application Events (`ApplicationEventPublisher`, `@EventListener`).

### 50. Builder Pattern?
**Answer:**
Constructs complex objects step by step.
*   **Usage:** `Lombok @Builder`.
*   `User.builder().name("John").age(30).build();`


---

## Additional Design Patterns (GOF)

### 51. Adapter Pattern?
**Answer:**
Allows incompatible interfaces to work together. Wraps an existing class with a new interface.
*   **Usage:** Integrating 3rd party libraries.
*   **Example:** `XmlToJsonAdapter`.

### 52. Facade Pattern?
**Answer:**
Provides a simplified interface to a complex system of classes.
*   **Usage:** API Gateway is a Facade for microservices. `ServiceFacade` wrapping multiple Repositories.

### 53. Decorator Pattern?
**Answer:**
Adds behavior to an object dynamically without altering its structure.
*   **Usage:** `java.io` classes (`BufferedReader(FileReader)`). Wrapper classes.

### 54. Proxy Pattern?
**Answer:**
Represents another object to control access to it.
*   **Usage:** Lazy Loading (Hibernate), AOP (Spring), Security controls.

### 55. Chain of Responsibility?
**Answer:**
Passes a request along a chain of handlers.
*   **Usage:** Spring Security Filters, Logging chains, Exception handling.

### 56. Command Pattern?
**Answer:**
Encapsulates a request as an object.
*   **Usage:** Job Queues, Undo/Redo operations. `Runnable` is a command.

### 57. Template Method Pattern?
**Answer:**
Defines the skeleton of an algorithm, letting subclasses override specific steps.
*   **Usage:** `JdbcTemplate`, `RestTemplate`. `handleRequest()` in Servlet.

### 58. Iterator Pattern?
**Answer:**
Access elements of a collection sequentially without exposing underlying representation.
*   **Usage:** `java.util.Iterator`, `foreach` loop.

### 59. Composite Pattern?
**Answer:**
Composes objects into tree structures to represent part-whole hierarchies.
*   **Usage:** UI Components, File Systems (Folder contains Files/Folders).

### 60. State Pattern?
**Answer:**
Allows an object to alter its behavior when its internal state changes.
*   **Usage:** Order Processing (New -> Paid -> Shipped). Finite State Machines.

---

## More Real-Time & Distributed Scenarios

### 61. How to design a Gaming Leaderboard (Real-Time)?
**Answer:**
*   **Data Structure:** Redis Sorted Set (ZSET).
*   **Operations:** `ZINCRBY` score. `ZREVRANGE` for top 10.
*   **Optimization:** If millions of users, use **Lossy Counting** (Count-Min Sketch) or Shard by rank range (Top 1000 in one shard, others in another).

### 62. How to implement "Who is Online" (Presence)?
**Answer:**
*   **Heartbeat:** Client sends ping every 30s.
*   **Storage:** Redis Key `online:userid` with TTL 35s.
*   **Check:** If Key exists, user is online.

### 63. How to design a URL Shortener (TinyURL)?
**Answer:**
*   **Core:** Map Long URL <-> Short ID.
*   **ID Generation:** Base62 encoding of a unique Database ID or Snowflake ID.
*   **Storage:** Key-Value store (NoSQL) for fast lookups.
*   **Collision:** Pre-generate keys (Key Generation Service) to avoid DB unique checks.

### 64. How to design a Web Crawler?
**Answer:**
*   **Queue:** URL Frontier (Kafka) to store URLs to visit.
*   **Deduplication:** Bloom Filter to check if URL already visited.
*   **Politeness:** Rate limit per domain.
*   **Storage:** BigTable/HBase for content.

### 65. How to handle "Flash Sales" (High Concurrency)?
**Answer:**
*   **Queue:** All requests go to Kafka first.
*   **Async:** Process orders from queue sequentially.
*   **Redis:** Decrement stock in Redis (Atomic). If stock < 0, reject.
*   **Frontend:** Show "Processing..." until Async job finishes.

### 66. How to design a Collaborative Editor (Google Docs)?
**Answer:**
*   **Algorithm:** Operational Transformation (OT) or CRDT (Conflict-free Replicated Data Type).
*   **Communication:** WebSockets for real-time keystrokes.
*   **Resolution:** Server resolves conflicting timestamps and pushes merged state.

### 67. How to design a Geo-Spatial Service (Uber/Grab)?
**Answer:**
*   **Indexing:** QuadTree or Geohash (Redis `GEOADD`).
*   **Query:** "Find drivers within 5km" -> `GEORADIUS`.
*   **Updates:** Drivers stream location (Kafka) -> Update Redis.

### 68. How to implement Distributed Rate Limiting?
**Answer:**
*   **Sliding Window Log:** Precise but high memory.
*   **Token Bucket:** Efficient. Store "Tokens" and "LastRefillTime" in Redis.
*   **Lua Script:** Ensure `Get` and `Decrement` are atomic in Redis.

### 69. How to design a Metric Monitoring System?
**Answer:**
*   **Model:** Time-Series Database (TSDB) like InfluxDB or Prometheus.
*   **Pull vs Push:** Prometheus pulls from endpoints.
*   **Aggregation:** Downsample old data (1 min res -> 1 hour res) to save space.

### 70. How to handle "Heavy Hitters" (Top K elements)?
**Answer:**
*   **Problem:** Find most frequent items in a massive stream.
*   **Solution:** **Count-Min Sketch** (Probabilistic data structure). Uses minimal memory to estimate counts with small error.

