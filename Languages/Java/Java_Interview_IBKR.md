# Java – In-Depth Interview Notes (IBKR JD)

## Table of Contents
1. [OOP Fundamentals](#1-oop-fundamentals)
2. [Collections & Generics](#2-collections--generics)
3. [Multithreading & Concurrency](#3-multithreading--concurrency)
4. [JVM Internals & Performance](#4-jvm-internals--performance)
5. [JDBC & Database Connectivity](#5-jdbc--database-connectivity)
6. [Java Streams & Functional Programming](#6-java-streams--functional-programming)
7. [Design Patterns](#7-design-patterns)
8. [Exception Handling](#8-exception-handling)
9. [Memory Management & GC](#9-memory-management--gc)
10. [Web Application Development Basics](#10-web-application-development-basics)

---

## 1. OOP Fundamentals

### Four Pillars
| Pillar | Definition | Example |
|---|---|---|
| **Encapsulation** | Bundling data + methods; restrict direct access | Private fields + getters/setters |
| **Abstraction** | Hide implementation details, expose interface | Abstract classes, Interfaces |
| **Inheritance** | Deriving new class from existing | `class Dog extends Animal` |
| **Polymorphism** | Same method, different behavior | Method overriding, overloading |

### Interface vs Abstract Class
```java
// Interface – pure contract (Java 8+ allows default methods)
interface Tradable {
    void execute();
    default void log() { System.out.println("Trade executed"); }
}

// Abstract Class – partial implementation
abstract class Order {
    protected String symbol;
    public abstract double getPrice(); // must override
    public void print() { System.out.println(symbol); } // concrete
}
```

> **Interview Tip:** "When to use interface vs abstract class?"
> - Interface: Unrelated classes sharing a contract (e.g., `Serializable`, `Runnable`)
> - Abstract class: Shared base code + IS-A relationship

---

## 2. Collections & Generics

### Key Interfaces
```
Collection
├── List (ordered, duplicates allowed) → ArrayList, LinkedList, Vector
├── Set (no duplicates) → HashSet, LinkedHashSet, TreeSet
└── Queue → LinkedList, PriorityQueue, ArrayDeque

Map (key-value) → HashMap, LinkedHashMap, TreeMap, ConcurrentHashMap
```

### When to Use What
| Need | Use |
|---|---|
| Fast random access | `ArrayList` |
| Fast insert/delete at head/tail | `LinkedList` |
| Unique elements, fast lookup | `HashSet` |
| Sorted unique elements | `TreeSet` |
| Thread-safe map | `ConcurrentHashMap` |
| Priority scheduling | `PriorityQueue` |

### ConcurrentHashMap vs HashMap
```java
// HashMap – NOT thread-safe
Map<String, Integer> map = new HashMap<>();

// ConcurrentHashMap – thread-safe, segment locking (Java 8: CAS + bin-level locking)
Map<String, Integer> concurrentMap = new ConcurrentHashMap<>();
concurrentMap.put("AAPL", 150);
concurrentMap.computeIfAbsent("GOOGL", k -> fetchPrice(k));
```

### Generics – Bounded Type Parameters
```java
// Upper bounded wildcard – read only
public double sumPrices(List<? extends Number> prices) {
    return prices.stream().mapToDouble(Number::doubleValue).sum();
}

// Generic class
public class Pair<A, B> {
    private A first; private B second;
    public Pair(A first, B second) { this.first = first; this.second = second; }
}
```

---

## 3. Multithreading & Concurrency

### Thread Creation
```java
// 1. Extend Thread
class TradeWorker extends Thread {
    public void run() { System.out.println("Processing trade..."); }
}

// 2. Implement Runnable (preferred – avoids single inheritance limitation)
Thread t = new Thread(() -> System.out.println("Runnable trade"));

// 3. ExecutorService (production use)
ExecutorService executor = Executors.newFixedThreadPool(10);
executor.submit(() -> processTrade("AAPL", 100));
executor.shutdown();
```

### Synchronization
```java
class OrderBook {
    private final List<Order> orders = new ArrayList<>();

    // synchronized method
    public synchronized void addOrder(Order o) { orders.add(o); }

    // synchronized block (finer granularity)
    public void removeOrder(Order o) {
        synchronized(orders) { orders.remove(o); }
    }
}
```

### volatile vs synchronized
```java
// volatile: ensures visibility across threads, NO atomicity
volatile boolean isMarketOpen = true;

// AtomicInteger: lock-free, CAS-based
AtomicInteger tradeCount = new AtomicInteger(0);
tradeCount.incrementAndGet(); // thread-safe
```

### CompletableFuture (Async Programming)
```java
CompletableFuture<Double> priceFuture = CompletableFuture
    .supplyAsync(() -> fetchMarketPrice("AAPL"))
    .thenApply(price -> price * 1.01) // add 1% markup
    .exceptionally(ex -> { System.err.println(ex); return 0.0; });

Double price = priceFuture.get(); // blocking
```

### Common Concurrency Issues
| Issue | Problem | Solution |
|---|---|---|
| **Race condition** | Multiple threads modify shared data | `synchronized`, `Lock` |
| **Deadlock** | Two threads wait for each other | Lock ordering, `tryLock()` |
| **Starvation** | Thread never gets CPU | Fair locks (`ReentrantLock(true)`) |
| **Livelock** | Threads keep responding to each other | Randomized retry |

---

## 4. JVM Internals & Performance

### JVM Architecture
```
Source (.java) → javac → Bytecode (.class) → JVM
                                                ├── Class Loader
                                                ├── JIT Compiler (hot spots → native)
                                                ├── Heap (objects)
                                                ├── Stack (method frames, local vars)
                                                ├── Method Area (class metadata)
                                                └── GC
```

### Heap Regions
```
Young Generation → Eden → Survivor S0/S1 (Minor GC)
Old Generation (Tenured) → long-lived objects (Major GC)
Metaspace (Java 8+) → class metadata (replaces PermGen)
```

### Performance Tips for Large Systems (IBKR-relevant)
```java
// 1. Use StringBuilder for string concatenation in loops
StringBuilder sb = new StringBuilder();
for (Trade t : trades) sb.append(t.getId()).append(",");

// 2. Prefer primitives over wrapper types (avoid autoboxing overhead)
int count = 0; // NOT Integer count = 0;

// 3. Use object pools for frequently created objects
// Apache Commons Pool2
GenericObjectPool<Connection> pool = new GenericObjectPool<>(factory);

// 4. Profile with JFR (Java Flight Recorder)
// java -XX:+FlightRecorder -XX:StartFlightRecording=duration=60s,filename=recording.jfr MyApp
```

---

## 5. JDBC & Database Connectivity

```java
// Standard JDBC flow
public class OracleConnector {
    private static final String URL = "jdbc:oracle:thin:@host:1521:orcl";
    private static final String USER = "ibkr_user";
    private static final String PASS = "secret";

    public static void main(String[] args) throws Exception {
        Class.forName("oracle.jdbc.OracleDriver");
        try (Connection conn = DriverManager.getConnection(URL, USER, PASS);
             PreparedStatement ps = conn.prepareStatement(
                 "SELECT symbol, price FROM market_data WHERE exchange = ?")) {
            ps.setString(1, "NYSE");
            ResultSet rs = ps.executeQuery();
            while (rs.next()) {
                System.out.printf("%s: %.2f%n", rs.getString("symbol"), rs.getDouble("price"));
            }
        } // auto-close
    }
}
```

### PreparedStatement vs Statement
| | `Statement` | `PreparedStatement` |
|---|---|---|
| SQL Injection | Vulnerable | Safe (parameterized) |
| Performance | Compiles each time | Pre-compiled |
| Use case | Static SQL | Dynamic with params |

### Connection Pooling (HikariCP)
```java
HikariConfig config = new HikariConfig();
config.setJdbcUrl("jdbc:oracle:thin:@host:1521:orcl");
config.setUsername("user"); config.setPassword("pass");
config.setMaximumPoolSize(20);
config.setMinimumIdle(5);
HikariDataSource ds = new HikariDataSource(config);
// Use ds.getConnection() instead of DriverManager
```

---

## 6. Java Streams & Functional Programming

```java
List<Trade> trades = getTradeData();

// Filter, transform, collect
List<String> highValueSymbols = trades.stream()
    .filter(t -> t.getValue() > 1_000_000)
    .sorted(Comparator.comparingDouble(Trade::getValue).reversed())
    .map(Trade::getSymbol)
    .distinct()
    .collect(Collectors.toList());

// Grouping (very useful for financial data aggregation)
Map<String, DoubleSummaryStatistics> statsBySymbol = trades.stream()
    .collect(Collectors.groupingBy(
        Trade::getSymbol,
        Collectors.summarizingDouble(Trade::getValue)
    ));
// statsBySymbol.get("AAPL").getAverage()

// Parallel Stream (use carefully – overhead for small collections)
double totalVolume = trades.parallelStream()
    .mapToDouble(Trade::getVolume)
    .sum();
```

---

## 7. Design Patterns

### Singleton (DB Connection Pool)
```java
public class DatabasePool {
    private static volatile DatabasePool instance;
    private DatabasePool() {}
    public static DatabasePool getInstance() {
        if (instance == null) {
            synchronized (DatabasePool.class) {
                if (instance == null) instance = new DatabasePool();
            }
        }
        return instance;
    }
}
```

### Factory (Order Types)
```java
interface Order { void execute(); }
class MarketOrder implements Order { public void execute() { /* market exec */ } }
class LimitOrder  implements Order { public void execute() { /* limit exec  */ } }

class OrderFactory {
    public static Order create(String type) {
        return switch (type) {
            case "MARKET" -> new MarketOrder();
            case "LIMIT"  -> new LimitOrder();
            default -> throw new IllegalArgumentException("Unknown: " + type);
        };
    }
}
```

### Observer (Market Data Feed)
```java
interface MarketListener { void onPriceUpdate(String symbol, double price); }

class MarketFeed {
    private List<MarketListener> listeners = new ArrayList<>();
    public void subscribe(MarketListener l) { listeners.add(l); }
    public void publish(String symbol, double price) {
        listeners.forEach(l -> l.onPriceUpdate(symbol, price));
    }
}
```

---

## 8. Exception Handling

```java
// Custom exception hierarchy
public class TradeException extends RuntimeException {
    private final String tradeId;
    public TradeException(String tradeId, String msg, Throwable cause) {
        super(msg, cause);
        this.tradeId = tradeId;
    }
}

// Try-with-resources (crucial for DB/file operations)
try (Connection conn = ds.getConnection();
     PreparedStatement ps = conn.prepareStatement(sql)) {
    // ...
} catch (TradeException e) {
    log.error("Trade {} failed: {}", e.tradeId, e.getMessage());
    throw e; // rethrow for upstream handling
} catch (SQLException e) {
    throw new TradeException(tradeId, "DB error", e);
} finally {
    // cleanup if not using try-with-resources
}
```

---

## 9. Memory Management & GC

### GC Algorithms
| GC | Best For | Flag |
|---|---|---|
| Serial | Small heaps, single-threaded | `-XX:+UseSerialGC` |
| Parallel | Throughput-focused batch | `-XX:+UseParallelGC` |
| G1 | Balanced latency/throughput (default Java 9+) | `-XX:+UseG1GC` |
| ZGC | Ultra-low latency (<10ms pauses) | `-XX:+UseZGC` |

```bash
# Tuning flags for large data systems (IBKR-relevant)
java -Xms4g -Xmx8g \
     -XX:+UseG1GC \
     -XX:MaxGCPauseMillis=200 \
     -XX:+HeapDumpOnOutOfMemoryError \
     -XX:HeapDumpPath=/logs/heap.hprof \
     MyLargeSystemApp
```

---

## 10. Web Application Development Basics

### REST with Spring Boot (quick reference)
```java
@RestController
@RequestMapping("/api/trades")
public class TradeController {

    @GetMapping("/{id}")
    public ResponseEntity<Trade> getTrade(@PathVariable String id) {
        return tradeService.findById(id)
            .map(ResponseEntity::ok)
            .orElse(ResponseEntity.notFound().build());
    }

    @PostMapping
    public ResponseEntity<Trade> createTrade(@RequestBody @Valid TradeRequest req) {
        Trade trade = tradeService.create(req);
        return ResponseEntity.status(HttpStatus.CREATED).body(trade);
    }
}
```

---

## Common Java Interview Questions for IBKR

| Question | Key Answer Points |
|---|---|
| Difference between `==` and `.equals()`? | `==` compares references; `.equals()` compares content |
| What is `final` keyword? | Final variable: constant, method: can't override, class: can't extend |
| What's a memory leak in Java? | GC can't collect referenced-but-unused objects (e.g., static collections) |
| How does `HashMap` work internally? | Hash table with array of linked lists/trees; `hashCode()` + `equals()` |
| What's the difference between `Callable` and `Runnable`? | `Callable` returns a result and can throw checked exceptions |
| Explain `volatile`? | Ensures visibility of variable changes across threads; no atomicity |
| What's a `ThreadLocal`? | Per-thread value; useful for DB connections, user context |
| What is the difference between `List.of()` and `new ArrayList()`? | `List.of()` is immutable; `ArrayList` is mutable |
