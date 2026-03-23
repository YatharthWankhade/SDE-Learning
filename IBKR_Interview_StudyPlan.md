# IBKR Interview Study Plan – Master Index

> **Role:** Java Developer – Oracle Reference Database Systems  
> **Company:** Interactive Brokers (IBKR)  
> **Created:** March 2026

---

## JD Requirements Mapping

| JD Requirement | Priority | Notes File |
|---|---|---|
| Java (Web + Non-Web applications) | 🔴 MUST | [Java Notes](Languages/Java/Java_Interview_IBKR.md) |
| Oracle / SQL | 🔴 MUST | [Oracle/SQL/PL-SQL Notes](DBMS/Oracle_SQL_PLSQL_Interview.md) |
| PL/SQL | 🔴 MUST | [Oracle/SQL/PL-SQL Notes](DBMS/Oracle_SQL_PLSQL_Interview.md) |
| Unix/Linux environments | 🔴 MUST | [Linux/Shell Notes](OperatingSystems/Linux_Unix_Shell_Interview.md) |
| Shell scripting (Unix Shell) | 🔴 MUST | [Linux/Shell Notes](OperatingSystems/Linux_Unix_Shell_Interview.md) |
| PERL scripting | 🔴 MUST | [Linux/Shell Notes](OperatingSystems/Linux_Unix_Shell_Interview.md) |
| Python | 🔴 MUST | [Python Notes](Languages/Python/Python_Interview_IBKR.md) |
| Scalability & large data systems | 🔴 MUST | [Scalable Systems Notes](SystemDesign/ScalableSystems_LargeData_Interview.md) |
| Reference Database Systems | 🔴 MUST | [Financial Domain Notes](SystemDesign/FinancialServices_IBKR_Domain.md) |
| Kafka | 🟡 PREFERRED | [Kafka Notes](SystemDesign/Kafka_Interview_IBKR.md) |
| REST web services | 🟡 PREFERRED | [REST/Git/Jenkins Notes](SystemDesign/REST_Git_Jenkins_Interview.md) |
| GIT | 🟡 PREFERRED | [REST/Git/Jenkins Notes](SystemDesign/REST_Git_Jenkins_Interview.md) |
| Jenkins | 🟡 PREFERRED | [REST/Git/Jenkins Notes](SystemDesign/REST_Git_Jenkins_Interview.md) |
| Financial Services domain | 🟡 PREFERRED | [Financial Domain Notes](SystemDesign/FinancialServices_IBKR_Domain.md) |

---

## Study Files

| File | Topics Covered |
|---|---|
| [`Languages/Java/Java_Interview_IBKR.md`](Languages/Java/Java_Interview_IBKR.md) | OOP, Collections, Multithreading, JVM, JDBC, Streams, Design Patterns, GC |
| [`DBMS/Oracle_SQL_PLSQL_Interview.md`](DBMS/Oracle_SQL_PLSQL_Interview.md) | SQL, Joins, Indexes, ACID, Transactions, PL/SQL, Procedures, Cursors, Triggers, Partitioning, Window Functions |
| [`OperatingSystems/Linux_Unix_Shell_Interview.md`](OperatingSystems/Linux_Unix_Shell_Interview.md) | Linux commands, permissions, processes, Bash scripting, PERL scripting, log processing, cron |
| [`Languages/Python/Python_Interview_IBKR.md`](Languages/Python/Python_Interview_IBKR.md) | Python fundamentals, performance optimization, cx_Oracle, concurrency (threading/multiprocessing/asyncio) |
| [`SystemDesign/Kafka_Interview_IBKR.md`](SystemDesign/Kafka_Interview_IBKR.md) | Kafka architecture, producers, consumers, partitions, replication, scaling, Kafka Streams |
| [`SystemDesign/REST_Git_Jenkins_Interview.md`](SystemDesign/REST_Git_Jenkins_Interview.md) | REST principles, HTTP methods, API design, JWT, Git workflow, Jenkins pipelines |
| [`SystemDesign/FinancialServices_IBKR_Domain.md`](SystemDesign/FinancialServices_IBKR_Domain.md) | Capital markets, trade lifecycle, instruments, ISIN/CUSIP, settlement, risk, IBKR context |
| [`SystemDesign/ScalableSystems_LargeData_Interview.md`](SystemDesign/ScalableSystems_LargeData_Interview.md) | CAP theorem, DB sharding, caching (Redis), load balancing, IBKR-scale design scenarios |

---

## 2-Week Study Schedule

### Week 1 – Core Technical

| Day | Focus | Time |
|---|---|---|
| Mon | Java – OOP, Collections, Generics | 2h |
| Tue | Java – Multithreading, Concurrency, CompletableFuture | 2h |
| Wed | Oracle SQL – Joins, Indexes, Query Optimization, EXPLAIN PLAN | 2h |
| Thu | Oracle PL/SQL – Procedures, Functions, Cursors, Triggers, Packages | 2h |
| Fri | Linux/Shell – Commands, permissions, bash scripting | 1.5h |
| Sat | PERL scripting + Log processing | 1.5h |
| Sun | Review Week 1 + LeetCode SQL questions | 2h |

### Week 2 – Advanced & Domain

| Day | Focus | Time |
|---|---|---|
| Mon | Python – Performance, cx_Oracle, concurrency | 2h |
| Tue | Kafka – Architecture, producers/consumers, Java client | 2h |
| Wed | Scalable Systems – Partitioning, caching, large data design | 2h |
| Thu | Financial Domain – Trade lifecycle, instruments, IBKR context | 1.5h |
| Fri | REST APIs + Git + Jenkins pipeline | 1.5h |
| Sat | Mock interview questions from all topics | 2h |
| Sun | Review weak areas + IBKR-specific scenario practice | 2h |

---

## Key Topics to Emphasize (IBKR-specific)

### 1. Oracle Reference Data Design
```sql
-- Know: MERGE, partitioning, SCD Type 2, window functions
-- Know: PL/SQL packages, bulk operations (BULK COLLECT, FORALL)
-- Know: Execution plans (EXPLAIN PLAN, DBMS_XPLAN)
```

### 2. Large-Scale Data Processing
```
- How to process millions of trades/ticks per day
- Batch processing patterns (chunked reads, bulk inserts)
- Kafka consumer groups for parallel processing
- Oracle partition pruning for efficient queries
```

### 3. Java + Oracle Integration
```java
// Know: PreparedStatement, CallableStatement, executemany equivalent in JDBC
// Know: HikariCP connection pooling
// Know: Calling PL/SQL procedures from Java
// Know: ResultSet handling, batch operations
```

### 4. System Design – Market Data
```
Be ready to design:
  "How would you build a system to ingest 1M market ticks/sec and 
   make them available to 50 downstream consumers with < 100ms latency?"
  
Answer: Kafka (12+ partitions) + consumer groups + Redis for hot data + Oracle for persistence
```

---

## Top 20 IBKR Interview Questions Cheatsheet

### Java
1. What's the difference between `synchronized` and `ReentrantLock`?
2. Explain `ConcurrentHashMap` – how does it achieve thread safety in Java 8+?
3. How do you prevent memory leaks in long-running Java applications?
4. What is the difference between `ExecutorService.submit()` and `execute()`?
5. Explain Java's `volatile` keyword and when to use it.

### Oracle / SQL
6. How do you optimize a query that does a full table scan on a 500M row table?
7. What's the difference between `RANK()` and `DENSE_RANK()`?
8. Explain Oracle's MVCC model – how do readers not block writers?
9. Write a PL/SQL procedure that bulk-loads market data with error handling.
10. What is Oracle partitioning and when would you use it at IBKR?

### Linux / Scripting
11. How do you find the top 10 most frequently occurring ERROR messages in a log file?
12. Write a shell script that monitors a directory for new files and triggers a Java process.
13. What does `set -euo pipefail` do in a bash script?
14. How do you use PERL to parse a fixed-width financial data file?

### Kafka
15. How do you ensure exactly-once message delivery in Kafka?
16. A Kafka consumer is lagging by 10 million messages. What could be the cause and fix?

### System Design
17. Design a reference data system for 1 million financial instruments globally.
18. How would you handle corporate actions (stock splits) across all downstream systems?

### Financial Domain
19. Explain the trade lifecycle from order entry to settlement.
20. What is T+1 settlement and why does it matter operationally?

---

## Quick Reference: Critical Commands

```bash
# Oracle SQL*Plus
sqlplus user/pass@host:1521/service
@script.sql
SET SERVEROUTPUT ON  -- enable DBMS_OUTPUT.PUT_LINE

# Kafka
kafka-topics.sh --list --bootstrap-server localhost:9092
kafka-consumer-groups.sh --describe --group mygroup --bootstrap-server localhost:9092

# Linux log analysis
grep "ERROR" app.log | awk '{print $NF}' | sort | uniq -c | sort -rn | head 10
tail -f app.log | grep --line-buffered "CRITICAL"

# Git
git log --oneline --graph --all
git stash && git pull && git stash pop
git rebase -i HEAD~5   # interactive rebase to squash commits
```
