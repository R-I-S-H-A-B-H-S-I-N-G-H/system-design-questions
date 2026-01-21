# Microservices & System Design Interview Questions (In-Depth Guide)

This document provides comprehensive answers to advanced architecture, distributed systems, and system design scenario questions.

## Table of Contents
1.  [Microservices Architecture & Patterns](#architecture)
2.  [Distributed Data Management (Saga, CQRS)](#distributed-data)
3.  [Resilience & Fault Tolerance](#resilience)
4.  [Scalability & High Performance](#scalability)
5.  [Real-Time System Design Scenarios](#scenarios)

---

## <a name="architecture"></a>1. Microservices Architecture & Patterns

### Q1: What is the API Gateway Pattern? (Zuul / Spring Cloud Gateway)
**Concept:**
In a microservices architecture, a client (Mobile/Web) should not communicate directly with 50 different microservices. It should talk to a single entry point.

**Responsibilities:**
1.  **Routing:** Like a reverse proxy. `/api/users` -> `User Service`, `/api/orders` -> `Order Service`.
2.  **Aggregation:** (Advanced) It can call User Service and Order Service and return a combined JSON to mobile clients to save battery/bandwidth.
3.  **Cross-Cutting Concerns (Offloading):**
    *   **Authentication:** Validate JWT at Gateway. Pass "User-ID" header downstream.
    *   **SSL Termination:** Handle HTTPS at Gateway, use HTTP internally.
    *   **Rate Limiting:** Protect backend from DDoS.
    *   **Logging/Analytics:** Capture all incoming traffic stats.

**Design Issue:** It becomes a Single Point of Failure and a bottleneck.
**Solution:** Deploy multiple instances of Gateway behind a Load Balancer (AWS ALB).

---

### Q2: Service Discovery (Eureka) - Why do we need it?
**The Problem:**
In the cloud (AWS/K8s), services scale up and down dynamically. An IP address is assigned effectively at startup.
Service A cannot "hardcode" the IP of Service B (`http://192.168.1.50:8080`) because that IP will die/change.

**The Mechanism:**
1.  **Registration:** When Service B starts, it calls Eureka Server: "I am SERVICE-B, my IP is X, Port Y". It sends heartbeat every 30s (`renew`).
2.  **Discovery:** Service A asks Eureka: "Give me list of IPs for SERVICE-B".
3.  **Load Balancing (Client-Side):** Service A (using **Ribbon** or **Spring Cloud LoadBalancer**) picks one IP from the list (Round Robin) and makes the call.

**Difference: K8s Service Discovery:**
Kubernetes has built-in discovery (DNS). Service A calls `http://service-b`. K8s resolves this to a ClusterIP (Virtual IP) which load balances to Pods. Spring Cloud Eureka is less common in K8s environments, but still useful for hybrid deployments.

---

## <a name="distributed-data"></a>2. Distributed Data Management

### Q3: How to handle Distributed Transactions? (Saga Pattern)
**The Problem:**
In Monolith, `OrderService` and `InventoryService` share one DB. You use `COMMIT/ROLLBACK` (ACID).
In Microservices, they have different DBs. You cannot lock both DBs (2PC is too slow/blocking).

**Solution: The Saga Pattern:**
A Saga is a sequence of local transactions. Each service updates its own DB and publishes an event/message to trigger the next step.

**Example (Order -> Payment -> Inventory):**
1.  **Order Service:** Creates Order (PENDING). Publishes `OrderCreated` event.
2.  **Payment Service:** Listens to event. Charges card. Publishes `PaymentSuccess`.
3.  **Inventory Service:** Listens. Reserves items. Publishes `InventoryReserved`.
4.  **Order Service:** Listens. Updates Order to CONFIRMED.

**Failure Handling (Compensating Transactions):**
If Inventory fails (Out of stock):
1.  Inventory publishes `InventoryFailed`.
2.  Payment Service listens -> **Refunds** the user (Compensating Action).
3.  Order Service listens -> Updates Order to FAILED.

**Approaches:**
*   **Choreography:** Services talk via events (Kafka). Decentralized. Hard to visualize flow.
*   **Orchestration:** A central "Orchestrator" service (or State Machine like **Camunda** / **Spring StateMachine**) commands services: "Payment, do this. Inventory, do that." Easier to manage complex flows.

---

### Q4: What is CQRS (Command Query Responsibility Segregation)?
**Concept:**
Separating the Read (Query) and Write (Command) models of the application.

**Why?**
*   **Writes:** Complex validation. Normalized DB (3rd Normal Form) to avoid redundancy.
*   **Reads:** Often require Joins across 10 tables. Normalized DB is slow for complex reads.

**Implementation:**
1.  **Command Side:** Application writes to Main DB (Postgres). Optimized for consistency.
2.  **Sync Mechanism:** Events are published (Kafka) when data changes.
3.  **Query Side:** A separate DB (NoSQL like Mongo or Search Engine like ElasticSearch) consumes events and builds a **denormalized** view.
    *   Example: A `UserOrders` document that contains User details AND List of Order Summaries pre-joined.
4.  **Reads:** Application reads strictly from the Query DB (Extremely fast).

**Trade-off:** **Eventual Consistency**. The Query DB might lag behind the Write DB by a few milliseconds.

---

## <a name="resilience"></a>3. Resilience & Fault Tolerance

### Q5: Explain Circuit Breaker Pattern (Resilience4j).
**The Scenario:**
Service A calls Service B. Service B hangs (DB locked).
Service A threads wait... and wait. Eventually, all threads in A are stuck waiting for B. Service A crashes (Cascading Failure).

**The Solution:**
Wrap the call in a Circuit Breaker.
1.  **Closed State (Normal):** Calls go through. Monitor failure rate.
2.  **Open State (Tripped):** If failures > 50%, circuit "Opens".
    *   **Fail Fast:** Subsequent calls are *immediately* rejected (Exception or Fallback) without calling B. This saves A's threads.
3.  **Half-Open State (Recovery):** After `waitDuration` (e.g., 5s), allow 1-2 calls to pass.
    *   Success? Close circuit (Resume).
    *   Fail? Open again.

**Fallback:**
What to return when Open?
*   Default value (Empty list).
*   Cached data (Stale but better than error).
*   Friendly error message ("Recommendation engine currently unavailable").

---

### Q6: What is the "Thundering Herd" Problem? How to solve it?
**Scenario:**
You cache a popular item (e.g., "Homepage Config") in Redis with TTL 10 minutes.
At 10:00, the key expires.
At 10:00:01, **10,000 users** request the homepage simultaneously.
All 10,000 get a "Cache Miss".
All 10,000 hit the Database at the exact same millisecond.
**Result:** DB CPU spikes to 100%, DB crashes.

**Solutions:**
1.  **Mutex / Distributed Lock:**
    *   When cache miss happens, try to acquire a Lock (Redis `SETNX`).
    *   Only 1 thread gets the lock -> computes value -> updates cache.
    *   Other 9,999 threads wait/retry or return stale data.
2.  **Jitter / Probabilistic Early Expiration:**
    *   If TTL is 10 mins, treat it as expired randomly between 9m and 10m.
    *   This spreads the re-computation load over time before the hard expiry.

---

## <a name="scenarios"></a>4. Real-Time System Design Scenarios

### Q7: Design a Rate Limiter (like in API Gateway).
**Requirements:** Limit user to 10 requests per minute.

**Approaches:**
1.  **Token Bucket Algorithm:**
    *   Concept: A bucket holds Tokens. Tokens are added at a fixed rate (e.g., 10/min).
    *   Request: Needs to consume 1 token to pass. If bucket empty -> 429 Too Many Requests.
    *   **Impl:** Redis.
2.  **Sliding Window Log:**
    *   Store timestamps of all requests in a Sorted Set (Redis ZSET).
    *   On request: Remove timestamps older than 1 min. Count remaining. If < 10, allow.
    *   **Pros:** Very accurate. **Cons:** High memory usage (stores all timestamps).
3.  **Sliding Window Counter (Best):**
    *   Hybrid of Fixed Window and Log.
    *   Divide time into small slots. Approximation. Low memory.

**Distributed Implementation:**
Use **Redis + Lua Script**.
Lua script ensures that `get_count` and `increment` happen atomically. Without atomicity, race conditions allow users to exceed limits.

---

### Q8: Design a Real-Time Chat Application (WhatsApp/Slack).
**Key Challenges:** Real-time delivery, Message History, Presence (Online/Offline).

**1. Protocol:**
*   HTTP is Request-Response (Pull). Bad for chat.
*   **WebSockets:** Full-duplex. Persistent connection between Client and Server. Best choice.

**2. Scaling (The Hard Part):**
WebSockets are **stateful**. User A is connected to Server 1. User B is connected to Server 2.
How does Server 1 send a message to User B? It doesn't know User B's connection.

**Solution: Pub/Sub (Redis or Kafka):**
1.  User A sends message to Server 1.
2.  Server 1 saves to DB (Cassandra/HBase for write-heavy history).
3.  Server 1 publishes message to **Redis Channel** `chat_room_123`.
4.  All Servers subscribe to Redis. Server 2 receives event.
5.  Server 2 checks: "Do I have User B connected?" Yes -> Push via WebSocket.

**3. Presence (Who is Online?):**
*   **Heartbeat:** Client sends "ping" every 10s.
*   **Redis:** `SET user:123:status "online" EX 15`. (TTL 15s).
*   If ping stops, key expires automatically -> User Offline.

---

### Q9: Design a URL Shortener (TinyURL).
**Core Problem:** Map a long URL to a short 7-char string.

**1. ID Generation:**
*   **Database Auto-Increment?** Hard to scale.
*   **Hash (MD5)?** Collisions.
*   **Base62 Encoding:**
    *   Use a distributed unique ID generator (Snowflake ID or Redis counter).
    *   Convert ID (integer) to Base62 (`[a-z, A-Z, 0-9]`).
    *   Example: ID `123456789` -> `Tx9bA`.

**2. Storage:**
*   Read-heavy (1 write : 100 reads).
*   **NoSQL (DynamoDB / Cassandra)** or fast KV store.
*   Schema: `{id: "Tx9bA", long_url: "..."}`.

**3. Caching:**
*   Crucial. Memcached/Redis.
*   Store `short_url -> long_url`.
*   Eviction Policy: LRU (Least Recently Used). 20% of URLs generate 80% of traffic.


---

## <a name="resilience"></a>Resilience & Fault Tolerance (Continued)

### Q10: How to implement Idempotency in Distributed Systems?
**The Problem:**
Network is unreliable. Service A calls Service B "Charge User $10".
Response is lost (Timeout).
Service A retries.
If Service B is not idempotent, user is charged $20.

**The Solution:**
1.  **Unique Request ID:** Client sends `X-Request-ID` (UUID).
2.  **Idempotency Key Store:** Service B checks Redis/DB: "Have I processed ID `123-abc`?"
    *   **Yes:** Return cached response immediately (Do not charge again).
    *   **No:** Process charge, save response, save ID.
    *   **In Progress:** Wait or Return error.

**Crucial:** The check and save must be atomic (or use DB Unique Constraints on the ID).

---

### Q11: What is the "Outbox Pattern"? (Dual Write Problem)
**The Scenario:**
You need to:
1.  Save Order to Database.
2.  Publish "OrderCreated" event to Kafka.

**The Failure:**
*   If you save to DB and crash before Kafka -> Data inconsistency.
*   If you publish to Kafka and crash before DB -> Ghost order.
*   You cannot wrap DB and Kafka in one ACID transaction (unless using 2PC, which is slow).

**The Solution (Outbox):**
1.  **Local Transaction:** Insert Order into `OrderTable` AND insert Event into `OutboxTable` (in the same DB). This is ACID.
2.  **CDC / Poller:** A separate process (Debezium or Poller) reads the `OutboxTable` and pushes to Kafka.
3.  **Result:** At-Least-Once delivery guaranteed.

---

### Q12: Explain "Bulkhead Pattern".
**Concept:**
Ship design: Use watertight compartments. If one floods, ship doesn't sink.

**Software:**
Isolate resources (ThreadPools) for different dependencies.
*   Service A calls Service B (slow) and Service C (fast).
*   If B hangs, it consumes all threads of A. A cannot call C anymore.
*   **Bulkhead:** Create Pool-B for B calls, Pool-C for C calls. If Pool-B fills up, calls to B fail, but calls to C continue working.

---

## <a name="scalability"></a>4. Scalability & High Performance

### Q13: How does Consistent Hashing work?
**Use Case:** Distributed Caching (Partitioning keys across N nodes).

**Modulo Hashing (`hash(key) % N`):**
If you add/remove a node (N changes to N+1), **ALL** keys are remapped. Cache is flushed. Disaster.

**Consistent Hashing:**
*   Imagine a Circle (Ring) of hash values (0 to 2^32).
*   Place Nodes on the Ring (hash of Node IP).
*   Place Keys on the Ring.
*   Mapping: Key belongs to the first Node found moving clockwise.
*   **Adding Node:** Only keys between the new node and its neighbor are remapped. Minimal disruption (1/N keys move).
*   **Virtual Nodes:** Use multiple points per node to balance load evenly.

---

### Q14: Database Sharding vs Partitioning vs Replication?
**1. Replication (Master-Slave):**
*   **Goal:** Read Scalability & High Availability.
*   **Master:** Handles Writes.
*   **Slaves:** Replicate data. Handle Reads.

**2. Partitioning (Local):**
*   Splitting a large table into smaller tables *on the same server* (e.g., by Year). Improves query speed.

**3. Sharding (Horizontal Partitioning):**
*   **Goal:** Write Scalability.
*   Splitting data across *multiple physical servers*.
*   **Shard Key:** Determines destination (e.g., UserID).
*   **Complexity:** Cross-shard joins are impossible/expensive.

---

### Q15: CDN (Content Delivery Network) Design?
**Concept:**
Network of edge servers geographically distributed.
**Workflow:**
1.  User requests `image.jpg`.
2.  DNS routes to nearest Edge Server.
3.  **Cache Hit:** Return image.
4.  **Cache Miss:** Edge fetches from Origin (S3), caches it, returns it.

**Interview Q: How to invalidate CDN content?**
*   **Purge:** Explicit API call (slow).
*   **Versioning (Best):** Rename file `image-v2.jpg`. Forces CDN to fetch new file.
*   **TTL:** Set appropriate `Cache-Control` headers.

---

### Q16: How to design a Notification System (Push/Email)?
**Architecture:**
1.  **API Gateway:** Accepts `sendNotification` request.
2.  **Validator:** Validates user, template.
3.  **Message Queue (Kafka):** Critical for decoupling. Separates acceptance from delivery. Prevents overload.
4.  **Workers:** Consume from Kafka.
5.  **Router:** Decides channel (SMS vs Email vs FCM).
6.  **3rd Party Integration:** Calls Twilio/SendGrid.
7.  **Retry Mechanism:** Exponential backoff for failures.
8.  **Deduplication:** Check ID to prevent spamming.

---

### Q17: Design Twitter Feed (Fan-out on Write vs Read)?
**Scenario:** User A tweets. 1M followers see it.

**Approach 1: Pull Model (Fan-out on Read)**
*   A tweets -> Insert into `Tweets` table.
*   Follower B loads feed -> Query: `SELECT * FROM Tweets WHERE user_id IN (following_ids)`.
*   **Pros:** Simple write. **Cons:** Expensive read (complex query for active users).

**Approach 2: Push Model (Fan-out on Write)**
*   A tweets -> System finds all 1M followers.
*   Inserts tweet ID into each follower's "Home Timeline" list (Redis List).
*   Follower B loads feed -> `GET timeline:user_B`. O(1).
*   **Pros:** Instant read. **Cons:** Massive write latency (1M writes). Backlog.

**Hybrid Approach (Twitter uses this):**
*   Regular users: Push Model.
*   Celebrities (Justin Bieber): Pull Model. (Don't push to 100M lists. Merge their tweets at read time).

---

### Q18: What is CAP Theorem? Proof?
**Statement:**
In a Distributed Data Store, you can guarantee only 2 of 3:
1.  **Consistency (C):** Every read receives the most recent write or an error.
2.  **Availability (A):** Every request receives a (non-error) response, without guarantee that it contains the most recent write.
3.  **Partition Tolerance (P):** System continues to operate despite network failures (dropped messages between nodes).

**Proof:**
Since network failures (P) *will* happen in distributed systems, you must choose P.
So the choice is really: **CP vs AP**.
*   **CP (Mongo/HBase):** If network splits, reject writes to ensure consistency. System goes down (Unavailable).
*   **AP (Cassandra/Dynamo):** If network splits, allow writes to both sides. Data diverges (Inconsistent). Sync later (Eventual Consistency).


---

## <a name="patterns-advanced"></a>Advanced Design Patterns & Concepts

### Q19: Design a System for "Flash Sales" (High Concurrency).
**Scenario:** 10,000 iPhones. 10 Million users at 12:00 PM.
**Challenges:** DB crash, Overselling.

**Design Strategy:**
1.  **Static Content:** Serve HTML/JS from CDN. Don't hit backend for page load.
2.  **Request Queueing:** Do not let 10M requests hit DB.
    *   Gateway -> Kafka.
    *   Return "Request Received. Please wait..."
3.  **Inventory Cache:**
    *   Load stock into Redis: `SET iphone_stock 10000`.
    *   **Lua Script:** Atomic `DECR`. If result < 0, return "Sold Out".
    *   This handles millions of ops/sec.
4.  **Async Processing:**
    *   Worker reads Kafka -> If Redis said OK -> Insert Order in DB.
    *   If DB fails, increment Redis back (Compensation).
5.  **Rate Limiting:** Block users trying >1 req/sec.

### Q20: What is Event Sourcing vs CQRS? Can they exist separately?
**Relationship:**
*   They are often used together but are distinct.
*   **Event Sourcing:** Persistence mechanism. Storing history of changes (`UserCreated`, `NameChanged`).
*   **CQRS:** Architectural pattern. Splitting Read/Write models.
**Separation:**
*   You can do **CQRS without Event Sourcing**: Write to SQL, Sync to ElasticSearch. (State-based persistence).
*   You can do **Event Sourcing without CQRS**: Replay events to build state in memory for every read (Inefficient).
*   **Best Together:** Events are the perfect sync mechanism for the Read Model.

### Q21: How to implement "Distributed Locking" with Redis (Redlock)?
**Algorithm (Simplified):**
1.  Client gets current time.
2.  Tries to acquire lock in N Redis instances (sequentially) using `SET resource random_value NX PX 30000`.
3.  Calculates elapsed time.
4.  If Lock acquired in majority (N/2 + 1) instances AND time < TTL -> Success.
5.  If fail, unlock all instances.
**Criticism:** Depends on clock synchronization. If clock jumps, lock validity breaks.
**Production:** Use **Redisson** library (Java) which handles the complexity and "Watchdog" (auto-extension of lock if thread is still working).

### Q22: Design a "Typeahead" / "Autocomplete" System (Google Search).
**Requirements:** Low latency (<100ms), High read throughput.
**Data Structure:** **Trie (Prefix Tree)**.
**Architecture:**
1.  **Storage:** Store frequently searched terms in a Trie.
2.  **Optimization:** Store "Top 5" hot searches at each Trie Node.
3.  **Database:** ElasticSearch (Fuzzy search) or Solr.
4.  **Caching:** Cache the result of prefixes (`"java"` -> `["java tutorial", "java 8"]`) in Redis.
5.  **Sampling:** Don't index every single unique query (spam). Only index queries appearing > N times.

### Q23: What is "Gossip Protocol"?
**Concept:**
Used in decentralized clusters (Cassandra, DynamoDB) to detect node failures and spread metadata.
**Mechanism:**
*   Every second, each node picks a random other node and exchanges state ("I am alive", "Node C is dead").
*   Information spreads like a virus (epidemic) exponentially (`O(log N)`).
*   **Failure Detection:** If Node A hasn't gossiped for X seconds, it is marked suspicious -> then dead.

### Q24: What is "Bloom Filter"?
**Data Structure:** Probabilistic. Space-efficient.
**Answers:** "Definitely No" or "Probably Yes".
**Use Case:**
*   **Databases:** Before checking disk for a key (expensive), check Bloom Filter (memory). If it says "No", skip disk read.
*   **Crawlers:** Has this URL been visited?
**Internals:**
*   Bit array of size M.
*   K hash functions.
*   To Add: Hash item K times, set bits to 1.
*   To Check: Hash item K times. If any bit is 0 -> Definitely No. If all 1 -> Probably Yes (Collision possible).

---

## <a name="deployment"></a>Deployment & DevOps

### Q25: What is "Service Mesh" (Istio/Linkerd)?
**Concept:**
Decouples network logic (retries, security, monitoring) from application code.
**Sidecar Pattern:**
A proxy (Envoy) runs alongside every Microservice container (in the same Pod).
**Features:**
*   **mTLS:** Automatic encryption between services.
*   **Traffic Splitting:** Canary deployment (send 1% to v2).
*   **Observability:** Automatic tracing/metrics without code changes.

### Q26: Strategies for Database Migration in Microservices (Zero Downtime)?
**Scenario:** Renaming column `name` to `fullname`.
**Steps (Expand and Contract):**
1.  **Expand:** Add new column `fullname`. Code writes to BOTH `name` and `fullname`. Reads from `name`.
2.  **Migrate:** Run script to copy old data `name` -> `fullname`.
3.  **Switch:** Deploy code to read from `fullname`.
4.  **Contract:** Remove column `name` and write logic.
**Tools:** Flyway, Liquibase.

### Q27: How to handle "Poison Message" in Kafka?
**Scenario:** A message format is corrupted. Consumer crashes every time it reads it. Infinite loop.
**Solution:**
1.  **Retry:** Try N times with backoff.
2.  **Dead Letter Queue (DLQ):** If retries fail, move message to a separate topic `topic-dlq`.
3.  **Alert:** Notify team to inspect DLQ.
4.  **Commit:** Commit offset in original topic so consumer can move to next message.

### Q28: What is "Backpressure"?
**Concept:**
Fast Producer, Slow Consumer. Memory fills up -> OOM.
**Mechanisms:**
1.  **TCP:** Window size reduces. Sender slows down.
2.  **Reactive Streams:** Consumer sends `request(n)` signal: "I can handle 5 items". Producer sends 5.
3.  **Blocking Queue:** Producer thread blocks if queue full.
4.  **Dropping:** Drop oldest/newest messages.

### Q29: Horizontal vs Vertical Scaling?
*   **Vertical (Scale Up):** Bigger machine (RAM/CPU).
    *   Limit: Hardware ceiling. Single point of failure. Easy (no code change).
*   **Horizontal (Scale Out):** More machines.
    *   Limit: Infinite.
    *   Complexity: Distributed state, Load balancing, Data consistency.

### Q30: What is "Serverless" (Lambda)? Cold Start?
**Concept:**
No managed servers. Code runs on-demand (Event-driven). Bill per millisecond.
**Cold Start:**
If function hasn't run recently, cloud provider must provision container + start JVM.
*   Java has high cold start (JVM startup).
*   **Mitigation:** GraalVM Native Image, SnapStart, Ping (Keep-warm).


---

## <a name="discovery-config"></a>Discovery & Configuration (Advanced)

### Q31: How does Eureka "Self-Preservation" mode work?
**Scenario:**
Network partition happens. Eureka Server cannot reach 50% of instances.
Normally, Eureka should evict them from the registry.
**Self-Preservation:**
If heartbeats drop below a threshold (85%), Eureka stops evicting instances.
**Reason:** It assumes the network is down, not the instances. It prefers to return a stale list than an empty list (Availability > Consistency).

### Q32: Client-Side Load Balancing (Spring Cloud LoadBalancer).
**Mechanism:**
1.  Feign Client asks Eureka for IPs of Service B.
2.  Eureka returns `[IP1, IP2, IP3]`.
3.  LoadBalancer (running inside Client) picks IP1.
4.  Client calls IP1 directly.
**Caching:** Client caches the list. Even if Eureka goes down, Client can still call Service B.

### Q33: How to broadcast configuration changes? (Spring Cloud Bus)
**Scenario:**
You update a property in Git. You want all 50 microservices to pick it up without restarting.
**Solution:**
1.  Push change to Git.
2.  Webhook calls `/monitor` on Config Server.
3.  Config Server publishes a "Refresh Event" to **RabbitMQ/Kafka** (Spring Cloud Bus).
4.  All microservices listening to the Bus receive the event.
5.  They trigger context refresh (`@RefreshScope` beans reload).

---

## <a name="resilience-advanced"></a>Resilience Patterns (Q41-70)

### Q41: Explain "Retry Pattern" with Exponential Backoff.
**Logic:**
If a call fails, retry. But don't retry immediately (spamming).
*   Attempt 1: Fail. Wait 100ms.
*   Attempt 2: Fail. Wait 200ms.
*   Attempt 3: Fail. Wait 400ms.
*   **Jitter:** Add random noise (wait 200ms + random(50)) to prevent all instances retrying at the exact same millisecond.

### Q42: What is the "Sidecar Pattern"?
**Concept:**
Deploy a helper container alongside the main application container (same Pod in K8s).
**Usage:**
*   **Service Mesh (Istio):** Sidecar handles mTLS, tracing, metrics.
*   **Polyglot:** Main app in NodeJS (which lacks Eureka support). Sidecar (Java) handles Eureka registration and proxies traffic to localhost NodeJS.

### Q43: What is the "Ambassador Pattern"?
**Concept:**
A type of Sidecar that handles outgoing connectivity logic.
**Usage:**
*   App connects to `localhost:port`.
*   Ambassador proxy receives request and handles:
    *   Smart routing.
    *   Circuit breaking.
    *   Auth tokens injection.

### Q44: What is the "Strangler Fig Pattern"?
**Migration Strategy (Monolith -> Microservices):**
1.  Put a Proxy (Gateway) in front of Monolith.
2.  Build new Microservice for one feature (e.g., Search).
3.  Update Proxy to route `/search` to Microservice, everything else to Monolith.
4.  Repeat until Monolith is strangled (empty).

---

## <a name="event-driven"></a>Event-Driven Architecture (Q71-100)

### Q71: Kafka Consumer Groups & Rebalancing.
**Consumer Group:**
A set of consumers working together to consume a topic.
**Partitioning:**
Topic has 10 partitions. Group has 5 consumers. Each consumer gets 2 partitions.
**Rebalancing:**
If a consumer dies, Kafka re-assigns its 2 partitions to the remaining 4 consumers.
**Stop-the-World:** During rebalance, consumption pauses.

### Q72: How to guarantee Message Ordering in Kafka?
**Rule:** Kafka guarantees ordering **only within a Partition**.
**Implementation:**
To ensure all events for `Order-123` are ordered:
*   Use `Order-ID` as the **Partition Key**.
*   All messages with same Key go to the same Partition.
*   Consumer reads partition sequentially.

### Q73: RabbitMQ vs Kafka?
**Comparison:**
1.  **Model:** RabbitMQ (Queue/Push) vs Kafka (Log/Pull).
2.  **Persistence:** RabbitMQ removes message after ack. Kafka retains message (configurable retention).
3.  **Use Case:**
    *   **Rabbit:** Complex routing (Exchange/RoutingKey), Job Queues.
    *   **Kafka:** High throughput streams, Event Sourcing, Log Aggregation.

### Q74: What is "Dead Letter Queue" (DLQ)?
**Concept:**
A queue to store messages that cannot be processed (poison messages).
**Workflow:**
1.  Consumer reads message. Fails processing (Bug/Bad Data).
2.  Retries 3 times. Fails.
3.  Consumer publishes message to `DLQ-Topic`.
4.  Commits offset (moves on).
5.  **Recovery:** Developers inspect DLQ, fix bug, and replay messages.

### Q75: How to implement "Transactional Outbox" with CDC?
**Deep Dive:**
1.  **App:** Writes to `Outbox` table.
2.  **CDC (Change Data Capture):** Tool like **Debezium** connects to DB Transaction Log (MySQL Binlog / Postgres WAL).
3.  **Debezium:** Detects the INSERT to Outbox.
4.  **Kafka Connect:** Publishes the row to Kafka.
**Benefit:** Zero code in application to talk to Kafka. Highly reliable.

### Q76: What is "Eventual Consistency"?
**Definition:**
If no new updates are made to a given data item, eventually all accesses to that item will return the last updated value.
**Trade-off:**
During the window of inconsistency (milliseconds to seconds), users might see stale data.
**Mitigation:**
*   UI tricks (Optimistic UI updates).
*   Version checks.

### Q77: What is "Leader Election" in Distributed Systems?
**Scenario:**
You have 5 instances of `SchedulerService`. You want only ONE to run the job.
**Implementation:**
1.  **Database:** `INSERT IGNORE INTO lock_table (id) VALUES (1)`. Only one succeeds.
2.  **Redis:** `SET resource ID NX EX 30`. (Redlock).
3.  **Kubernetes:** Use K8s native Leader Election API (based on ConfigMaps/Leases).

