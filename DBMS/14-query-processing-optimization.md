# Query Processing and Optimization

## What is it?

Query processing is the multi-stage pipeline by which a DBMS translates a declarative SQL query string into an executable sequence of physical low-level database operations. The **Query Optimizer** is the "brain" of the database, evaluating thousands of candidate execution plans and selecting the plan with the lowest estimated I/O and CPU cost.

## Simple Explanation

When you order a ride on Uber:
- You don't tell the driver which exact streets, turns, and highway lanes to take; you simply state the destination (**Declarative SQL**).
- Uber's navigation software analyzes traffic reports, toll costs, and road closures to compute the fastest route (**Cost-Based Query Optimizer**).
- The driver turns the steering wheel and accelerates along the chosen route (**Execution Engine**).

## Hinglish Explanation

Jab aap SQL query likhte ho, toh database directly execute nahi karta:
1. **Parser**: Syntax check karta hai aur query tree banata hai.
2. **Optimizer**: Sabse important component. Yeh mathematical rules (RBO) aur table statistics (CBO) use karke decide karta hai ki kaunsa Join algorithm chalana hai aur kaunsa Index use karna hai.
3. **Execution Engine (Volcano Model)**: Physical operators (`Index Scan`, `Hash Join`) ko pipeline mein execute karta hai.
Sabse popular interview topic hai: **Nested Loop Join vs. Hash Join vs. Sort-Merge Join** aur `EXPLAIN ANALYZE` read karna.

## Why Do We Need It?

A single SQL query with 4 joins can be executed in over $4! \times 3^3 = 648$ different physical ways. A naive plan might take 4 hours to run a Full Table Scan, while an optimized plan using Hash Joins and Index Scans executes in 12 milliseconds on the exact same hardware.

## How It Works

### The Query Processing Pipeline

```text
       SQL Query String: "SELECT * FROM orders WHERE amount > 100;"
                                  │
                                  ▼
      ┌────────────────────────────────────────────────────────┐
      │   Parser & Lexer (Syntax check ➔ Abstract Syntax Tree) │
      └───────────────────────────┬────────────────────────────┘
                                  │
                                  ▼
      ┌────────────────────────────────────────────────────────┐
      │  Catalog Binder (Semantic check: tables/columns exist) │
      └───────────────────────────┬────────────────────────────┘
                                  │
                                  ▼
      ┌────────────────────────────────────────────────────────┐
      │  Logical Optimizer (Relational Algebra Rewrites / RBO)  │
      │  - Pushdown selections (Filter rows early)             │
      │  - Eliminate redundant joins / subqueries              │
      └───────────────────────────┬────────────────────────────┘
                                  │
                                  ▼
      ┌────────────────────────────────────────────────────────┐
      │  Physical Optimizer (Cost-Based Optimizer - CBO)       │
      │  - Uses Catalog Statistics & Histograms                │
      │  - Chooses: Index Scan vs. Seq Scan                    │
      │  - Chooses: Nested Loop vs. Hash Join vs. Merge Join   │
      └───────────────────────────┬────────────────────────────┘
                                  │
                                  ▼
      ┌────────────────────────────────────────────────────────┐
      │  Execution Engine (Volcano Iterator: open, next, close)│
      └────────────────────────────────────────────────────────┘
```

---

### Physical Join Algorithms Comparison

#### 1. Nested Loop Join
For each row in outer table $R$, scan inner table $S$:
- **Cost**: $O(|R| \times |S|)$.
- **Best For**: Small outer table joined with an inner table having an **index** on the join key (**Index Nested Loop Join**).

#### 2. Hash Join
1. **Build Phase**: Read smaller table into an in-memory hash table on the join key.
2. **Probe Phase**: Scan larger table, hashing each join key to probe the in-memory table.
- **Cost**: $O(|R| + |S|)$ linear time!
- **Best For**: Large, unindexed tables with equi-joins (`R.id = S.id`).

#### 3. Sort-Merge Join
1. Sort both tables on the join key (if not already sorted by an index).
2. Walk both sorted lists simultaneously like a two-pointer merge.
- **Cost**: $O(|R|\log|R| + |S|\log|S|)$ for sorting, then $O(|R| + |S|)$ for merge.
- **Best For**: Inputs already sorted by B+ Tree clustered index, or non-equi joins (`<`, `>`).

---

### Understanding `EXPLAIN` and `EXPLAIN ANALYZE` (PostgreSQL)

```text
EXPLAIN ANALYZE SELECT * FROM users u JOIN orders o ON u.id = o.user_id WHERE u.age > 30;

Hash Join  (cost=15.20..42.50 rows=500 width=72) (actual time=0.82..3.14 rows=480 loops=1)
  Hash Cond: (o.user_id = u.id)
  ->  Seq Scan on orders o  (cost=0.00..20.00 rows=1000 width=36)
  ->  Hash  (cost=10.00..10.00 rows=200 width=36)
        ->  Bitmap Heap Scan on users u  (cost=4.20..10.00 rows=200 width=36)
              Recheck Cond: (age > 30)
              ->  Bitmap Index Scan on idx_users_age  (cost=0.00..4.15 rows=200 width=0)
Planning Time: 0.18 ms | Execution Time: 3.45 ms
```
- **`cost=15.20..42.50`**: Startup cost (15.20) to total estimated cost (42.50) in disk I/O fetch units.
- **`actual time=0.82..3.14`**: Measured time in milliseconds during true execution.
- **`Bitmap Index Scan`**: Collects matching page IDs from the index into a bitmask, then fetches only those pages in sequential order.

## Important Terminology

| Term | Meaning |
|---|---|
| **Cost-Based Optimizer (CBO)**| Evaluates candidate plans using estimated I/O, CPU, and statistics |
| **Cardinality Estimation** | Estimating how many rows a filter or join will output |
| **Data Distribution Histogram**| Statistical frequency buckets of column values stored in system catalog |
| **Volcano Iterator Model** | Standard pull-based execution engine (`open()`, `next()`, `close()`) |
| **Predicate Pushdown** | Optimizing by applying `WHERE` filters before performing expensive joins |

## Real-World Example: Optimizer Failure from Stale Statistics

In PostgreSQL or MySQL:
- If a table undergoes millions of insertions or deletions without running `ANALYZE`, the catalog statistics remain stale.
- The CBO may assume the table contains only 10 rows and choose a slow Nested Loop Join instead of a Hash Join, causing query latency to spike from 10 ms to 45 seconds.
- **Fix**: Run `ANALYZE table_name;` or configure proactive auto-vacuuming.

## Important Properties

| Join Algorithm | Pre-conditions | Memory Requirement | Complexity |
|---|---|---|---|
| **Nested Loop** | None (works on any condition) | Minimal ($O(1)$) | $O(|R| \times |S|)$ |
| **Index Nested Loop**| Inner table has index on join key| Minimal | $O(|R| \log |S|)$ |
| **Hash Join** | Equi-joins only (`=`) | High (needs hash table in RAM) | **$O(|R| + |S|)$** |
| **Sort-Merge Join** | Equi-joins or inequality (`<`, `>`)| Moderate (sorting buffer) | $O(N \log N)$ or linear if sorted |

### Common Misconceptions

* `EXPLAIN` does NOT execute your query; it only prints the optimizer's *estimated plan*. `EXPLAIN ANALYZE` **actually executes** the query and returns real timings (be careful running `EXPLAIN ANALYZE DELETE ...` in production!).
* Hash Joins cannot handle range joins like `ON a.val BETWEEN b.start AND b.end`; hash tables require exact key equality.
* Higher database CPU does not always mean poor queries; sometimes an index is missing, forcing the CPU to evaluate millions of hash comparisons on every scan.

### What Interviewers Usually Test

1. Comparing Nested Loop, Hash Join, and Sort-Merge Join trade-offs.
2. Reading and interpreting an `EXPLAIN ANALYZE` output.
3. Explaining Predicate Pushdown and Relational Algebra rewrites.
4. Why database statistics and histograms matter in Cost-Based Optimization.

### Common Follow-up Questions

1. What happens in a Hash Join if the build table is too large to fit into RAM?
2. What is the difference between `EXPLAIN` and `EXPLAIN ANALYZE`?
3. How does the Volcano Iterator Model work?
4. What is a Bitmap Index Scan in PostgreSQL?
5. How does a database query optimizer decide whether to use an index or a sequential table scan?

### Follow-up Answers

1. **Grace Hash Join**: Both tables are partitioned using a hash function into multiple disk buckets; the engine joins matching partition buckets in pairs using available memory.
2. `EXPLAIN` displays the query planner's estimated cost without running the query; `EXPLAIN ANALYZE` actually executes the query, measures wall-clock time, and reports actual row counts vs. estimates.
3. Each physical relational operator (Scan, Filter, Join) is an iterator implementing `open()`, `next()`, and `close()`. The parent operator calls `next()` on its children, streaming tuples row-by-row in a pipeline without storing entire intermediate tables in RAM.
4. A two-stage scan: the index is first scanned to build a memory bitmap of physical pages containing matches; the engine then visits the table heap in sequential page order, avoiding random I/O.
5. If the estimated selectivity indicates more than ~5-15% of the table rows match, the CBO often abandons the index in favor of a Sequential Scan because sequential block reads are significantly faster than thousands of random index lookups.
