# 24. DBMS in System Design & Production Best Practices

## What is it?
In production software engineering, building reliable systems requires pairing database theory with real-world infrastructure patterns. **DBMS System Design** encompasses the operational architecture required to keep databases fast, available, secure, and resilient under massive production scale. This includes **Connection Pooling**, **Caching Strategies**, **Read-Write Splitting & CQRS**, **Zero-Downtime Schema Migrations (Expand-Contract)**, and **SQL Injection Prevention**.

---

## Simple Explanation
Imagine running a busy doctor's clinic:
- **Connection Pooling**: Instead of building a brand-new examination room and hiring a new receptionist every single time a patient arrives at the front door (opening a fresh TCP socket and forking an OS process), you maintain 10 permanently open examination rooms. Patients enter, get treated, and step out so the next patient can use the same room.
- **Caching (Cache-Aside)**: The receptionist keeps a cheat-sheet of answers to the 5 most frequent questions on a desk notepad. If an answer is on the pad, it is answered in 1 second; if not, the receptionist walks to the back archive room to find the file.
- **Expand-Contract Schema Migration**: If you want to replace old patient ID cards with biometric chips, you don't shut the hospital down for 3 days. You support both ID cards and biometric chips simultaneously for 2 months, migrate records in the background, and only retire ID cards once everyone is using chips.

---

## Hinglish Explanation
Interview system design rounds aur high-scale production apps mein database performance sirf queries par depend nahi karti, balki surrounding architecture par depend karti hai:
- **Connection Pooling**: Har database connection ek heavy OS process/thread create karta hai (Postgres mein ~10MB RAM per connection). Agar 5,000 users ek saath connect karein, database crash ho jayega. Hum **HikariCP** ya **PgBouncer** use karke 50-100 reusable connections ka pool banate hain jo thousands of web requests ko smoothly serve karta hai.
- **Caching Patterns**:
  - **Cache-Aside**: App pehle Redis mein dekhta hai. Agar mila $\rightarrow$ **Cache Hit**. Agar nahi mila $\rightarrow$ DB se padho, Redis mein daalo, return karo (**Cache Miss**).
  - **Write-Through**: Pehle cache update karo, fir synchronously DB update karo.
  - **Write-Back (Write-Behind)**: Cache mein likh kar turant client ko OK bol do, background worker asynchronously batch mein DB pe write karega (Extreme write speed, but data loss risk if Redis crashes).
- **Expand & Contract Pattern**: Production mein kabhi bhi `ALTER TABLE users RENAME COLUMN phone TO mobile_number;` mat chalao! Table lock ho jayega aur running app code crash ho jayega. Nayi column add karo, code se dono mein likho, purana data background mein copy karo, read nayi column se switch karo, aur fir purani column drop karo.

---

## Why Do We Need It?
1. **Preventing Connection Starvation & OOM Crashes**: Unbounded client connections exhaust server memory and trigger kernel Out-Of-Memory (OOM) killer terminations.
2. **Sub-Millisecond Read Latency (100x Speedup)**: Caching hot database records in RAM via Redis eliminates expensive disk I/O and B+ Tree traversals.
3. **Zero-Downtime Deployments**: Modern SaaS platforms cannot afford maintenance windows; schema changes must deploy seamlessly while processing live checkout transactions.

---

## How It Works

### 1. Connection Pooling Architecture

```
Application Threads (Tomcat, Gunicorn, Node.js)
  Thread 1   Thread 2   Thread 3   Thread 4 ... Thread 500
      │          │          │          │           │
      ▼          ▼          ▼          ▼           ▼
┌─────────────────────────────────────────────────────────────┐
│                 CONNECTION POOL (e.g., HikariCP)            │
│               [ Conn 1 ][ Conn 2 ] ... [ Conn 20 ]          │
└──────────────────────────────┬──────────────────────────────┘
                               │ 20 Persistent TCP Connections
                               ▼
┌─────────────────────────────────────────────────────────────┐
│                 PostgreSQL / MySQL Server                   │
│   (20 Dedicated Backend Worker Processes - CPU Cache Hot!)  │
└─────────────────────────────────────────────────────────────┘
```

#### The Famous HikariCP Connection Pool Sizing Formula:
$$\text{Pool Size} = \mathbf{T_n \times (\text{Core Count} \times 2) + \text{Effective Spindle Count}}$$
- *Counter-intuitive Truth*: Setting pool size to 1,000 makes a database **slower**, not faster!
- On a 16-core server with SSD storage, a pool size of **32 to 40 connections** yields maximum throughput. More connections cause thrashing, CPU context switching, and cache line invalidation.

---

### 2. Caching Patterns Comparison

```
1. CACHE-ASIDE (Lazy Loading)
   App ──1. Read Request──► Cache (Redis)
    ▲                          │ Miss!
    │──3. Return Data──────────┤
    │                          ▼
    └────2. Read & Cache◄── Database

2. WRITE-THROUGH
   App ──1. Write Data──► Cache ──2. Sync Write──► Database
    ▲                       │
    └──3. Success ACK───────┘

3. WRITE-BACK / WRITE-BEHIND
   App ──1. Write Data──► Cache (Dirty flag)
    ▲                       │
    └──2. Instant ACK───────┤
                            ▼ (Async Queue / Batch)
                         Database
```

#### Cache Stampede (Thundering Herd) Prevention
When a hot cache key (e.g., "world_cup_final_score") expires:
- 10,000 concurrent requests miss the cache at the exact same millisecond.
- All 10,000 queries hit the database simultaneously, causing a database crash (**Cache Stampede**).
- **Solution: Distributed Mutex Locking**:
  Only the first thread acquires a distributed lock (`SETNX lock:score 1 EX 5`) to query the DB and refresh the cache; all other threads wait or return the stale value temporarily.

---

### 3. Zero-Downtime Schema Migration: Expand & Contract Pattern

**Goal**: Rename column `phone_number` to `mobile_number` on a 50-million-row table without downtime.

```
PHASE 1: EXPAND
- Add new column: ALTER TABLE users ADD COLUMN mobile_number VARCHAR(20);
- Code V1: Reads from phone_number, writes to BOTH phone_number AND mobile_number.

PHASE 2: BACKFILL (Background Migration)
- Run batched worker: UPDATE users SET mobile_number = phone_number WHERE mobile_number IS NULL LIMIT 5000;
- Runs off-peak until all historical rows are synchronized.

PHASE 3: SWITCH CODE (Code V2)
- Code V2: Reads from mobile_number, writes to BOTH.
- Code V3: Reads from mobile_number, writes ONLY to mobile_number.

PHASE 4: CONTRACT
- Drop old column: ALTER TABLE users DROP COLUMN phone_number;
```

---

### 4. SQL Injection Prevention: Prepared Statements

SQL Injection occurs when user input is concatenated directly into a query string, tricking the parser into executing malicious SQL commands:

```
VULNERABLE (Dynamic String Concatenation):
String query = "SELECT * FROM users WHERE user = '" + inputUser + "' AND pass = '" + inputPass + "'";
Attacker Input: inputUser = "admin' -- "
Parsed Query:  SELECT * FROM users WHERE user = 'admin' -- ' AND pass = '...'
Result: Everything after -- is ignored as a comment; attacker logs in as admin without password!

SECURE (Parameterized Prepared Statement):
PreparedStatement stmt = conn.prepareStatement("SELECT * FROM users WHERE user = ? AND pass = ?");
stmt.setString(1, inputUser);
stmt.setString(2, inputPass);
ResultSet rs = stmt.executeQuery();
```

#### Why Prepared Statements Are 100% Immune:
1. **Compilation Phase**: The database parses the SQL structure, tokenizes keywords (`SELECT`, `WHERE`), builds the query execution plan, and compiles it **BEFORE** user parameters are inserted.
2. **Parameter Binding Phase**: The database treats parameters strictly as **literal data values (constants)**. Even if the user passes `'; DROP TABLE users; --`, the engine treats the entire string as a literal username text search, never executing it as code!

---

## Important Terminology

| Term | Definition |
| :--- | :--- |
| **Connection Pooling** | Cache of pre-established database connections reused by incoming application threads. |
| **PgBouncer** | Lightweight connection pooler for PostgreSQL operating in Session, Transaction, or Statement pooling mode. |
| **CQRS** | Command Query Responsibility Segregation; separating read models (replicas) from write models (primary). |
| **Cache Stampede** | Sudden surge of concurrent database queries triggered when a high-traffic cache key expires. |
| **Expand-Contract** | Migration pattern breaking destructive schema changes into non-breaking, incremental phased rollouts. |
| **Prepared Statement**| Pre-compiled SQL template separating query code structure from runtime variable parameters. |
| **CDC (Change Data Capture)**| Architecture reading database WAL logs (e.g., Debezium) to stream table row changes to caches/search indexes. |

---

## Example: Production HikariCP Configuration (Java)

```java
import com.zaxxer.hikari.HikariConfig;
import com.zaxxer.hikari.HikariDataSource;

public class DatabaseManager {
    private static HikariDataSource dataSource;

    static {
        HikariConfig config = new HikariConfig();
        config.setJdbcUrl("jdbc:postgresql://db-primary.internal:5432/ecommerce");
        config.setUsername("app_user");
        config.setPassword(System.getenv("DB_PASSWORD"));

        // Production Pool Tuning
        config.setMaximumPoolSize(30);       // Bounded according to CPU cores
        config.setMinimumIdle(10);           // Keep warm connections ready
        config.setIdleTimeout(30000);        // 30 seconds idle release
        config.setConnectionTimeout(3000);   // Fail fast after 3 seconds wait
        config.setMaxLifetime(1800000);      // 30 minutes connection rotation

        // Performance optimizations
        config.addDataSourceProperty("cachePrepStmts", "true");
        config.addDataSourceProperty("prepStmtCacheSize", "250");
        config.addDataSourceProperty("prepStmtCacheSqlLimit", "2048");

        dataSource = new HikariDataSource(config);
    }

    public static HikariDataSource getDataSource() {
        return dataSource;
    }
}
```

---

## Real-World Example: GitHub's Gh-ost Zero-Downtime Migration Tool
Because standard `ALTER TABLE` locks large tables in MySQL:
- GitHub engineered **gh-ost (GitHub's Online Schema Transmogrifier)**:
  1. Creates a ghost table (`_users_gho`) with the desired new schema.
  2. Reads MySQL binary logs (binlog) directly to capture live row changes (INSERT, UPDATE, DELETE) occurring on the original table.
  3. Slowly backfills historical rows from the old table to the ghost table in small chunks without locking.
  4. Swaps table names atomically using MySQL's instant metadata lock:
     ```sql
     RENAME TABLE users TO _users_old, _users_gho TO users;
     ```
  5. The entire schema migration completes with zero table locks and zero application downtime!

---

## Important Properties

| Caching Strategy | Read Latency | Write Latency | Cache Consistency | Risk of Data Loss |
| :--- | :--- | :--- | :--- | :--- |
| **Cache-Aside** | Low on hit; high on miss | Standard DB write | Eventual (relies on TTL / invalidation) | None |
| **Write-Through** | Ultra Low | Higher (sync dual-write) | Strong (Cache matches DB) | None |
| **Write-Back** | Ultra Low | **Ultra Low** (instant ack) | High temporary skew | **High** (if cache node dies before DB flush) |
| **Refresh-Ahead** | Predictably Low | Standard | Strong for predictable access patterns | None |

---

### Common Misconceptions
1. *Misconception*: "More database connections in the connection pool will make our web application faster."
   - *Reality*: **False**. An oversized connection pool degrades performance. When 500 connections compete for 8 CPU cores, the OS spends more time context-switching between threads and thrashing memory than executing queries. A small, tightly bounded pool (~20-40) saturates CPU execution pipelines and delivers highest overall throughput.
2. *Misconception*: "Sanitizing strings with custom regexes or escaping quotes prevents SQL Injection."
   - *Reality*: **False**. Custom regex sanitation is notorious for subtle bypasses (hex encoding, multi-byte unicode tricks, second-order SQL injection). The **ONLY** bulletproof defense is Parameterized Queries / Prepared Statements.

---

### What Interviewers Usually Test
- **Calculating Connection Pool Size**: Why HikariCP recommends $(2 \times \text{cores}) + \text{spindles}$ rather than hundreds of connections.
- **Explain the Expand & Contract Migration Flow**: Walking through the phased rollout of a non-backward-compatible column rename or table split.
- **How Prepared Statements Prevent SQL Injection Under the Hood**: Explaining AST pre-compilation vs parameter data binding.

---

### Common Follow-up Questions

1. What is the difference between PgBouncer's Session Pooling and Transaction Pooling modes?
2. How do you handle cache invalidation when a database row is updated?
3. What is the difference between First-Order and Second-Order SQL Injection?
4. How do you prevent a single runaway analytical query from taking down an entire production OLTP database?
5. What is the "N+1 Query Problem" in ORMs (Hibernate / Django), and how do you resolve it?

---

### Follow-up Answers

1. **PgBouncer Pooling Modes**:
   - **Session Pooling**: A connection is assigned to a client for the entire duration of its TCP connection (safest; supports prepared statements and session variables; least multiplexing).
   - **Transaction Pooling**: A connection is assigned only for the duration of a single `BEGIN ... COMMIT` block. Once committed, the connection returns to the pool for another user (massive scalability; breaks session-level state).
2. **Cache Invalidation Strategies**:
   - **Delete on Update (Recommended)**: Evict the key from Redis (`DEL user:42`) when the DB updates. Next read repopulates cache fresh.
   - **CDC Invalidation**: Debezium captures the WAL update and publishes an event to an eviction worker.
3. **First vs Second-Order SQL Injection**:
   - **First-Order**: The attacker's payload executes immediately in the initial vulnerable query.
   - **Second-Order**: The attacker stores malicious SQL payload into the database (e.g., as their registered name). The payload sits harmlessly until a separate, trusted internal admin script pulls the string and concatenates it into a dynamic query unsafely.
4. **Guarding OLTP from Runaway Queries**:
   - Set statement timeouts: `SET statement_timeout = '3000ms';`.
   - Route all heavy reporting / analytical queries to asynchronous **Read Replicas** or an analytical warehouse (ClickHouse / Snowflake).
   - Enforce database resource governor quotas (cgroups / PostgreSQL `work_mem` limits).
5. **N+1 Query Problem**: An ORM retrieves 100 orders with 1 query (`SELECT * FROM orders LIMIT 100`), then executes 100 separate queries to fetch customer names (`SELECT * FROM customers WHERE id = ?`). It causes 101 round-trips to the DB instead of 1. Resolved using SQL `JOIN` or ORM eager loading (`select_related` in Django, `JOIN FETCH` in JPA/Hibernate).
