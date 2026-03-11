# Oracle, SQL & PL/SQL – In-Depth Interview Notes (IBKR JD)

## Table of Contents
1. [SQL Fundamentals](#1-sql-fundamentals)
2. [Joins](#2-joins)
3. [Indexes & Query Optimization](#3-indexes--query-optimization)
4. [Transactions & ACID](#4-transactions--acid)
5. [Oracle-Specific Features](#5-oracle-specific-features)
6. [PL/SQL – Procedures, Functions, Triggers](#6-plsql--procedures-functions-triggers)
7. [Partitioning (Large Data Sets)](#7-partitioning-large-data-sets)
8. [Analytical & Window Functions](#8-analytical--window-functions)
9. [Reference Database Design Patterns](#9-reference-database-design-patterns)
10. [Common Interview Questions](#10-common-interview-questions)

---

## 1. SQL Fundamentals

### DDL / DML / DCL / TCL
| Category | Commands |
|---|---|
| **DDL** | `CREATE`, `ALTER`, `DROP`, `TRUNCATE`, `RENAME` |
| **DML** | `INSERT`, `UPDATE`, `DELETE`, `MERGE` |
| **DCL** | `GRANT`, `REVOKE` |
| **TCL** | `COMMIT`, `ROLLBACK`, `SAVEPOINT` |

### Aggregate Functions
```sql
SELECT 
    symbol,
    COUNT(*)                    AS trade_count,
    SUM(quantity * price)       AS total_value,
    AVG(price)                  AS avg_price,
    MIN(price)                  AS day_low,
    MAX(price)                  AS day_high
FROM trades
WHERE trade_date = TRUNC(SYSDATE)
GROUP BY symbol
HAVING SUM(quantity * price) > 1000000
ORDER BY total_value DESC;
```

### MERGE (Upsert) – Critical for Reference DB Systems
```sql
-- Insert or update instrument reference data
MERGE INTO instruments tgt
USING (SELECT 'AAPL' AS isin, 'Apple Inc' AS name, 'EQUITY' AS type FROM dual) src
ON (tgt.isin = src.isin)
WHEN MATCHED THEN
    UPDATE SET tgt.name = src.name, tgt.type = src.type, tgt.updated_dt = SYSDATE
WHEN NOT MATCHED THEN
    INSERT (isin, name, type, created_dt)
    VALUES (src.isin, src.name, src.type, SYSDATE);
```

---

## 2. Joins

```sql
-- INNER JOIN – matching rows only
SELECT t.trade_id, i.name, t.quantity
FROM trades t
INNER JOIN instruments i ON t.isin = i.isin;

-- LEFT OUTER JOIN – all trades, even without instrument data
SELECT t.trade_id, i.name
FROM trades t
LEFT JOIN instruments i ON t.isin = i.isin;

-- SELF JOIN – hierarchical data (e.g. manager-employee)
SELECT e.name AS employee, m.name AS manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.emp_id;

-- CROSS JOIN – Cartesian product (use with care!)
SELECT a.symbol, b.exchange FROM symbols a CROSS JOIN exchanges b;
```

### Join Types Visual
```
INNER:  A ∩ B
LEFT:   A ∪ (A ∩ B)  [all of A]
RIGHT:  B ∪ (A ∩ B)  [all of B]
FULL:   A ∪ B
```

---

## 3. Indexes & Query Optimization

### Types of Indexes in Oracle
| Index Type | Use Case |
|---|---|
| **B-Tree** (default) | Equality and range queries on high-cardinality columns |
| **Bitmap** | Low-cardinality columns (e.g., status, type) in data warehouses |
| **Function-Based** | Queries on expressions (`UPPER(name)`) |
| **Composite** | Multi-column predicates (column order matters!) |
| **Unique** | Enforce uniqueness |
| **Partitioned** | Index on partitioned table |

```sql
-- Create a composite index (symbol + trade_date most selective first)
CREATE INDEX idx_trades_sym_dt ON trades(symbol, trade_date);

-- Function-based index
CREATE INDEX idx_upper_name ON instruments(UPPER(name));
SELECT * FROM instruments WHERE UPPER(name) = 'APPLE INC'; -- uses index

-- Check if query uses index
EXPLAIN PLAN FOR SELECT * FROM trades WHERE symbol = 'AAPL';
SELECT * FROM TABLE(DBMS_XPLAN.DISPLAY);
```

### Oracle Query Optimizer Hints
```sql
-- Force index use
SELECT /*+ INDEX(t idx_trades_sym_dt) */ * FROM trades t WHERE symbol = 'AAPL';

-- Full table scan
SELECT /*+ FULL(t) */ * FROM trades t;

-- Parallel query for large tables
SELECT /*+ PARALLEL(t, 8) */ COUNT(*) FROM large_market_data t;
```

### N+1 Query Problem (Java/JDBC)
```java
// BAD: N+1 queries
for (String symbol : symbols) {
    Trade t = jdbc.queryForObject("SELECT * FROM trades WHERE symbol=?", symbol);
}

// GOOD: Single query with IN clause
String sql = "SELECT * FROM trades WHERE symbol IN (" +
             symbols.stream().map(s -> "?").collect(joining(",")) + ")";
```

---

## 4. Transactions & ACID

### ACID Properties
| Property | Meaning | Oracle Mechanism |
|---|---|---|
| **Atomicity** | All-or-nothing | `COMMIT` / `ROLLBACK` |
| **Consistency** | DB goes from valid state to valid state | Constraints, triggers |
| **Isolation** | Concurrent transactions don't interfere | Locking, MVCC |
| **Durability** | Committed data survives failures | Redo logs, archive logs |

### Oracle Isolation Levels
```sql
-- Read Committed (default Oracle) – sees only committed data
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;

-- Serializable – full isolation, may fail with ORA-08177
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;

-- Oracle's MVCC: uses UNDO tablespace; readers don't block writers
```

### Deadlock Example
```sql
-- Session 1
UPDATE trades SET status = 'SETTLED' WHERE trade_id = 101; -- locks row 101
-- Session 2
UPDATE trades SET status = 'SETTLED' WHERE trade_id = 102; -- locks row 102
-- Session 1 now tries to lock 102, Session 2 tries 101 → DEADLOCK
-- Oracle detects and rolls back one session with ORA-00060
```

### SAVEPOINT
```sql
BEGIN
    INSERT INTO trades VALUES (1001, 'AAPL', 100, 150.00, SYSDATE);
    SAVEPOINT after_insert;
    UPDATE positions SET qty = qty + 100 WHERE symbol = 'AAPL';
    -- something fails:
    ROLLBACK TO after_insert; -- only rolls back UPDATE, insert stays
    COMMIT;
END;
```

---

## 5. Oracle-Specific Features

### Sequences (Auto-increment in Oracle)
```sql
CREATE SEQUENCE trade_seq START WITH 1 INCREMENT BY 1 NOCACHE NOCYCLE;

INSERT INTO trades (trade_id, symbol, price)
VALUES (trade_seq.NEXTVAL, 'AAPL', 150.00);

-- Oracle 12c+ identity column
CREATE TABLE trades (
    trade_id NUMBER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    symbol   VARCHAR2(10)
);
```

### Dual Table
```sql
SELECT SYSDATE FROM dual;
SELECT trade_seq.NEXTVAL FROM dual;
SELECT 1+1 FROM dual; -- → 2
```

### Oracle Data Types
| Type | Notes |
|---|---|
| `NUMBER(p,s)` | Numeric, precision p, scale s |
| `VARCHAR2(n)` | Variable-length string (max 4000 bytes in SQL; 32767 in PL/SQL) |
| `DATE` | Date + time (no milliseconds) |
| `TIMESTAMP` | Date + time with fractional seconds |
| `CLOB` | Large character data (> 4000 chars) |
| `BLOB` | Binary large object |

### Rownum vs Row_Number()
```sql
-- Old way: ROWNUM (evaluated before ORDER BY - careful!)
SELECT * FROM (
    SELECT * FROM trades ORDER BY trade_date DESC
) WHERE ROWNUM <= 10;

-- Modern way: FETCH FIRST (Oracle 12c+)
SELECT * FROM trades ORDER BY trade_date DESC FETCH FIRST 10 ROWS ONLY;

-- Window function (flexible)
SELECT * FROM (
    SELECT t.*, ROW_NUMBER() OVER (ORDER BY trade_date DESC) AS rn FROM trades t
) WHERE rn <= 10;
```

---

## 6. PL/SQL – Procedures, Functions, Triggers

### Anonymous Block
```sql
DECLARE
    v_symbol    VARCHAR2(10) := 'AAPL';
    v_price     NUMBER;
BEGIN
    SELECT price INTO v_price FROM market_data WHERE symbol = v_symbol;
    DBMS_OUTPUT.PUT_LINE('Price: ' || v_price);
EXCEPTION
    WHEN NO_DATA_FOUND THEN
        DBMS_OUTPUT.PUT_LINE('Symbol not found');
    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('Error: ' || SQLERRM);
END;
/
```

### Stored Procedure
```sql
CREATE OR REPLACE PROCEDURE load_market_data(
    p_symbol    IN  VARCHAR2,
    p_price     IN  NUMBER,
    p_result    OUT VARCHAR2
) AS
BEGIN
    MERGE INTO market_data tgt
    USING (SELECT p_symbol AS symbol, p_price AS price FROM dual) src
    ON (tgt.symbol = src.symbol)
    WHEN MATCHED THEN UPDATE SET tgt.price = src.price, tgt.updated_dt = SYSDATE
    WHEN NOT MATCHED THEN INSERT (symbol, price, updated_dt)
                          VALUES (src.symbol, src.price, SYSDATE);
    COMMIT;
    p_result := 'SUCCESS';
EXCEPTION
    WHEN OTHERS THEN
        ROLLBACK;
        p_result := 'ERROR: ' || SQLERRM;
END;
/

-- Call from Java
CallableStatement cs = conn.prepareCall("{call load_market_data(?, ?, ?)}");
cs.setString(1, "AAPL");
cs.setDouble(2, 175.50);
cs.registerOutParameter(3, Types.VARCHAR);
cs.execute();
String result = cs.getString(3);
```

### Function
```sql
CREATE OR REPLACE FUNCTION get_portfolio_value(p_account_id NUMBER)
RETURN NUMBER AS
    v_total NUMBER := 0;
BEGIN
    SELECT SUM(p.quantity * m.price)
    INTO v_total
    FROM positions p
    JOIN market_data m ON p.symbol = m.symbol
    WHERE p.account_id = p_account_id;
    RETURN NVL(v_total, 0);
END;
/

-- Usage
SELECT account_id, get_portfolio_value(account_id) AS portfolio_value
FROM accounts;
```

### Cursor (Bulk Processing)
```sql
DECLARE
    CURSOR c_trades IS
        SELECT trade_id, symbol, quantity FROM pending_trades;
    TYPE trade_tab IS TABLE OF c_trades%ROWTYPE;
    l_trades trade_tab;
BEGIN
    OPEN c_trades;
    LOOP
        FETCH c_trades BULK COLLECT INTO l_trades LIMIT 1000; -- batch of 1000
        EXIT WHEN l_trades.COUNT = 0;
        FORALL i IN 1..l_trades.COUNT
            UPDATE trade_settlements SET status = 'PROCESSED'
            WHERE trade_id = l_trades(i).trade_id;
        COMMIT;
    END LOOP;
    CLOSE c_trades;
END;
/
```

### Trigger
```sql
-- Audit trigger: track changes to reference data
CREATE OR REPLACE TRIGGER trg_instrument_audit
AFTER UPDATE ON instruments
FOR EACH ROW
BEGIN
    INSERT INTO audit_log (table_name, operation, old_val, new_val, changed_by, changed_dt)
    VALUES ('INSTRUMENTS', 'UPDATE',
            'ISIN:' || :OLD.isin || ' NAME:' || :OLD.name,
            'ISIN:' || :NEW.isin || ' NAME:' || :NEW.name,
            SYS_CONTEXT('USERENV', 'SESSION_USER'), SYSDATE);
END;
/
```

### Package (Organizing Code)
```sql
-- Package Specification
CREATE OR REPLACE PACKAGE trade_pkg AS
    PROCEDURE process_trade(p_trade_id NUMBER);
    FUNCTION get_trade_count(p_symbol VARCHAR2) RETURN NUMBER;
END trade_pkg;
/

-- Package Body
CREATE OR REPLACE PACKAGE BODY trade_pkg AS
    PROCEDURE process_trade(p_trade_id NUMBER) AS BEGIN NULL; END;
    FUNCTION get_trade_count(p_symbol VARCHAR2) RETURN NUMBER AS
        v_count NUMBER;
    BEGIN
        SELECT COUNT(*) INTO v_count FROM trades WHERE symbol = p_symbol;
        RETURN v_count;
    END;
END trade_pkg;
/
```

---

## 7. Partitioning (Large Data Sets)

> Critical for IBKR – managing massive trade/market data tables

```sql
-- Range Partitioning by date (most common for time-series data)
CREATE TABLE trades (
    trade_id   NUMBER,
    symbol     VARCHAR2(10),
    trade_date DATE,
    quantity   NUMBER,
    price      NUMBER
)
PARTITION BY RANGE (trade_date) (
    PARTITION p_2024_q1 VALUES LESS THAN (DATE '2024-04-01'),
    PARTITION p_2024_q2 VALUES LESS THAN (DATE '2024-07-01'),
    PARTITION p_2024_q3 VALUES LESS THAN (DATE '2024-10-01'),
    PARTITION p_2024_q4 VALUES LESS THAN (DATE '2025-01-01'),
    PARTITION p_future   VALUES LESS THAN (MAXVALUE)
);

-- Query pruning: Oracle only scans relevant partitions
SELECT * FROM trades WHERE trade_date BETWEEN DATE '2024-01-01' AND DATE '2024-03-31';

-- List Partitioning by region
CREATE TABLE accounts PARTITION BY LIST (region) (
    PARTITION p_us   VALUES ('US'),
    PARTITION p_eu   VALUES ('EU', 'UK'),
    PARTITION p_apac VALUES ('APAC', 'IN', 'JP')
);

-- Hash Partitioning for even distribution
CREATE TABLE market_ticks PARTITION BY HASH (symbol) PARTITIONS 8;
```

---

## 8. Analytical & Window Functions

```sql
-- ROW_NUMBER: Rank trades per symbol per day
SELECT symbol, trade_date, quantity, price,
       ROW_NUMBER() OVER (PARTITION BY symbol, trade_date ORDER BY trade_id) AS rn
FROM trades;

-- RANK vs DENSE_RANK
SELECT symbol, price,
       RANK() OVER (ORDER BY price DESC)       AS rank_val,      -- gaps for ties
       DENSE_RANK() OVER (ORDER BY price DESC) AS dense_rank_val -- no gaps
FROM market_data;

-- LAG/LEAD: Compare current price with previous
SELECT symbol, trade_date, price,
       LAG(price, 1) OVER (PARTITION BY symbol ORDER BY trade_date) AS prev_price,
       price - LAG(price, 1) OVER (PARTITION BY symbol ORDER BY trade_date) AS price_change
FROM market_data;

-- Running Total
SELECT trade_date, symbol, quantity,
       SUM(quantity) OVER (PARTITION BY symbol ORDER BY trade_date
                           ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS running_qty
FROM trades;

-- Moving Average (7-day)
SELECT trade_date, price,
       AVG(price) OVER (ORDER BY trade_date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) AS ma7
FROM market_data WHERE symbol = 'AAPL';
```

---

## 9. Reference Database Design Patterns

> IBKR Reference Database = master data for instruments, accounts, counterparties etc.

### Slowly Changing Dimensions (SCD)
```sql
-- SCD Type 2: keep history row, insert new row on change
CREATE TABLE instrument_history (
    hist_id     NUMBER GENERATED ALWAYS AS IDENTITY,
    isin        VARCHAR2(20),
    name        VARCHAR2(200),
    valid_from  DATE,
    valid_to    DATE DEFAULT DATE '9999-12-31',  -- open-ended current record
    is_current  CHAR(1) DEFAULT 'Y'
);

-- On update:
PROCEDURE update_instrument(p_isin VARCHAR2, p_new_name VARCHAR2) AS BEGIN
    -- Close old record
    UPDATE instrument_history SET valid_to = SYSDATE-1, is_current = 'N'
    WHERE isin = p_isin AND is_current = 'Y';
    -- Insert new record
    INSERT INTO instrument_history (isin, name, valid_from, is_current)
    VALUES (p_isin, p_new_name, SYSDATE, 'Y');
    COMMIT;
END;
```

### Common Constraints
```sql
CREATE TABLE orders (
    order_id   NUMBER         CONSTRAINT pk_orders PRIMARY KEY,
    account_id NUMBER         CONSTRAINT fk_account REFERENCES accounts(account_id),
    side       CHAR(1)        CONSTRAINT chk_side CHECK (side IN ('B','S')),
    quantity   NUMBER         CONSTRAINT chk_qty CHECK (quantity > 0),
    symbol     VARCHAR2(10)   NOT NULL,
    created_dt DATE           DEFAULT SYSDATE
);
```

---

## 10. Common Interview Questions

| Question | Key Points |
|---|---|
| **DELETE vs TRUNCATE vs DROP** | DELETE: DML, row-by-row, can rollback; TRUNCATE: DDL, removes all rows, can't rollback; DROP: removes table itself |
| **What is a view?** | Virtual table based on a query; no physical storage; can be updatable if simple |
| **Clustered vs Non-clustered index?** | Oracle doesn't have clustered indexes (SQL Server term); nearest = Index-Organized Table (IOT) |
| **What is normalization?** | 1NF: atomic values; 2NF: no partial dependency; 3NF: no transitive dependency; BCNF: every determinant is a candidate key |
| **When to denormalize?** | Read-heavy analytics, performance critical, reporting tables |
| **What is a materialized view?** | Pre-computed view stored as table; refreshed on schedule; great for aggregation on large datasets |
| **Explain Oracle MVCC** | Multi-Version Concurrency Control; uses UNDO segments; readers don't block writers |
| **What's the difference between UNION and UNION ALL?** | UNION removes duplicates (slower); UNION ALL keeps all rows (faster) |
| **How to avoid SQL injection in Java?** | Always use `PreparedStatement` with `?` parameters; never concatenate user input |
| **What is a correlated subquery?** | Subquery that references outer query; evaluated once per row; usually slow |

```sql
-- Correlated subquery example
SELECT e.name, e.salary
FROM employees e
WHERE e.salary > (
    SELECT AVG(salary) FROM employees WHERE dept_id = e.dept_id  -- references outer e
);

-- Better with window function:
SELECT name, salary FROM (
    SELECT name, salary, AVG(salary) OVER (PARTITION BY dept_id) AS avg_dept_sal
    FROM employees
) WHERE salary > avg_dept_sal;
```
