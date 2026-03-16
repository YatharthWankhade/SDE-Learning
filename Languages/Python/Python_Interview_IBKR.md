# Python – Performance & Scripting Interview Notes (IBKR JD)

## Table of Contents
1. [Python Fundamentals Refresher](#1-python-fundamentals-refresher)
2. [Performance Optimization](#2-performance-optimization)
3. [Database with Python (cx_Oracle)](#3-database-with-python-cx_oracle)
4. [File Processing & Data Handling](#4-file-processing--data-handling)
5. [Concurrency in Python](#5-concurrency-in-python)
6. [Common Libraries for Data Systems](#6-common-libraries-for-data-systems)
7. [Common Interview Questions](#7-common-interview-questions)

---

## 1. Python Fundamentals Refresher

### Data Structures
```python
# List – mutable ordered sequence
symbols = ['AAPL', 'GOOGL', 'MSFT']
symbols.append('AMZN')

# Tuple – immutable ordered sequence (good for fixed records)
trade = ('T001', 'AAPL', 100, 175.50)
trade_id, symbol, qty, price = trade  # unpacking

# Dictionary – key-value store
prices = {'AAPL': 175.50, 'GOOGL': 140.20}
prices.get('TSLA', 0.0)  # safe get with default

# Set – unique elements
unique_symbols = {'AAPL', 'AAPL', 'GOOGL'}  # → {'AAPL', 'GOOGL'}
```

### Comprehensions
```python
# List comprehension
squared = [x**2 for x in range(10) if x % 2 == 0]

# Dictionary comprehension
price_map = {row['symbol']: row['price'] for row in trade_data if row['active']}

# Generator (lazy evaluation – memory efficient for large datasets)
large_stream = (row for row in read_file('trades.csv'))  # doesn't load all in memory
total = sum(float(r.split(',')[2]) for r in large_stream)
```

### Decorators (commonly asked)
```python
import functools, time, logging

# Timing decorator
def timer(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        result = func(*args, **kwargs)
        log.info(f"{func.__name__} took {time.perf_counter()-start:.3f}s")
        return result
    return wrapper

# Retry decorator (useful for DB/API calls)
def retry(max_attempts=3, delay=1.0):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(max_attempts):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if attempt == max_attempts - 1:
                        raise
                    time.sleep(delay * (attempt + 1))
        return wrapper
    return decorator

@timer
@retry(max_attempts=3, delay=2.0)
def fetch_market_data(symbol: str) -> dict:
    # ... API call
    pass
```

---

## 2. Performance Optimization

### Profiling First
```python
# cProfile – find bottlenecks before optimizing
import cProfile, pstats

profiler = cProfile.Profile()
profiler.enable()
process_large_dataset()
profiler.disable()

stats = pstats.Stats(profiler)
stats.sort_stats('cumulative')
stats.print_stats(20)    # top 20 bottlenecks

# Line profiler (pip install line_profiler)
# @profile decorator on function, then: kernprof -l -v script.py
```

### Memory Optimization
```python
# Use __slots__ to reduce memory overhead for many instances
class Trade:
    __slots__ = ['trade_id', 'symbol', 'quantity', 'price']  # no __dict__ overhead
    def __init__(self, trade_id, symbol, quantity, price):
        self.trade_id = trade_id
        self.symbol = symbol
        self.quantity = quantity
        self.price = price

# Generator vs list – process 10M rows without loading all in memory
def read_trades_chunked(filepath, chunk_size=10_000):
    with open(filepath) as f:
        chunk = []
        for line in f:
            chunk.append(line.strip().split(','))
            if len(chunk) == chunk_size:
                yield chunk
                chunk = []
        if chunk:
            yield chunk

for batch in read_trades_chunked('huge_trades.csv'):
    process_batch(batch)   # only chunk_size rows in memory

# sys.getsizeof() to check object size
import sys
lst = list(range(1_000_000))
gen = (x for x in range(1_000_000))
print(sys.getsizeof(lst))  # ~8MB
print(sys.getsizeof(gen))  # ~120 bytes!
```

### NumPy for Numerical Performance
```python
import numpy as np

# Vectorized operations (C speed, no Python loop)
prices = np.array([175.50, 140.20, 310.75, 189.00])
returns = np.diff(prices) / prices[:-1]   # % returns vectorized

# Much faster than list comprehension for numeric work
n = 1_000_000
prices_list = [1.0 + i*0.001 for i in range(n)]
prices_arr  = np.arange(1.0, 1001.0, 0.001)

# Benchmark: np.sum is ~100x faster than sum() on large arrays
%timeit sum(prices_list)     # ~30ms
%timeit np.sum(prices_arr)   # ~0.3ms
```

### String Optimization
```python
# BAD: string concatenation in loop (creates new string each time)
result = ""
for symbol in symbols:
    result += symbol + ","

# GOOD: join
result = ",".join(symbols)

# GOOD: format strings (f-strings are fastest)
msg = f"Symbol: {symbol}, Price: {price:.2f}"
```

---

## 3. Database with Python (cx_Oracle)

```python
import cx_Oracle

# Connection
dsn = cx_Oracle.makedsn("oracle.ibkr.com", 1521, service_name="PROD")
conn = cx_Oracle.connect(user="ibkr_user", password="secret", dsn=dsn)
cursor = conn.cursor()

# Query with bind variables (SQL injection safe)
cursor.execute(
    "SELECT symbol, price FROM market_data WHERE exchange = :1 AND price > :2",
    ("NYSE", 100.0)
)
rows = cursor.fetchall()    # fetch all
# OR fetchmany(1000) for large result sets

# Bulk insert (executemany – orders of magnitude faster than loop)
trades = [("AAPL", 100, 175.50), ("GOOGL", 50, 140.20), ("MSFT", 200, 310.00)]
cursor.executemany(
    "INSERT INTO trades (symbol, quantity, price) VALUES (:1, :2, :3)",
    trades
)
conn.commit()

# Stored procedure call
result_var = cursor.var(cx_Oracle.STRING)
cursor.callproc("load_market_data", ["AAPL", 175.50, result_var])
print(result_var.getvalue())   # 'SUCCESS'

cursor.close()
conn.close()
```

### Connection Pool (production)
```python
# Create pool once at startup
pool = cx_Oracle.SessionPool(
    user="ibkr_user", password="secret", dsn=dsn,
    min=5, max=20, increment=1, threaded=True
)

# Use from pool
with pool.acquire() as conn:
    with conn.cursor() as cursor:
        cursor.execute("SELECT 1 FROM dual")

# Close pool on shutdown
pool.close()
```

---

## 4. File Processing & Data Handling

```python
import csv, json
from pathlib import Path
from contextlib import contextmanager

# CSV processing
with open('trades.csv', newline='') as f:
    reader = csv.DictReader(f)   # header row becomes dict keys
    for row in reader:
        print(row['symbol'], float(row['price']))

# Write CSV
with open('output.csv', 'w', newline='') as f:
    writer = csv.DictWriter(f, fieldnames=['symbol', 'price', 'ts'])
    writer.writeheader()
    writer.writerows([{'symbol': 'AAPL', 'price': 175.50, 'ts': '2024-03-10'}])

# JSON
data = json.loads('{"symbol": "AAPL", "price": 175.50}')
json_str = json.dumps(data, indent=2)

# Large JSON (streaming)
import ijson  # pip install ijson
with open('large_file.json', 'rb') as f:
    for trade in ijson.items(f, 'trades.item'):   # stream parse
        process(trade)

# Path operations
root = Path('/data/trades')
for csv_file in root.glob('**/*.csv'):
    print(csv_file.name, csv_file.stat().st_size)
```

---

## 5. Concurrency in Python

### The GIL (Global Interpreter Lock)
```
Python's GIL: only ONE thread executes Python bytecode at a time.

→ Threading: good for I/O-bound tasks (DB queries, API calls, file I/O)
→ Multiprocessing: good for CPU-bound tasks (data crunching, computation)
→ asyncio: good for async I/O (thousands of concurrent connections)
```

### Threading (I/O bound – DB queries)
```python
from concurrent.futures import ThreadPoolExecutor, as_completed

def fetch_symbol_data(symbol):
    # DB query or API call (I/O bound)
    return db.query("SELECT * FROM market_data WHERE symbol = :1", symbol)

symbols = ['AAPL', 'GOOGL', 'MSFT', 'AMZN', 'TSLA']
with ThreadPoolExecutor(max_workers=10) as executor:
    futures = {executor.submit(fetch_symbol_data, s): s for s in symbols}
    for future in as_completed(futures):
        symbol = futures[future]
        data = future.result()
        print(f"{symbol}: {data}")
```

### Multiprocessing (CPU bound)
```python
from multiprocessing import Pool
import pandas as pd

def process_chunk(filepath):
    df = pd.read_csv(filepath)
    return df.groupby('symbol')['value'].sum().to_dict()

files = [f'/data/chunk_{i}.csv' for i in range(8)]
with Pool(processes=8) as pool:
    results = pool.map(process_chunk, files)

# Merge results
combined = {}
for r in results:
    for k, v in r.items():
        combined[k] = combined.get(k, 0) + v
```

### asyncio (async I/O)
```python
import asyncio, aiohttp

async def fetch_price(session, symbol):
    async with session.get(f'http://api.example.com/price/{symbol}') as resp:
        return await resp.json()

async def main():
    async with aiohttp.ClientSession() as session:
        tasks = [fetch_price(session, s) for s in ['AAPL','GOOGL','MSFT']]
        results = await asyncio.gather(*tasks)
        for r in results:
            print(r)

asyncio.run(main())
```

---

## 6. Common Libraries for Data Systems

| Library | Purpose | IBKR Use Case |
|---|---|---|
| `cx_Oracle` | Oracle DB | Primary DB operations |
| `pandas` | Data analysis | Trade data aggregation, reports |
| `numpy` | Numerical computing | Price calculations, risk metrics |
| `kafka-python` | Kafka client | Consume market data streams |
| `sqlalchemy` | ORM / DB toolkit | DB abstraction |
| `requests/aiohttp` | HTTP client | REST API calls |
| `pydantic` | Data validation | Validate API payloads |
| `logging` | Logging | Structured application logs |
| `pytest` | Testing | Unit and integration tests |

---

## 7. Common Interview Questions

| Question | Key Answer |
|---|---|
| **What is the GIL?** | Global Interpreter Lock – prevents true parallel execution of Python threads; use multiprocessing for CPU-bound work |
| **`list` vs `tuple` vs `set`?** | List: mutable ordered; Tuple: immutable ordered (faster memory); Set: unordered unique (O(1) lookup) |
| **What is a generator?** | Function using `yield`; produces values lazily; memory efficient for large data |
| **`deepcopy` vs `copy`?** | `copy`: shallow copy (nested objects still shared); `deepcopy`: fully independent copy |
| **What is `*args` and `**kwargs`?** | `*args`: variable positional args (tuple); `**kwargs`: variable keyword args (dict) |
| **How does Python's `dict` work?** | Hash table internally; O(1) average get/set; ordered by insertion order (Python 3.7+) |
| **What is a context manager?** | Implements `__enter__`/`__exit__` or uses `@contextmanager`; ensures cleanup (e.g., `with open(...)`) |
| **How to profile Python code?** | `cProfile` for function-level; `line_profiler` for line-level; `memory_profiler` for memory |
| **What is `functools.lru_cache`?** | Memoization decorator; caches function results by args; great for repeated expensive computations |

```python
# lru_cache example
from functools import lru_cache

@lru_cache(maxsize=1000)
def get_instrument_details(isin: str) -> dict:
    return db.query_instrument(isin)  # only hits DB once per ISIN
```
