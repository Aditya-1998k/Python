### DuckDB is the New Pandas
- Speaker: S Anand
- Must Visit Anand Blog : [Blog](https://www.s-anand.net/blog/)

**Introduction**
- `Pandas` = great for small/medium datasets, but struggles with scale.
- `DuckDB` = `SQLite for Analytics` → embedded, fast, SQL-native, works in memory or on disk.

**Core Features of DuckDB**
- In-memory & on-disk processing (hybrid).
- `Vectorized execution` → faster than Pandas row-by-row ops.
- `Multi-core parallelism` by default.
- `SQL-first`: works with familiar SQL, no need for complex Pandas chaining.
- `Interoperability`: Python, R, Java, Node.js, WASM (runs in browser).
- Data source connectors: Parquet, CSV, JSON, Arrow, SQL Server, Postgres, S3, etc.
- Tiny `footprint` (single binary, no server setup).

**DuckDB vs Pandas vs SQL Server**

**Pandas**
- Pros: Simple API, strong ecosystem.
- Cons:
  1. Single-threaded
  2. Memory-bound
  3. Complex syntax for joins/aggregations

**SQL Server**
- Pros: Optimized for transactional queries, indexing, mature.
- Cons:
  1. Needs server setup/admin
  2. Expensive licensing
  3. Overkill for ad-hoc analytics

**DuckDB**
Pros:
1. Faster than Pandas (esp. joins, groupbys)
2. No server overhead (embedded like SQLite)
3. Can query directly from files (e.g., parquet, csv)

SQL syntax → more universal than Pandas API

Cons:
1. Younger ecosystem vs. Pandas
2. More SQL mindset needed

**Performance Comparisons**
```
| Task                   | Pandas             | SQL Server             | DuckDB                 |
| ---------------------- | ------------------ | ---------------------- | ---------------------- |
| Large CSV load (GBs)   | Slow, memory hog   | Faster but needs setup | Blazing fast, no setup |
| Join (10M+ rows)       | Struggles          | Good w/ indexes        | Comparable or faster   |
| Aggregations (groupby) | Slow (single-core) | Parallelizable         | Fast, multi-core       |
| On-disk Parquet        | Needs pyarrow      | ETL required           | Native support         |
```

Demo:

DuckDB handled multi-GB datasets on a laptop without choking memory, while Pandas either crashed or slowed dramatically.

Example Workflow
Pandas
```python
import pandas as pd

df = pd.read_csv("big.csv")
result = df.groupby("category")["value"].mean()
```

DuckDB
```
import duckdb

result = duckdb.query("""
    SELECT category, AVG(value) 
    FROM 'big.csv'
    GROUP BY category
""").to_df()
```

**Future Trends**
DuckDB becoming the default analytical backend for:
1. Jupyter notebooks
2. Data science pipelines
3. Browser-based analytics (via WASM)
4. LLMs will help auto-generate SQL → you don’t need to “learn DuckDB” deeply.
5. Possible replacement of Pandas for many data exploration + ETL tasks.

Use DuckDB when:
1. Data is too big for Pandas but too small to justify full DB setup.
2. You want fast, SQL-native analytics inside Python.
3. Still keep Pandas for quick manipulations + ecosystem libraries.

Thi
nk of DuckDB as “SQLite for analytics” → fast, light, everywhere.
