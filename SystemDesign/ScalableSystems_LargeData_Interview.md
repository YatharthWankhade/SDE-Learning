# Scalable Systems Design – Interview Notes (IBKR JD)

## Table of Contents
1. [Scalability Fundamentals](#1-scalability-fundamentals)
2. [Database Scalability](#2-database-scalability)
3. [Caching Strategies](#3-caching-strategies)
4. [Message Queues & Async Processing](#4-message-queues--async-processing)
5. [Load Balancing](#5-load-balancing)
6. [Designing for Massive Data: IBKR Scenarios](#6-designing-for-massive-data-ibkr-scenarios)
7. [Monitoring & Observability](#7-monitoring--observability)
8. [Common Interview Questions](#8-common-interview-questions)

---

## 1. Scalability Fundamentals

### Horizontal vs Vertical Scaling
```
Vertical (Scale Up):
  Add more CPU/RAM to single machine
  ✓ Simple – no code changes
  ✗ Physical limits; single point of failure; expensive

Horizontal (Scale Out):
  Add more machines
  ✓ Theoretically unlimited; fault-tolerant
  ✗ Complexity: state management, consistency, network

At IBKR scale → horizontal scaling + stateless application tier
```

### CAP Theorem (IBKR Systems)
```
Distributed system can guarantee only 2 of 3:
  C = Consistency   (all nodes see same data at same time)
  A = Availability  (system always responds)
  P = Partition Tolerance (works even if network splits)

Since network partitions always happen in distributed systems:
  CP (Consistency + Partition):
    → Choose this for: Oracle databases, ZooKeeper
    → Risk: may be unavailable during network splits
    → IBKR reference data needs consistency

  AP (Availability + Partition):
    → Choose this for: DNS, Cassandra, DynamoDB, Kafka consumers
    → Risk: may serve stale data

"IBKR's trade matching must be CP (accuracy over availability)"
```

### PACELC (Extension of CAP)
```
Even without partition (normal operation):
  Trade-off between Latency and Consistency

Low latency → may sacrifice consistency (eventual)
Strong consistency → higher latency

Oracle with strong isolation → higher latency but consistent
Kafka read → low latency but eventually consistent
```

---

## 2. Database Scalability

### Read Replicas (Oracle Data Guard)
```
Primary (writer) → Standby replica (reader)
  ↓ redo logs sync
  
Application:
  - All writes → Primary
  - Read-heavy queries, reports → Standby
  - Reduces load on primary by ~60%

Oracle Active Data Guard → allows reads on standby
```

### Database Sharding
```
Split data across multiple DB instances by shard key

Example: Trade data sharded by account_id
  Shard 0: account_id % 4 == 0
  Shard 1: account_id % 4 == 1
  Shard 2: account_id % 4 == 2
  Shard 3: account_id % 4 == 3

Advantages:
  Write throughput scales linearly with shards
  Each shard has smaller dataset → faster queries

Disadvantages:
  Cross-shard queries are complex/expensive
  Rebalancing shards is hard
  Application must know routing logic
```

### Connection Pool Sizing Formula
```
pool_size = (core_count * 2) + effective_spindle_count

For 8-core server with SSD (1 effective spindle):
  pool_size = 8 * 2 + 1 = 17

HikariCP recommends: pool_size = (2 * num_cores) + 1
Too many connections → context switching overhead (worse than too few)
```

---

## 3. Caching Strategies

### Cache Patterns
```
Cache-Aside (Lazy Loading):           Cache Invalidation Strategies:
  Application checks cache first        1. TTL (time-to-live): cache expires after N seconds
  Miss → load from DB → store in cache  2. Event-driven: update cache when DB changes
  ✓ Only cache what's needed            3. Cache-aside: delete cache on update, reload on next miss
  ✗ Cold start; cache stampede risk     4. Write-Through: write to cache AND DB simultaneously

Write-Through:
  Write to cache and DB simultaneously
  ✓ Cache always fresh
  ✗ Write latency for non-cached keys
```

### Redis for IBKR Use Cases
```java
// Connect to Redis
Jedis jedis = new Jedis("redis-host", 6379);
// Or JedisPool for thread-safety

// Cache instrument reference data
String key = "instrument:" + isin;
String cached = jedis.get(key);
if (cached == null) {
    Instrument inst = db.findByIsin(isin);      // DB hit
    jedis.setex(key, 3600, toJson(inst));       // cache for 1 hour
    return inst;
}
return fromJson(cached, Instrument.class);

// Session/rate limiting
jedis.incr("ratelimit:" + userId);
jedis.expire("ratelimit:" + userId, 60);  // reset each minute
int count = Integer.parseInt(jedis.get("ratelimit:" + userId));
if (count > 100) throw new RateLimitException("Too many requests");

// Sorted set for leaderboard / ranking
jedis.zadd("portfolio_values", portfolioValue, accountId);
List<String> top10 = jedis.zrevrange("portfolio_values", 0, 9);
```

### Cache Stampede Prevention
```java
// Problem: cache expires → 10,000 threads simultaneously query DB
// Solution 1: Jitter (random TTL)
int ttl = 3600 + (int)(Math.random() * 600);  // 60-70 minutes
jedis.setex(key, ttl, value);

// Solution 2: Mutex (only one thread loads from DB)
if (jedis.set("lock:" + key, "1", SetParams.setParams().nx().ex(5)) != null) {
    // This thread got the lock – load from DB
    String value = db.load(key);
    jedis.setex(key, ttl, value);
    jedis.del("lock:" + key);
}
```

---

## 4. Message Queues & Async Processing

### When to Use Async
```
Synchronous:  Client → Server → DB → Response  (sequential, blocking)
Asynchronous: Client → Queue  ← Worker → DB (decoupled, non-blocking)

Use async when:
  1. Operation is slow (email, report generation, batch DB load)
  2. Need to decouple producer from consumer speed
  3. Need to absorb traffic spikes (queue acts as buffer)
  4. Need retry on failure

IBKR example:
  Trade Execution → Kafka → Risk System (async)
                         → Reporting System (async)
                         → Client Notification (async)
```

### Queue vs Topic
```
Queue (RabbitMQ, JMS):
  One message → one consumer (competing consumers)
  Message deleted after consumption
  Good for: work queues, task distribution

Topic (Kafka):
  One message → multiple consumer groups (fan-out)
  Message retained for configurable period
  Good for: event streaming, audit trails, replay
```

---

## 5. Load Balancing

### Algorithms
| Algorithm | Description | Best For |
|---|---|---|
| **Round Robin** | Requests distributed sequentially | Homogeneous servers, stateless apps |
| **Least Connections** | Send to server with fewest connections | Variable request lengths |
| **IP Hash** | Same client → same server | Session affinity needed |
| **Weighted** | More requests to more powerful servers | Heterogeneous server farm |
| **Random** | Random selection | Simple stateless |

### Layer 4 vs Layer 7 Load Balancer
```
L4 (Transport Layer – TCP/UDP):
  Routes based on IP and port
  Faster, less overhead
  Can't see HTTP headers/content
  Example: AWS NLB

L7 (Application Layer – HTTP):  
  Routes based on URL, headers, cookies
  Can do SSL termination, content-based routing
  Slightly more overhead
  Example: AWS ALB, Nginx
```

---

## 6. Designing for Massive Data: IBKR Scenarios

### Scenario 1: Market Data Feed (1M ticks/second)
```
Design:
  Exchanges → Market Data Feed Handler → Kafka (partitioned by symbol)
                                       ↓
                              Consumer Groups:
                              - Trade Systems
                              - Risk Calculation
                              - Price Display
                              - EOD Pricing

Key choices:
  - Kafka: handles 1M+ msgs/sec, replay, multiple consumers
  - Partitioned by symbol → ordering guarantee per symbol
  - Consumer groups → independent scaling of each downstream system
  - Avro serialization → schema evolution, compact binary format
  - Monitoring: Prometheus + Grafana for consumer lag
```

### Scenario 2: Reference Database – 1M+ Instruments
```
Design:
  External Data Vendors → ETL (Java/Python) → Oracle Staging → Validated → Production
                                                     ↓
                                           Data Quality Checks:
                                           - ISIN format validation
                                           - Cross-reference validation
                                           - Duplicate detection
                                           - Historical consistency

Key Oracle Objects:
  - Partitioned tables by instrument type or exchange
  - Materialized views for frequently joined lookups
  - PL/SQL packages for ETL pipeline logic
  - Oracle streams / Kafka for change propagation

Cache layer (Redis):
  - ISIN → instrument details (99% of lookups; rarely changes)
  - TTL = 24h with event-driven invalidation on updates
```

### Scenario 3: Trade Data at Scale (Historical + Real-time)
```
Hot Data (recent 3 months) → Oracle (indexed, fast queries)
Warm Data (3 months - 2 years) → Oracle partitioned/compressed
Cold Data (2+ years) → Oracle export to HDFS / S3 / Oracle Exadata
                       → Query via Spark, Athena, or Oracle external tables

Oracle Partitioning strategy:
  PARTITION BY RANGE (trade_date)   → yearly/quarterly
  SUBPARTITION BY HASH (account_id) → even distribution

Indexes:
  (trade_date, symbol) → most common query patterns
  (account_id, trade_date) → account statement queries
```

---

## 7. Monitoring & Observability

### Three Pillars
```
Metrics    → numeric time-series data (latency, throughput, error rate)
             Tools: Prometheus, Grafana, Datadog
Logs       → discrete events with context
             Tools: ELK Stack (Elasticsearch, Logstash, Kibana)
Traces     → request path through distributed services
             Tools: Jaeger, Zipkin, AWS X-Ray
```

### Key Metrics for IBKR Systems
```
Business:
  - Trade throughput (trades/second)
  - Order rejection rate
  - Settlement failure rate
  - Market data delay (latency from exchange to application)

Technical:
  - DB query latency (p50, p95, p99)
  - Kafka consumer lag (alert if > 10,000 messages)
  - JVM GC pause time (alert if > 500ms)
  - Connection pool usage % (alert if > 80%)
  - Error rate (alert if > 0.1%)
```

### Alerting Philosophy (SLO/SLI)
```
SLI (Service Level Indicator): actual measurement (99.5% availability)
SLO (Service Level Objective): target (99.9% availability)
SLA (Service Level Agreement): contractual commitment with penalties

Error Budget = 1 - SLO = 0.1% downtime allowed per month
             = ~43 minutes of downtime per month
```

---

## 8. Common Interview Questions

| Question | Key Points |
|---|---|
| **How do you handle 1 million messages per second?** | Kafka with sufficient partitions; consumer group parallelism; async processing; Avro serialization |
| **How to design a system that never loses data?** | Durable message queues (Kafka with replication); synchronous DB writes with acks; idempotent processing; DLQ for failures |
| **How do you scale a read-heavy system?** | Read replicas; caching layer (Redis); CDN; denormalization; materialized views |
| **How to handle cache inconsistency?** | TTL + event-driven invalidation; eventual consistency acceptable for reference data |
| **What is the difference between latency and throughput?** | Latency: time for single request; Throughput: requests per unit time. Can trade off: batch processing ↑ throughput but ↑ latency |
| **How to design for fault tolerance?** | Redundancy (active-active), circuit breakers, retries with exponential backoff, DLQ, graceful degradation |
| **What is a circuit breaker pattern?** | When failures exceed threshold, "open" circuit → fail fast without calling failing service; periodically test if recovered |
| **Database vs NoSQL – when to choose?** | RDBMS: structured data, ACID, complex queries, reporting; NoSQL: scale-out, flexible schema, simple lookups, high write throughput |
