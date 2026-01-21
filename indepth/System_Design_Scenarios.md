# System Design Scenarios (In-Depth Guide)

This document provides end-to-end design solutions for common system design interview questions.

## Table of Contents
1.  [Real-Time Chat (WhatsApp)](#chat)
2.  [Notification Service](#notification)
3.  [URL Shortener (TinyURL)](#tinyurl)
4.  [High Scale System (1M Users)](#high-scale)
5.  [Web Crawler](#crawler)
6.  [Flash Sales System](#flash-sale)
7.  [Rate Limiter](#rate-limiter)
8.  [Distributed Leaderboard](#leaderboard)
9.  [Typeahead / Autocomplete](#typeahead)
10. [Video Streaming (Netflix)](#netflix)

---

## <a name="chat"></a>1. Design Real-Time Chat (WhatsApp)

### Requirements
*   **Functional:** 1-on-1 chat, Group chat, Sent/Delivered/Read receipts, Online status.
*   **Non-Functional:** Low latency, Persistent storage, High availability.

### Architecture
1.  **Connection:** **WebSockets**. Keep persistent connection.
2.  **Protocol:** MQTT (lightweight) or custom JSON.
3.  **Service Discovery:** Zookeeper/Eureka to map `User` -> `GatewayServer`.

### Key Components
*   **Chat Server (Stateful):** Holds WebSocket connections.
*   **Presence Service:** Redis. `SET user:123:status "online" TTL=30s`. Client heartbeats.
*   **Message Store:**
    *   **Hot Data (Recent):** Cassandra/HBase (Write heavy). Key: `ChatID`, SortKey: `Timestamp`.
    *   **Cold Data:** Archive to S3.

### Workflow (Message Sending)
1.  User A sends message to Chat Server 1.
2.  Server 1 saves to Cassandra.
3.  Server 1 checks Redis: "Where is User B?".
4.  Redis says: "User B is on Chat Server 5".
5.  Server 1 forwards message to Server 5 (via Kafka or RPC).
6.  Server 5 pushes to User B via WebSocket.
7.  **Receipts:** User B sends "Ack". Server 5 updates status in DB and notifies User A.

---

## <a name="notification"></a>2. Design Notification Service

### Requirements
*   Send Email, SMS, Push Notifications.
*   Prevent spamming.
*   Prioritize OTPs over Marketing.

### Architecture
1.  **Front-End:** API Gateway (`/send`).
2.  **Validator:** Checks payload size, user preferences (Opt-out).
3.  **Rate Limiter:** Token bucket per user.
4.  **Prioritization (Queues):**
    *   **High Priority (Kafka Topic):** OTP, Security Alerts.
    *   **Low Priority (Kafka Topic):** Marketing.
5.  **Workers:**
    *   Consume from Kafka.
    *   Route to provider (Twilio, SendGrid, FCM).
6.  **Idempotency:** Track `notification_id` in Redis to prevent duplicates.

---

## <a name="tinyurl"></a>3. Design URL Shortener (TinyURL)

### Requirements
*   Shorten arbitrary URL to 7 chars.
*   Redirect short URL to original.
*   High Read Ratio (100:1).

### Core Logic (Base62)
*   Characters: `A-Z, a-z, 0-9` = 62 chars.
*   Length 7: `62^7` = 3.5 Trillion combinations. Enough.
*   **Algorithm:**
    1.  Generate unique long ID (Snowflake/Database Sequence).
    2.  Convert ID to Base62. `12345` -> `d9a`.
    3.  Save `{id: "d9a", long_url: "..."}`.

### Storage & Scaling
*   **DB:** NoSQL (DynamoDB/Cassandra) or RDBMS.
*   **Caching:** Critical. Redis checks `d9a` -> `long_url`. 99% hits served from Cache.
*   **Collision:** Pre-generation service (KGS) creates keys in advance.

---

## <a name="high-scale"></a>4. Design High Scale System (1M Users)

### Layers of Scaling
1.  **Edge:** CDN (Cloudflare) caches HTML/JS/Images. WAF filters bad traffic.
2.  **Load Balancer:** Layer 7 ALB. Terminates SSL. Routes to services.
3.  **App Layer:** Stateless Microservices (K8s). Scale horizontally (HPA).
4.  **Cache:** Redis Cluster. Distributed Caching.
5.  **Database:**
    *   **Reads:** Read Replicas (Master-Slave).
    *   **Writes:** Sharding (Partition by UserID).
6.  **Async:** Offload PDF generation, Emails, Reports to Kafka.

---

## <a name="crawler"></a>5. Design Web Crawler

### Components
1.  **Seed URLs:** Starting point.
2.  **URL Frontier:** A Prioritized Queue (Kafka/Redis) storing URLs to visit.
3.  **DNS Resolver:** Custom DNS cache to speed up resolution.
4.  **HTML Downloader:** Fetches content.
5.  **Deduplication:**
    *   **Content:** MD5 hash of content.
    *   **URL:** **Bloom Filter** (Space efficient check if URL visited).
6.  **Storage:** BigTable / HBase.

### Politeness
*   Respect `robots.txt`.
*   Rate limit per domain. (Map: `Domain -> LastVisitTime`).

---

## <a name="flash-sale"></a>6. Design Flash Sales System

### Challenges
*   Inventory: 100 items.
*   Traffic: 1 Million users at 12:00:00.
*   Goal: Don't crash DB. Don't oversell.

### Strategy
1.  **Static Content:** Serve page from CDN.
2.  **Rate Limiting:** Aggressive IP-based blocking at Gateway.
3.  **Queueing:**
    *   User clicks "Buy".
    *   Request -> Kafka.
    *   User sees "You are in queue...".
4.  **Inventory Processor (Single Thread):**
    *   Reads from Kafka.
    *   Decrements **Redis Counter**. `DECR stock`.
    *   If >= 0: Success. Trigger Payment.
    *   If < 0: Fail. Notify user.
5.  **Post-Processing:** Async sync Redis stock to SQL DB.

---

## <a name="rate-limiter"></a>7. Design Rate Limiter

### Algorithms
1.  **Token Bucket:** Efficient. Bursty traffic allowed.
2.  **Leaky Bucket:** Constant rate output.
3.  **Sliding Window Log:** Accurate but expensive (stores timestamps).

### Distributed Implementation
*   **Redis + Lua:**
    *   Key: `user_id:minute_timestamp`.
    *   `INCR key`.
    *   `EXPIRE key 60`.
    *   If result > Limit, block.
*   **Race Conditions:** Lua script ensures atomicity.

---

## <a name="leaderboard"></a>8. Design Distributed Leaderboard

### Requirements
*   Real-time rank updates.
*   Top 10 players.
*   Millions of players.

### Data Structure
*   **Redis Sorted Set (ZSET):**
    *   Internally uses **Skip List**.
    *   `O(log N)` for insert and rank lookup.

### Operations
1.  **Update Score:** `ZINCRBY leaderboard 50 "user1"`.
2.  **Get Rank:** `ZREVRANK leaderboard "user1"`.
3.  **Get Top 10:** `ZREVRANGE leaderboard 0 9`.

### Scaling
If ZSET too big for one Redis node:
*   **Partitioning:** Shard by Score Range (Tier 1: 0-1000, Tier 2: 1000-2000).
*   Or Shard by User ID (and aggregate Top K from all shards - scatter gather).

---

## <a name="typeahead"></a>9. Design Typeahead / Autocomplete

### Data Structure
*   **Trie (Prefix Tree):** Efficient for prefix lookups.
*   Each node stores Top 5 popular searches for that prefix.

### Architecture
1.  **Ingestion:** Log search queries -> Kafka -> Aggregator (Hadoop/Spark) -> Update Trie.
2.  **Storage:** Serialize Trie to document store (Mongo) or Key-Value.
3.  **Serving:** In-memory Cache (Redis) stores results for frequent prefixes (`"jav" -> ["java", "javascript"]`).
4.  **Sampling:** Only update Trie for significant query volume changes (not every single query).

---

## <a name="netflix"></a>10. Design Video Streaming (Netflix)

### Architecture
1.  **Upload:** Partner uploads raw video (4K, MKV).
2.  **Processing (Encoding):**
    *   Split video into 5-minute chunks.
    *   Encode each chunk into multiple resolutions (1080p, 720p, 480p) and bitrates.
    *   Workers scale horizontally.
3.  **Storage:** S3 (Blob storage).
4.  **Distribution (CDN):**
    *   Push chunks to Edge Servers globally (Open Connect).
5.  **Playback (Client):**
    *   **Adaptive Bitrate Streaming (HLS/DASH):**
    *   Client detects bandwidth.
    *   Downloads appropriate chunk (e.g., 1080p chunk if fast, 480p if slow).

