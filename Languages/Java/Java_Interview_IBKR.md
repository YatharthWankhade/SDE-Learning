# Java – In-Depth Interview Notes (IBKR JD)

> **Target level:** 2+ years | Focus: backend Java, concurrency, JVM, system design patterns, IBKR financial context

## Table of Contents
1. [OOP Fundamentals](#1-oop-fundamentals)
2. [Collections & Generics](#2-collections--generics)
3. [Multithreading & Concurrency](#3-multithreading--concurrency)
4. [Java Memory Model (JMM)](#4-java-memory-model-jmm)
5. [JVM Internals & Performance](#5-jvm-internals--performance)
6. [JDBC & Database Connectivity](#6-jdbc--database-connectivity)
7. [Java Streams & Functional Programming](#7-java-streams--functional-programming)
8. [Design Patterns in Trading Systems](#8-design-patterns-in-trading-systems)
9. [Exception Handling](#9-exception-handling)
10. [Memory Management & GC](#10-memory-management--gc)
11. [Spring Framework Essentials](#11-spring-framework-essentials)
12. [Kafka Integration in Java](#12-kafka-integration-in-java)
13. [Java 11–17+ Modern Features](#13-java-1117-modern-features)
14. [Common Interview Questions](#14-common-interview-questions)

---

## 1. OOP Fundamentals

### Four Pillars (with Financial Context)

| Pillar | Definition | IBKR Example |
|---|---|---|
| **Encapsulation** | Bundle data + methods; protect internal state | `Order` class: private `price`, `quantity` → only via validated setters |
| **Abstraction** | Hide complexity behind a simple interface | `OrderRouter` interface hides SOR logic; callers just call `route(order)` |
| **Inheritance** | Derive specialized behavior from base class | `class EquityOrder extends Order`, `class FuturesOrder extends Order` |
| **Polymorphism** | Same interface, different implementations | `order.execute()` calls correct implementation at runtime |

### Interface vs Abstract Class – Decision Guide

```java
// Interface – pure contract; multiple inheritance allowed
// Use when: UNRELATED classes share a behavior
interface Executable {
    void execute();
    default void audit() { System.out.println("Audit: " + this.getClass().getSimpleName()); }
}

interface Cancellable {
    boolean cancel(String reason);
}

// Class can implement BOTH
class LimitOrder implements Executable, Cancellable {
    public void execute() { /* exchange routing logic */ }
    public boolean cancel(String reason) { return true; }
}

// Abstract Class – partial implementation with shared state
// Use when: classes share IS-A relationship AND common implementation
abstract class BaseOrder {
    protected final String orderId;
    protected final String symbol;
    protected OrderStatus status = OrderStatus.NEW;

    protected BaseOrder(String orderId, String symbol) {
        this.orderId = orderId;
        this.symbol = symbol;
    }

    // Subclasses MUST implement pricing logic
    public abstract double calculateNotional();

    // Shared logic
    public void updateStatus(OrderStatus newStatus) {
        System.out.printf("Order %s: %s → %s%n", orderId, status, newStatus);
        this.status = newStatus;
    }
}
```

### Key Java Keywords

```java
// final: variable (constant), method (no override), class (no extend)
final class ImmutableTrade { }           // can't extend
final double COMMISSION_RATE = 0.001;    // constant

// static: belongs to class, not instance
static int tradeCount = 0;
static int getCount() { return tradeCount; }

// this vs super
class EquityOrder extends BaseOrder {
    private int quantity;

    public EquityOrder(String id, String symbol, int qty) {
        super(id, symbol);   // call BaseOrder constructor
        this.quantity = qty; // this = current instance
    }

    @Override
    public double calculateNotional() {
        return quantity * getCurrentPrice(); // polymorphic
    }
}

// instanceof (Java 16+ pattern matching)
if (order instanceof LimitOrder lo) {
    System.out.println("Limit price: " + lo.getLimitPrice());
}
```

### SOLID Principles (Critical for IBKR Interview)

```
S – Single Responsibility: Each class has ONE reasons to change
    Good: OrderValidator, OrderRouter, OrderPersister (separate classes)
    Bad: OrderService does validation + routing + DB writes

O – Open/Closed: Open for extension, closed for modification
    Good: Add new OrderType by implementing interface, no existing code changes
    Bad: Giant if/else on order type in core routing logic

L – Liskov Substitution: Subtypes must be substitutable for parent types
    Bad: FuturesOrder overrides cancel() to throw exception (breaks contract)

I – Interface Segregation: Don't force clients to implement unused methods
    Bad: One giant TradingSystem interface with 20 methods
    Good: Executable, Cancellable, Amendable as separate interfaces

D – Dependency Inversion: Depend on abstractions, not concretions
    Bad: TradeService directly creates new OracleRepository()
    Good: TradeService takes Repository interface via constructor injection
```

---

## 2. Collections & Generics

### Collection Hierarchy

```
Iterable
└── Collection
    ├── List (ordered, indexed, duplicates OK)
    │   ├── ArrayList    – O(1) get, O(n) insert middle; resizable array
    │   ├── LinkedList   – O(1) head/tail insert; O(n) indexed get; DoublyLinked
    │   └── CopyOnWriteArrayList – thread-safe; snapshot iterator
    ├── Set (no duplicates)
    │   ├── HashSet      – O(1) avg ops; unordered; uses hashCode()+equals()
    │   ├── LinkedHashSet– preserves insertion order; slightly slower
    │   └── TreeSet      – sorted (natural or Comparator); O(log n) ops
    └── Queue / Deque
        ├── ArrayDeque   – faster than Stack and LinkedList for stack/queue
        ├── PriorityQueue– min-heap; O(log n) add/poll; useful for order book
        └── BlockingQueue – thread-safe producer-consumer (LinkedBlockingQueue, ArrayBlockingQueue)

Map (separate hierarchy)
├── HashMap          – O(1) avg; unordered; null keys allowed
├── LinkedHashMap    – insertion/access-order maintained; LRU cache use case
├── TreeMap          – sorted by key; O(log n); range queries (useful for price levels)
├── ConcurrentHashMap– segment/bin-level locking; no full lock for reads
└── EnumMap          – ultra-fast for enum keys; array-backed
```

### Internal Working of HashMap (Must Know)

```java
// HashMap internals:
// Array of "buckets" (Node<K,V>[]); default capacity 16, load factor 0.75
// Put: key.hashCode() → index → if collision, LinkedList (Java 8+: Tree if > 8 items)
// Get: hashCode() → bucket → equals() to find key

// CRITICAL: Contract between hashCode() and equals()
// If a.equals(b) → a.hashCode() MUST == b.hashCode()
// If hashCode same → equals may or may not be true (collision)

class InstrumentKey {
    private final String isin;
    private final String exchange;

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof InstrumentKey other)) return false;
        return Objects.equals(isin, other.isin) && Objects.equals(exchange, other.exchange);
    }

    @Override
    public int hashCode() {
        return Objects.hash(isin, exchange); // consistent with equals
    }
}

Map<InstrumentKey, Double> prices = new HashMap<>();
```

### Thread-Safe Collections (Critical for Concurrent Systems)

```java
// ConcurrentHashMap – best for high-concurrency read-write maps
Map<String, Double> priceCache = new ConcurrentHashMap<>();

// compute atomically:
priceCache.compute("AAPL", (key, old) -> old == null ? 175.0 : old * 1.01);
priceCache.merge("AAPL", 1.0, Double::sum); // merge with existing

// Collections.synchronizedMap – wraps any map; full lock per operation (slower)
Map<String, Double> synced = Collections.synchronizedMap(new HashMap<>());

// BlockingQueue – producer-consumer handoff (IBKR: order pipeline)
BlockingQueue<Order> orderQueue = new LinkedBlockingQueue<>(1000);
// Producer thread:
orderQueue.put(newOrder);      // blocks if full
// Consumer thread:
Order order = orderQueue.take(); // blocks if empty

// CopyOnWriteArrayList – thread-safe list; good for rare writes, many reads
// e.g., list of market data subscribers
List<MarketListener> listeners = new CopyOnWriteArrayList<>();
```

### Generics – Deep

```java
// Bounded wildcards (PECS: Producer Extends, Consumer Super)
// ? extends T → you can READ T from it (covariant – producer)
public double sumValues(List<? extends Number> numbers) {
    return numbers.stream().mapToDouble(Number::doubleValue).sum();
}

// ? super T → you can WRITE T into it (contravariant – consumer)
public void addDefaults(List<? super Integer> list) {
    list.add(0); // safe to add Integer into List<? super Integer>
}

// Generic method with bounded type
public <T extends Comparable<T>> T max(List<T> list) {
    return list.stream().max(Comparator.naturalOrder()).orElseThrow();
}

// Type erasure: generics are compile-time only; at runtime List<String> == List<Integer>
// Cannot: new T(); instanceof List<String>; T[] arr = new T[10];
```

---

## 3. Multithreading & Concurrency

### Thread Lifecycle

```
NEW → RUNNABLE → (BLOCKED | WAITING | TIMED_WAITING) → TERMINATED

Thread.start()         → NEW → RUNNABLE
Thread.sleep(ms)       → TIMED_WAITING
object.wait()          → WAITING (released lock)
object.notify()        → WAITING → RUNNABLE
synchronized block lock → BLOCKED (waiting for monitor)
```

### Executors Framework (Use This, Not Raw Threads)

```java
// Fixed thread pool – bounded parallelism
ExecutorService pool = Executors.newFixedThreadPool(
    Runtime.getRuntime().availableProcessors()
);

// Scheduled executor – for timed/periodic tasks
ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(2);
scheduler.scheduleAtFixedRate(
    () -> publishMarketData(),
    0, 100, TimeUnit.MILLISECONDS  // every 100ms
);

// ForkJoinPool (Java 7+) – work-stealing; best for divide-and-conquer
ForkJoinPool fjPool = new ForkJoinPool(4);
fjPool.submit(() -> {
    // parallel sorting, recursive tasks
});

// ThreadPoolExecutor – full control (preferred in production)
ThreadPoolExecutor executor = new ThreadPoolExecutor(
    5,                          // corePoolSize
    20,                         // maximumPoolSize
    60L, TimeUnit.SECONDS,      // keepAliveTime
    new ArrayBlockingQueue<>(500), // work queue
    new ThreadPoolExecutor.CallerRunsPolicy() // rejection: caller's thread runs task
);

// Always shutdown gracefully:
executor.shutdown();
if (!executor.awaitTermination(30, TimeUnit.SECONDS)) {
    executor.shutdownNow();
}
```

### Locks – Beyond synchronized

```java
import java.util.concurrent.locks.*;

ReentrantLock lock = new ReentrantLock(true); // fair = FIFO ordering

// Basic usage
lock.lock();
try {
    // critical section
} finally {
    lock.unlock(); // ALWAYS in finally
}

// tryLock – prevents deadlock (attempt with timeout)
if (lock.tryLock(100, TimeUnit.MILLISECONDS)) {
    try { /* work */ }
    finally { lock.unlock(); }
} else {
    // handle lock not acquired
}

// ReadWriteLock – multiple readers OR one writer (good for price cache)
ReadWriteLock rwLock = new ReentrantReadWriteLock();
Lock readLock  = rwLock.readLock();
Lock writeLock = rwLock.writeLock();

// Concurrent readers (O(n) throughput for reads)
readLock.lock();
try { return priceMap.get(symbol); }
finally { readLock.unlock(); }

// Exclusive writer
writeLock.lock();
try { priceMap.put(symbol, newPrice); }
finally { writeLock.unlock(); }

// StampedLock (Java 8+) – optimistic reads, fastest for read-heavy
StampedLock sl = new StampedLock();
long stamp = sl.tryOptimisticRead();
double price = priceCache.get("AAPL"); // read without lock
if (!sl.validate(stamp)) {
    // someone wrote; fall back to read lock
    stamp = sl.readLock();
    try { price = priceCache.get("AAPL"); }
    finally { sl.unlockRead(stamp); }
}
```

### Atomic Classes & CAS

```java
// CAS (Compare-And-Swap) – hardware-level atomic ops; lock-free
AtomicInteger tradeCount = new AtomicInteger(0);
tradeCount.incrementAndGet();                         // thread-safe ++
tradeCount.compareAndSet(expected, newVal);           // CAS

AtomicLong sequenceId = new AtomicLong(0);
long nextId = sequenceId.getAndIncrement();           // unique ID generator

AtomicReference<OrderStatus> statusRef = new AtomicReference<>(OrderStatus.NEW);
statusRef.compareAndSet(OrderStatus.NEW, OrderStatus.SENT); // CAS on object

// LongAdder – preferred over AtomicLong for high-contention counters
LongAdder hitCounter = new LongAdder();
hitCounter.increment();  // striped; less contention
hitCounter.sum();        // read total

// AtomicReferenceArray – thread-safe array (useful for lock-free ring buffers)
AtomicReferenceArray<Order> ring = new AtomicReferenceArray<>(1024);
```

### CompletableFuture – Async Pipelines

```java
// Chain async operations (non-blocking)
CompletableFuture<TradeResult> result = CompletableFuture
    .supplyAsync(() -> validateOrder(order), executor)        // async validation
    .thenApplyAsync(valid -> routeToExchange(valid), executor) // async routing
    .thenApplyAsync(resp -> parseExecReport(resp), executor)  // parse result
    .exceptionally(ex -> {
        log.error("Order failed", ex);
        return TradeResult.failed(ex.getMessage());
    });

// Combine multiple futures (e.g., fetch price + reference data simultaneously)
CompletableFuture<Double> priceFuture = CompletableFuture.supplyAsync(() -> getPrice("AAPL"));
CompletableFuture<Instrument> instrFuture = CompletableFuture.supplyAsync(() -> getInstrument("AAPL"));

CompletableFuture<OrderCheck> combined = priceFuture.thenCombine(instrFuture,
    (price, instr) -> new OrderCheck(price, instr));

// Wait for ALL
CompletableFuture<Void> all = CompletableFuture.allOf(priceFuture, instrFuture);
all.join();

// Wait for FIRST to complete
CompletableFuture<Double> first = CompletableFuture.anyOf(priceFuture, backupPriceFuture)
    .thenApply(o -> (Double) o);
```

### Producer-Consumer Pattern (Common in Trading)

```java
// Classic: BlockingQueue between threads
class OrderProcessor {
    private final BlockingQueue<Order> queue = new LinkedBlockingQueue<>(10_000);

    // Producer (e.g., FIX engine thread)
    public void receiveOrder(Order order) throws InterruptedException {
        queue.put(order); // blocks if queue full → backpressure
    }

    // Consumer (e.g., risk check thread)
    public void processOrders() {
        while (!Thread.currentThread().isInterrupted()) {
            try {
                Order order = queue.take(); // blocks until available
                riskCheck(order);
                route(order);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                break;
            }
        }
    }
}

// LMAX Disruptor pattern (used in HFT at IBKR-like systems):
// Ring buffer → single writer → multiple processors
// No garbage (pre-allocated); no locking; 6M+ ops/sec
```

### Common Concurrency Bugs

```java
// 1. RACE CONDITION – unprotected shared state
int balance = 1000;
// Thread A: balance += 500;  Thread B: balance -= 200;
// Non-atomic read-modify-write → result indeterminate
// Fix: AtomicInteger, synchronized, or Lock

// 2. DEADLOCK
// Thread 1 holds Lock A, waits for Lock B
// Thread 2 holds Lock B, waits for Lock A
// Fix: always acquire locks in SAME ORDER; use tryLock with timeout

// 3. VISIBILITY without volatile
class Broken {
    boolean stop = false;  // may be cached in CPU register
    void run() { while (!stop) { /* loop forever */ } }
    void halt() { stop = true; } // other thread may not see it!
}
// Fix: volatile boolean stop = true;

// 4. DOUBLE-CHECKED LOCKING without volatile
// Fix: use volatile (JMM: volatile write happens-before volatile read)
private static volatile Singleton instance;

// 5. THREAD LEAK – not shutting down ExecutorService
// Fix: always call shutdown(); use try-with-resources pattern

// 6. MISSED SIGNALS – checking condition outside while loop
synchronized (lock) {
    if (!ready) lock.wait();  // BAD: spurious wakeups possible
}
synchronized (lock) {
    while (!ready) lock.wait(); // GOOD: re-check condition after wakeup
}
```

---

## 4. Java Memory Model (JMM)

```
JMM defines how threads interact through memory.
Key concept: HAPPENS-BEFORE relationship

Happens-Before rules:
  1. Program order: actions in a thread in program order
  2. Monitor lock: unlock → subsequent lock of same monitor
  3. volatile: write to volatile → subsequent read of same variable
  4. Thread start: Thread.start() → all actions in started thread
  5. Thread join: all actions in thread → Thread.join() returns

Implications:
  - Without happens-before, JVM + CPU can reorder instructions
  - volatile: guarantees visibility AND prevents reordering (memory fence)
  - synchronized: guarantees both atomicity AND visibility
```

```java
// Happens-before via volatile
class FlagExample {
    volatile boolean ready = false;
    int value = 0;

    void writer() {
        value = 42;         // 1
        ready = true;       // 2 - volatile write, creates happens-before fence
    }

    void reader() {
        while (!ready) {}   // wait for volatile write
        // value is GUARANTEED to be 42 here (happens-before ensured)
        System.out.println(value); // safe: sees 42
    }
}

// Without volatile, reader may see value=0 even if ready=true
// due to CPU/compiler reordering of lines 1 and 2

// Memory fence (low-level concept):
// volatile write = StoreStore + StoreLoad fence
// volatile read  = LoadLoad + LoadStore fence
// → prevents reordering across the fence
```

---

## 5. JVM Internals & Performance

### JVM Architecture

```
Source (.java) → javac → Bytecode (.class) → JVM
                                                ├── ClassLoader
                                                │   ├── Bootstrap (rt.jar)
                                                │   ├── Extension
                                                │   └── Application (classpath)
                                                ├── Bytecode Verifier
                                                ├── Interpreter → JIT Compiler
                                                │   (C1: fast compile; C2: optimized)
                                                │   Tiered compilation: interpret → C1 → C2
                                                ├── Runtime Data Areas
                                                │   ├── Heap (shared; objects)
                                                │   ├── Stack (per-thread; frames)
                                                │   ├── Method Area/Metaspace
                                                │   ├── PC Register (per-thread)
                                                │   └── Native Method Stack
                                                └── GC
```

### Class Loading – How It Works

```java
// ClassLoader delegation model:
// Child CL → Parent CL → Bootstrap CL
// Prevents duplicate class loading; security

// Class initialization order:
// 1. Static fields (in declaration order)
// 2. Static initializer blocks
// 3. Constructor when new instance created

class TradeSystem {
    static final String VERSION;        // 1. static field
    static {
        VERSION = "2.5.1";              // 2. static initializer
        System.out.println("Loaded");
    }
    int id;
    TradeSystem(int id) { this.id = id; } // 3. constructor
}

// Lazy class loading: class loaded on first use (first new, static access, or reflection)
// Class.forName("oracle.jdbc.OracleDriver") → explicitly loads driver class
```

### JVM Performance Tuning (IBKR Relevant)

```bash
# Heap sizing
-Xms4g            # initial heap (set == Xmx to avoid resizing pauses)
-Xmx8g            # maximum heap
-Xss512k          # stack size per thread (reduce if many threads)

# GC selection
-XX:+UseG1GC                  # default Java 9+; balanced for large heaps
-XX:MaxGCPauseMillis=100      # target max pause (G1 tries to meet this)
-XX:+UseZGC                   # Java 11+; sub-millisecond GC; for latency-sensitive
-XX:+UseShenandoahGC          # low-pause; concurrent; RedHat contribution

# G1 tuning
-XX:G1HeapRegionSize=16m      # larger regions reduce overhead for large heaps
-XX:InitiatingHeapOccupancyPercent=45  # start concurrent GC when heap 45% full

# Monitoring / diagnostics
-XX:+HeapDumpOnOutOfMemoryError
-XX:HeapDumpPath=/var/log/heapdump.hprof
-XX:+PrintGCDetails
-Xlog:gc*:file=/var/log/gc.log:time,uptime:filecount=5,filesize=20m

# JIT
-XX:+PrintCompilation            # see what JIT compiles
-XX:CompileThreshold=5000        # method call count before JIT
-XX:+TieredCompilation           # default; C1→C2 tiered

# Java Flight Recorder (production safe, low overhead)
java -XX:+FlightRecorder \
     -XX:StartFlightRecording=duration=120s,filename=/tmp/trade-recording.jfr \
     MyApp
```

### Common Performance Anti-Patterns

```java
// 1. Autoboxing in hot loops
Double total = 0.0;
for (Trade t : trades) total += t.getValue(); // creates Double objects → GC pressure
// Fix:
double total = 0.0; // primitive

// 2. String concatenation in loop
String result = "";
for (Trade t : trades) result += t.getId() + ","; // O(n^2), new String each iteration
// Fix:
StringBuilder sb = new StringBuilder();
for (Trade t : trades) sb.append(t.getId()).append(",");

// 3. Unnecessary object creation in hot path
// Bad – creates new SimpleDateFormat per call (heavy)
String fmt(Date d) { return new SimpleDateFormat("yyyy-MM-dd").format(d); }
// Fix: ThreadLocal<SimpleDateFormat> or use DateTimeFormatter (thread-safe)

// 4. Incorrect use of parallelStream
// Overhead > benefit for small lists or I/O bound operations
trades.parallelStream().forEach(t -> db.save(t)); // don't do this (I/O in parallel)

// 5. Memory leak via static collections
class Cache {
    static Map<String, byte[]> data = new HashMap<>();  // never cleared → OOM
    // Fix: use WeakHashMap, Caffeine, or bounded cache with eviction
}
```

---

## 6. JDBC & Database Connectivity

### Full Production-Grade JDBC Pattern

```java
public class InstrumentRepository {
    private final DataSource dataSource; // injected via constructor (DI)

    public InstrumentRepository(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    // Parameterized query – prevent SQL injection
    public Optional<Instrument> findByIsin(String isin) {
        String sql = "SELECT * FROM instruments WHERE isin = ? AND is_active = 1";
        try (Connection conn = dataSource.getConnection();
             PreparedStatement ps = conn.prepareStatement(sql)) {
            ps.setString(1, isin);
            ps.setQueryTimeout(5); // 5 second timeout
            try (ResultSet rs = ps.executeQuery()) {
                if (rs.next()) {
                    return Optional.of(mapRow(rs));
                }
                return Optional.empty();
            }
        } catch (SQLException e) {
            throw new DataAccessException("Failed to find instrument " + isin, e);
        }
    }

    // Batch update – efficient for bulk operations
    public void bulkUpdatePrices(List<PriceUpdate> updates) {
        String sql = "UPDATE market_prices SET price = ?, updated_at = ? WHERE isin = ?";
        try (Connection conn = dataSource.getConnection();
             PreparedStatement ps = conn.prepareStatement(sql)) {
            conn.setAutoCommit(false); // manual transaction
            for (PriceUpdate p : updates) {
                ps.setDouble(1, p.getPrice());
                ps.setTimestamp(2, Timestamp.from(Instant.now()));
                ps.setString(3, p.getIsin());
                ps.addBatch(); // add to batch
            }
            int[] results = ps.executeBatch(); // execute all at once
            conn.commit();
        } catch (SQLException e) {
            // rollback handled by try-with-resources auto-close or explicit
            throw new DataAccessException("Bulk price update failed", e);
        }
    }

    // Row mapper
    private Instrument mapRow(ResultSet rs) throws SQLException {
        return new Instrument(
            rs.getString("isin"),
            rs.getString("symbol"),
            rs.getString("exchange"),
            rs.getDouble("tick_size"),
            rs.getInt("lot_size")
        );
    }
}
```

### Connection Pooling (HikariCP – Production Standard)

```java
HikariConfig config = new HikariConfig();
config.setJdbcUrl("jdbc:oracle:thin:@//host:1521/orcl");
config.setUsername("ibkr_user");
config.setPassword("secret");

// Pool sizing: Hikari recommends = (cpu_cores * 2) + effective_spindle_count
config.setMaximumPoolSize(20);
config.setMinimumIdle(5);
config.setConnectionTimeout(3000);          // throw if no conn available in 3s
config.setIdleTimeout(600_000);             // remove idle conn after 10min
config.setMaxLifetime(1_800_000);           // recycle conn every 30min
config.setConnectionTestQuery("SELECT 1 FROM DUAL"); // Oracle health check

HikariDataSource ds = new HikariDataSource(config);
```

### Transaction Isolation Levels

```
Level                Read Dirty?  Non-repeatable?  Phantom?  Notes
───────────────────  ───────────  ───────────────  ────────  ──────────────────────────
READ_UNCOMMITTED     Yes          Yes              Yes       Fastest; least safe
READ_COMMITTED       No           Yes              Yes       Oracle default (most DBs)
REPEATABLE_READ      No           No               Yes       MySQL default
SERIALIZABLE         No           No               No        Safest; most locking overhead

// Set isolation in JDBC
conn.setTransactionIsolation(Connection.TRANSACTION_READ_COMMITTED);

// In financial systems: usually READ_COMMITTED to avoid locks,
// combined with optimistic locking via version column
```

```sql
-- Optimistic locking pattern (no DB-level lock; conflict detection)
UPDATE orders
   SET status = 'FILLED', version = version + 1
 WHERE order_id = ?
   AND version = ?     -- fails if another thread already updated
```

---

## 7. Java Streams & Functional Programming

### Functional Interfaces

```java
// Core functional interfaces (java.util.function)
Predicate<Trade>         filter   = t -> t.getValue() > 1_000_000;
Function<Trade, String>  mapper   = Trade::getSymbol;
Consumer<Trade>          printer  = t -> System.out.println(t.getId());
Supplier<Trade>          factory  = () -> new Trade();
BiFunction<String, Double, String> fmt = (sym, price) -> sym + "=" + price;

// Method references
Function<String, Integer> parse = Integer::parseInt;         // static
Consumer<Order>           log   = System.out::println;       // instance of 'out'
Function<String, Trade>   ctor  = Trade::new;                // constructor
```

### Streams – Production Patterns

```java
List<Trade> trades = getTradeData();

// 1. Filter + aggregate
Map<String, DoubleSummaryStatistics> bySymbol = trades.stream()
    .filter(t -> t.getStatus() == TradeStatus.FILLED)
    .collect(Collectors.groupingBy(
        Trade::getSymbol,
        Collectors.summarizingDouble(Trade::getValue)
    ));
System.out.printf("AAPL avg trade size: %.2f%n", bySymbol.get("AAPL").getAverage());

// 2. FlatMap – flatten nested collections
List<Instrument> allInstruments = portfolios.stream()
    .flatMap(p -> p.getInstruments().stream())
    .distinct()
    .collect(Collectors.toList());

// 3. Partitioning
Map<Boolean, List<Trade>> partitioned = trades.stream()
    .collect(Collectors.partitioningBy(t -> t.getValue() > 1_000_000));
List<Trade> highValue = partitioned.get(true);
List<Trade> lowValue  = partitioned.get(false);

// 4. Joining
String symbolList = trades.stream()
    .map(Trade::getSymbol)
    .distinct()
    .sorted()
    .collect(Collectors.joining(", ", "[", "]")); // [AAPL, GOOGL, MSFT]

// 5. toMap (watch for duplicate key exception)
Map<String, Trade> lastTradeBySymbol = trades.stream()
    .collect(Collectors.toMap(
        Trade::getSymbol,
        t -> t,
        (existing, replacement) -> replacement // mergeFunction for duplicates
    ));

// 6. Custom collector (reduce to single value)
Optional<Trade> biggestTrade = trades.stream()
    .max(Comparator.comparingDouble(Trade::getValue));

// 7. Parallel stream – use only for CPU-bound, large collections, no shared mutable state
double totalNotional = trades.parallelStream()
    .mapToDouble(t -> t.getPrice() * t.getQuantity())
    .sum();
// Warning: don't use parallelStream with DB calls, synchronized blocks, or I/O
```

### Optional (No More NullPointerException)

```java
Optional<Instrument> inst = instrumentRepo.findByIsin(isin);

// Bad – defeats purpose of Optional
if (inst.isPresent()) { return inst.get(); }

// Good
inst.ifPresent(i -> log.info("Found: {}", i.getSymbol()));

String symbol = inst.map(Instrument::getSymbol)
                    .orElse("UNKNOWN");

double tickSize = inst.map(Instrument::getTickSize)
                      .orElseThrow(() -> new NotFoundException("Instrument: " + isin));

// Chain Optionals
Optional<String> currency = inst.map(Instrument::getCurrency)
    .filter(c -> !c.isEmpty());
```

---

## 8. Design Patterns in Trading Systems

### Singleton – Thread-Safe (Double-Checked)

```java
public class ConnectionPool {
    // volatile prevents reordering of object construction + reference assignment
    private static volatile ConnectionPool instance;

    private ConnectionPool() { /* init pool */ }

    public static ConnectionPool getInstance() {
        if (instance == null) {                         // 1st check (no sync)
            synchronized (ConnectionPool.class) {
                if (instance == null) {                 // 2nd check (with sync)
                    instance = new ConnectionPool();
                }
            }
        }
        return instance;
    }
}
// Better alternative: use enum (inherently thread-safe, handles serialization)
enum DBPool { INSTANCE; public Connection getConn() { return pool.borrow(); } }
```

### Factory & Abstract Factory (Order Creation)

```java
// Factory Method
interface Order { void execute(); String getType(); }

class MarketOrder implements Order {
    private final String symbol; private final int qty;
    public MarketOrder(String symbol, int qty) { this.symbol = symbol; this.qty = qty; }
    public void execute() { /* route to exchange at market */ }
    public String getType() { return "MARKET"; }
}

class LimitOrder implements Order {
    private final String symbol; private final int qty; private final double price;
    public LimitOrder(String symbol, int qty, double price) {
        this.symbol = symbol; this.qty = qty; this.price = price;
    }
    public void execute() { /* add to order book at limit */ }
    public String getType() { return "LIMIT"; }
}

class OrderFactory {
    public static Order create(OrderRequest req) {
        return switch (req.getType()) {
            case "MARKET" -> new MarketOrder(req.getSymbol(), req.getQty());
            case "LIMIT"  -> new LimitOrder(req.getSymbol(), req.getQty(), req.getPrice());
            case "STOP"   -> new StopOrder(req.getSymbol(), req.getQty(), req.getStop());
            default -> throw new UnsupportedOrderTypeException(req.getType());
        };
    }
}
```

### Builder Pattern (Complex Objects)

```java
// Immutable order object with many optional fields
public class TradeRequest {
    private final String symbol;     // required
    private final int quantity;      // required
    private final double limitPrice; // optional
    private final String tif;        // optional (default: DAY)
    private final String account;    // optional

    private TradeRequest(Builder b) {
        this.symbol = b.symbol;
        this.quantity = b.quantity;
        this.limitPrice = b.limitPrice;
        this.tif = b.tif;
        this.account = b.account;
    }

    public static class Builder {
        private final String symbol;
        private final int quantity;
        private double limitPrice = -1;
        private String tif = "DAY";
        private String account;

        public Builder(String symbol, int quantity) {
            this.symbol = symbol;
            this.quantity = quantity;
        }
        public Builder limitPrice(double p) { this.limitPrice = p; return this; }
        public Builder tif(String t) { this.tif = t; return this; }
        public Builder account(String a) { this.account = a; return this; }
        public TradeRequest build() {
            if (symbol == null || quantity <= 0) throw new IllegalStateException();
            return new TradeRequest(this);
        }
    }
}

// Usage
TradeRequest req = new TradeRequest.Builder("AAPL", 100)
    .limitPrice(175.50)
    .tif("GTC")
    .account("U1234567")
    .build();
```

### Observer / Event Bus (Market Data, Risk Events)

```java
// Type-safe event bus
@FunctionalInterface
interface EventHandler<T> { void handle(T event); }

class EventBus<T> {
    private final List<EventHandler<T>> handlers = new CopyOnWriteArrayList<>();

    public void subscribe(EventHandler<T> handler) { handlers.add(handler); }

    public void publish(T event) {
        handlers.forEach(h -> {
            try { h.handle(event); }
            catch (Exception e) { log.error("Handler failed", e); }
        });
    }
}

// Usage:
EventBus<PriceUpdate> priceBus = new EventBus<>();
priceBus.subscribe(update -> riskEngine.recompute(update));
priceBus.subscribe(update -> positionService.markToMarket(update));
priceBus.publish(new PriceUpdate("AAPL", 175.50));
```

### Strategy Pattern (Routing Strategies)

```java
// Different execution strategies behind same interface
interface RoutingStrategy {
    Exchange selectExchange(Order order, List<Exchange> available);
}

class BestPriceStrategy implements RoutingStrategy {
    public Exchange selectExchange(Order order, List<Exchange> exchanges) {
        return exchanges.stream()
            .min(Comparator.comparingDouble(e -> e.getAskPrice(order.getSymbol())))
            .orElseThrow();
    }
}

class FastestFillStrategy implements RoutingStrategy {
    public Exchange selectExchange(Order order, List<Exchange> exchanges) {
        return exchanges.stream()
            .max(Comparator.comparingDouble(e -> e.getFillProbability(order)))
            .orElseThrow();
    }
}

class SmartRouter {
    private RoutingStrategy strategy;

    public void setStrategy(RoutingStrategy strategy) { this.strategy = strategy; }

    public void route(Order order, List<Exchange> exchanges) {
        Exchange chosen = strategy.selectExchange(order, exchanges);
        chosen.submit(order);
    }
}
```

### Command Pattern (Order Lifecycle)

```java
interface Command { void execute(); void undo(); }

class SubmitOrderCommand implements Command {
    private final Order order;
    private final OrderBook book;

    public SubmitOrderCommand(Order order, OrderBook book) {
        this.order = order; this.book = book;
    }
    public void execute() { book.add(order); }
    public void undo()    { book.remove(order); }
}

// CommandInvoker maintains history for undo
class TradingSession {
    private final Deque<Command> history = new ArrayDeque<>();

    public void execute(Command cmd) {
        cmd.execute();
        history.push(cmd);
    }

    public void undo() {
        if (!history.isEmpty()) history.pop().undo();
    }
}
```

### Decorator Pattern (Order Validation Chain)

```java
interface OrderValidator {
    ValidationResult validate(Order order);
}

// Base validator
class MarginValidator implements OrderValidator {
    public ValidationResult validate(Order order) {
        double buyingPower = accountService.getBuyingPower(order.getAccountId());
        double notional = order.getQuantity() * order.getPrice();
        return buyingPower >= notional
            ? ValidationResult.ok()
            : ValidationResult.fail("Insufficient margin");
    }
}

// Decorator adds behavior
class PositionLimitDecorator implements OrderValidator {
    private final OrderValidator wrapped;

    PositionLimitDecorator(OrderValidator wrapped) { this.wrapped = wrapped; }

    public ValidationResult validate(Order order) {
        int currentPos = positionService.getPosition(order.getSymbol());
        if (currentPos + order.getQuantity() > MAX_POSITION) {
            return ValidationResult.fail("Position limit exceeded");
        }
        return wrapped.validate(order); // delegate to wrapped
    }
}

// Chain: PDT check → Position limit → Margin check
OrderValidator validator = new PDTDecorator(
    new PositionLimitDecorator(
        new MarginValidator()
    )
);
```

---

## 9. Exception Handling

### Exception Hierarchy

```
Throwable
├── Error (JVM errors – don't catch normally)
│   ├── OutOfMemoryError
│   ├── StackOverflowError
│   └── VirtualMachineError
└── Exception
    ├── RuntimeException (unchecked – don't need try/catch)
    │   ├── NullPointerException
    │   ├── IllegalArgumentException
    │   ├── IllegalStateException
    │   └── ConcurrentModificationException
    └── Checked Exceptions (must declare or catch)
        ├── IOException
        ├── SQLException
        └── InterruptedException (very important in concurrent code!)
```

### Production Exception Patterns

```java
// 1. Custom exception hierarchy for trading domain
public class TradingException extends RuntimeException {
    private final String tradeId;
    private final ErrorCode code;

    public TradingException(String tradeId, ErrorCode code, String msg, Throwable cause) {
        super(msg, cause);
        this.tradeId = tradeId;
        this.code = code;
    }

    // Getter, no setter – immutable exception
}

public class InsufficientMarginException extends TradingException {
    private final double required;
    private final double available;

    public InsufficientMarginException(String tradeId, double required, double available) {
        super(tradeId, ErrorCode.MARGIN_BREACH,
              String.format("Required %.2f, available %.2f", required, available), null);
        this.required = required;
        this.available = available;
    }
}

// 2. Try-with-resources for ALL resources (Connection, Stream, Lock, etc.)
try (Connection conn = dataSource.getConnection();
     PreparedStatement ps = conn.prepareStatement(sql)) {
    // resources auto-closed in REVERSE ORDER even on exception
} catch (SQLException e) {
    throw new DataAccessException("DB operation failed", e); // wrap checked
}

// 3. InterruptedException – NEVER swallow silently
public void processOrders() {
    try {
        Order order = queue.take();
    } catch (InterruptedException e) {
        Thread.currentThread().interrupt(); // RESTORE interrupt flag
        log.warn("Processing interrupted");
        return; // exit cleanly
    }
}

// 4. Multi-catch (Java 7+)
try { /* risky */ }
catch (IOException | SQLException e) {
    throw new InfrastructureException("IO or DB error", e);
}

// 5. Exception translation – don't let infrastructure exceptions leak
public Instrument findInstrument(String isin) {
    try {
        return jdbcTemplate.queryForObject(sql, params, rowMapper);
    } catch (EmptyResultDataAccessException e) {
        throw new InstrumentNotFoundException(isin);  // domain exception
    } catch (DataAccessException e) {
        throw new ReferenceDataException("DB lookup failed for " + isin, e);
    }
}
```

---

## 10. Memory Management & GC

### Heap Regions

```
Young Generation (short-lived objects):
  Eden Space     → new objects created here
  Survivor (S0)  → survivors from Eden after Minor GC
  Survivor (S1)  → alternate survivor space

Old/Tenured Generation:
  Objects that survive N Minor GCs get promoted here
  Major GC (full GC) more expensive; avoid frequent major GCs

Metaspace (Java 8+, was PermGen):
  Class metadata, static fields
  Grows dynamically; can OOM if too many classes loaded (ClassLoader leak)

GC Events:
  Minor GC: Young Gen only; fast (milliseconds)
  Major GC: Old Gen; slow (can pause application)
  Full GC:  Entire heap; most disruptive; avoid by tuning
```

### GC Algorithms

| GC | Java Version | Pause | Throughput | Notes |
|---|---|---|---|---|
| **Serial** | All | High | Low | Single-threaded; tiny heaps only |
| **Parallel** | All | Medium | High | Throughput focus; batch processing |
| **G1 (Garbage First)** | 9+ (default) | Low-Medium | High | Region-based; predictable pauses |
| **ZGC** | 11+ | < 1ms | Medium | Concurrent; scales to TBs; ideal for latency |
| **Shenandoah** | 11+ | < 1ms | Medium | Concurrent; OpenJDK; low-pause |

```bash
# For IBKR-style reference data systems (large heap, moderate latency tolerance)
-XX:+UseG1GC -XX:MaxGCPauseMillis=150 -Xms8g -Xmx8g

# For order execution engine (ultra-low latency)
-XX:+UseZGC -Xms16g -Xmx16g
```

### Memory Leaks – Common Causes

```java
// 1. Static collection grows without bound
class Cache {
    static final Map<String, byte[]> cache = new HashMap<>();
    static void store(String key, byte[] data) { cache.put(key, data); }
    // Never removed → OOM over time
    // Fix: use Caffeine / Guava LoadingCache with size/time eviction
}

// 2. Unclosed resources
Connection conn = ds.getConnection();
// conn never closed → connection leak → pool exhausted
// Fix: always try-with-resources

// 3. Inner class holding outer reference
class Outer {
    private byte[] largeData = new byte[10_000_000];
    class Inner { /* holds implicit reference to Outer → largeData not collected */ }
    // Fix: use static inner class if outer ref not needed
}

// 4. ThreadLocal not removed
ThreadLocal<UserContext> ctx = new ThreadLocal<>();
ctx.set(new UserContext(userId));
// If thread pool reuses thread without cleanup → stale context
// Fix: always call ctx.remove() in finally block

// 5. Listener not unregistered
eventBus.subscribe(myListener); // myListener holds references
// Object stays alive even if app no longer needs it
// Fix: use WeakReference listeners or explicit unsubscribe lifecycle
```

---

## 11. Spring Framework Essentials

### Dependency Injection & IoC

```java
// Constructor injection (preferred – immutable, easier to test)
@Service
public class OrderService {
    private final OrderRepository repo;
    private final RiskEngine riskEngine;
    private final OrderRouter router;

    public OrderService(OrderRepository repo, RiskEngine riskEngine, OrderRouter router) {
        this.repo = repo;
        this.riskEngine = riskEngine;
        this.router = router;
    }

    @Transactional
    public TradeResult submitOrder(OrderRequest req) {
        Order order = Order.from(req);
        riskEngine.check(order);   // throws on risk violation
        repo.save(order);
        return router.route(order);
    }
}

// Configuration class (Java-based config – preferred over XML)
@Configuration
public class AppConfig {
    @Bean
    public DataSource dataSource(@Value("${db.url}") String url,
                                  @Value("${db.user}") String user,
                                  @Value("${db.pass}") String pass) {
        HikariConfig cfg = new HikariConfig();
        cfg.setJdbcUrl(url); cfg.setUsername(user); cfg.setPassword(pass);
        cfg.setMaximumPoolSize(20);
        return new HikariDataSource(cfg);
    }

    @Bean
    public OrderRepository orderRepository(DataSource ds) {
        return new JdbcOrderRepository(ds);
    }
}
```

### Spring Annotations Cheat Sheet

```
@Component       Generic Spring-managed bean
@Service         Business layer (semantic alias)
@Repository      Data access layer; translates DB exceptions to DataAccessException
@Controller      Web MVC controller
@RestController  @Controller + @ResponseBody (JSON responses)

@Autowired       Inject dependency (prefer constructor injection)
@Value("${key}") Inject property value from application.properties / env
@Profile         Activate bean only for specific profile (dev/prod)

@Transactional   Wrap method in DB transaction; rollback on RuntimeException
@Cacheable       Cache method result; skip invocation if cached
@Scheduled       Run method on fixed rate/delay
@Async           Run method asynchronously in Thread pool

@SpringBootApplication = @Configuration + @EnableAutoConfiguration + @ComponentScan
```

### Spring @Transactional – Key Behaviors

```java
@Transactional(
    propagation = Propagation.REQUIRED,        // default; join existing or start new
    isolation   = Isolation.READ_COMMITTED,    // isolation level
    rollbackFor = { TradingException.class },  // rollback on this exception
    timeout     = 5                            // 5 second timeout
)
public void processCorpAction(CorporateAction ca) {
    // All operations share ONE transaction
    positionService.adjust(ca);       // part of same tx
    priceService.adjustHistorical(ca); // part of same tx
    auditLog.record(ca);              // part of same tx
    // If any throws → ALL rolled back
}

// GOTCHA: @Transactional only works for calls from OUTSIDE the bean
// (Spring uses proxy; self-invocation bypasses proxy)
class A {
    @Transactional
    public void outer() { inner(); } // inner()'s @Transactional is IGNORED

    @Transactional
    public void inner() { /* not in its own tx when called from outer() */ }
}
```

---

## 12. Kafka Integration in Java

### Kafka Core Concepts (IBKR uses Kafka for event streaming)

```
Topic       = named stream of records; append-only log
Partition   = unit of parallelism; ordered within partition
Offset      = position of message in partition; consumer tracks this
Consumer Group = multiple consumers share topic; each partition → one consumer
Broker      = Kafka server node
ZooKeeper / KRaft = cluster metadata management

Producer → Topic[Partition0, Partition1, ...]
                  ↓
Consumer Group A: [Consumer1 (P0), Consumer2 (P1)]
Consumer Group B: [Consumer3 (P0, P1)] (independent offset)
```

### Java Producer

```java
Properties props = new Properties();
props.put("bootstrap.servers", "kafka:9092");
props.put("key.serializer",   "org.apache.kafka.common.serialization.StringSerializer");
props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
props.put("acks", "all");           // wait for all ISR replicas to ack
props.put("retries", 3);
props.put("linger.ms", 5);         // batch for 5ms to improve throughput
props.put("compression.type", "snappy");

KafkaProducer<String, String> producer = new KafkaProducer<>(props);

// Async send with callback
ProducerRecord<String, String> record = new ProducerRecord<>(
    "trade-events", order.getOrderId(), toJson(executionReport)
);
producer.send(record, (metadata, exception) -> {
    if (exception != null) {
        log.error("Failed to publish trade event", exception);
    } else {
        log.debug("Published to partition {} offset {}", metadata.partition(), metadata.offset());
    }
});
producer.flush(); // ensure all buffered records are sent
```

### Java Consumer

```java
Properties props = new Properties();
props.put("bootstrap.servers", "kafka:9092");
props.put("group.id", "risk-engine-group");
props.put("key.deserializer",   "org.apache.kafka.common.serialization.StringDeserializer");
props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
props.put("auto.offset.reset", "earliest");       // start from beginning if no offset
props.put("enable.auto.commit", "false");         // manual commit for at-least-once

KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
consumer.subscribe(List.of("trade-events", "price-updates"));

try {
    while (running) {
        ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));
        for (ConsumerRecord<String, String> record : records) {
            try {
                processEvent(record.value());
            } catch (Exception e) {
                log.error("Failed to process record at offset {}", record.offset(), e);
                // Dead-letter queue pattern: publish to error topic
                dlqProducer.send(new ProducerRecord<>("trade-events-dlq", record.value()));
            }
        }
        consumer.commitSync(); // commit after processing batch
    }
} finally {
    consumer.close();
}
```

---

## 13. Java 11–17+ Modern Features

```java
// Java 11: String methods
" Hello ".strip();           // like trim() but unicode-aware
"".isBlank();                // true if empty or whitespace
"a\nb\nc".lines().count();   // 3 (stream of lines)
"Na".repeat(4);              // NaNaNaNa

// Java 14: Records (immutable data carriers)
record TradeEvent(String orderId, String symbol, double price, Instant timestamp) {}
// Auto-generates: constructor, getters, equals(), hashCode(), toString()
TradeEvent ev = new TradeEvent("ORD1", "AAPL", 175.50, Instant.now());
System.out.println(ev.symbol()); // getter (no 'get' prefix)

// Java 14: Switch Expressions
String category = switch (instrument.getType()) {
    case "EQUITY"  -> "EQ";
    case "FUTURES" -> "FU";
    case "OPTION"  -> "OP";
    default        -> throw new IllegalArgumentException(instrument.getType());
};

// Java 15: Text Blocks (multi-line strings – useful for SQL/JSON)
String sql = """
    SELECT i.isin, i.symbol, p.price
      FROM instruments i
      JOIN market_prices p ON p.isin = i.isin
     WHERE i.exchange = ?
       AND i.is_active = 1
    """;

// Java 16: Pattern Matching instanceof
if (order instanceof LimitOrder lo && lo.getLimitPrice() < 0) {
    throw new InvalidOrderException("Negative price");
}

// Java 17: Sealed Classes (restricts which classes can extend)
sealed interface OrderType permits MarketOrder, LimitOrder, StopOrder {}
final class MarketOrder implements OrderType { }
final class LimitOrder  implements OrderType { private double price; }
// Non-exhaustive switch is now a compile error → safer
double getPrice(OrderType o) {
    return switch (o) {
        case MarketOrder m  -> getMarketPrice(m.getSymbol());
        case LimitOrder l   -> l.getPrice();
        case StopOrder s    -> s.getStopPrice();
        // No default needed – sealed hierarchy is exhaustive
    };
}
```

---

## 14. Common Interview Questions

### Core Java

| Question | Detailed Answer |
|---|---|
| `==` vs `.equals()`? | `==` compares object references (memory address). `.equals()` compares value (logical equality). For `String`, `Integer` etc., use `.equals()`. Always override `equals()` with `hashCode()` together. |
| What is `final`? | Variable: value can't be reassigned (object can still be mutated). Method: can't be overridden. Class: can't be subclassed. Used heavily for immutability and constants. |
| `HashMap` internal working? | Array of `Node<K,V>` buckets. Index = `hashCode() & (capacity-1)`. Collision → linked list; if list grows > 8 → TreeNode (red-black tree). Load factor 0.75 triggers resize (double capacity, rehash). |
| What is a memory leak in Java? | Object that is no longer needed but still referenced (GC can't collect). Common: static collections, unclosed resources, inner class references, ThreadLocal not removed, listeners not unregistered. Diagnose with heap dump + MAT/VisualVM. |
| `Callable` vs `Runnable`? | `Runnable.run()` returns void, can't throw checked exceptions. `Callable.call()` returns `V` and can throw checked exceptions. Use `Callable` with `ExecutorService.submit()` to get a `Future<V>`. |
| `volatile`? | Guarantees visibility of variable changes across threads. Prevents CPU caching + compiler reordering of that variable. Does NOT provide atomicity (use `synchronized` or `Atomic*` for that). |
| `ThreadLocal`? | Separate value per thread. Use for: connection per thread, user session context, SimpleDateFormat instances. Always call `remove()` in finally to prevent leaks in thread pools. |
| Difference between `wait()` and `sleep()`? | `wait()`: releases monitor lock; must be called in `synchronized`; woken by `notify()`/`notifyAll()`. `sleep()`: doesn't release lock; delays execution; woken by interrupt or timeout. |
| What is the Java Memory Model? | Defines visibility + ordering guarantees for multi-threaded programs. Uses happens-before relationship. Key: `volatile` writes happen-before subsequent volatile reads; synchronized blocks establish happens-before. |

### Concurrency

| Question | Detailed Answer |
|---|---|
| Deadlock – how to avoid? | Acquire locks in consistent global order. Use `tryLock(timeout)` instead of blocking. Minimize lock scope. Detect with thread dumps (look for BLOCKED threads in cycles). |
| `synchronized` vs `ReentrantLock`? | `synchronized`: simpler; auto-unlock; can't interrupt waiting. `ReentrantLock`: `tryLock()`, interruptible wait, fairness policy, multiple conditions (`Condition`). Prefer `synchronized` unless you need those features. |
| What is `ConcurrentHashMap`? | Thread-safe map. Java 8+: segments replaced with bin-level locking (CAS + synchronized per bucket). No full-table lock for puts. `null` keys/values not allowed. `computeIfAbsent` is atomic. |
| How does `ForkJoinPool` work? | Divide-and-conquer parallelism. Work-stealing: idle threads steal tasks from busy thread queues. Used by default in `parallelStream()`. Best for CPU-bound recursive tasks. |

### Design

| Question | Detailed Answer |
|---|---|
| How to design a thread-safe LRU cache? | `LinkedHashMap` with `accessOrder=true` + `synchronized` wrapper, OR `ConcurrentHashMap` + `ConcurrentLinkedDeque` for order tracking. Or use Caffeine library (battle-tested). |
| `Singleton` – thread safety options? | 1. Eager init (static final field). 2. DCL with `volatile`. 3. Static holder class (lazy + safe). 4. Enum singleton (simplest, handles serialization). |
| How would you implement a rate limiter in Java? | Token bucket: `AtomicLong` tokens + `ScheduledExecutorService` to refill. Or `Semaphore` with fixed permits. Production: Resilience4j `RateLimiter` or Guava `RateLimiter`. |

---

*Notes maintained for IBKR SDE Interview — Java core + financial context*
*Last updated: March 2026*
