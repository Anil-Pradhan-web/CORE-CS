# 19. MVCC & Transaction Isolation Levels

## What is it?
**Multi-Version Concurrency Control (MVCC)** is the state-of-the-art concurrency control paradigm used by modern relational database engines (PostgreSQL, MySQL InnoDB, Oracle). Instead of overwriting data in-place and acquiring blocking read locks, MVCC maintains multiple historical versions of each record. This enables the golden database guarantee: **"Readers never block Writers, and Writers never block Readers."**
The **ANSI SQL Isolation Levels** define the degree to which a transaction is isolated from concurrent modifications by other transactions.

---

## Simple Explanation
Imagine a newspaper editing room:
- **Lock-based system (Single version)**: If the editor wants to proofread page 3, no journalist can write or edit page 3 until the editor puts the paper down. Everyone waits.
- **MVCC (Multi-version)**: When the journalist starts writing "Edition 2", the editor continues reading a printed snapshot of "Edition 1". The editor is never blocked by the journalist's typing, and the journalist does not wait for the editor. Only when Edition 2 is officially published does the new snapshot become visible to future readers.

---

## Hinglish Explanation
Traditional database engines mein agar koi user `SELECT * FROM orders` chala deta tha jo 10 minute leta tha, toh poora orders table lock ho jata tha—koi naya order `INSERT` ya `UPDATE` nahi ho sakta tha!
- **MVCC** is problem ko khatam karta hai:
  - Jab koi query row modify karti hai, purani row overwrite nahi hoti. Ek nayi version row ban jati hai.
  - Har row ke paas hidden metadata hota hai: `xmin` (kis transaction ne create kiya) aur `xmax` (kis transaction ne delete/replace kiya).
  - Jab koi reader query aati hai, usko database ka ek point-in-time **Snapshot (Read View)** de diya jata hai. Reader sirf wahi versions dekhta hai jo uske snapshot time se pehle commit ho chuki thi.
  - Is wajah se **Readers writers ko block nahi karte, aur writers readers ko block nahi karte**.
- **Isolation Levels**: Char standard levels hote hain: `READ UNCOMMITTED`, `READ COMMITTED`, `REPEATABLE READ`, aur `SERIALIZABLE`. Higher level par data correctness badhti hai lekin concurrency aur performance thoda kam ho jati hai.

---

## Why Do We Need It?
1. **Massive Read Scalability**: Modern web workloads are 90%+ reads (browsing catalogs, feeds, dashboards). MVCC allows millions of concurrent reads to execute with zero lock contention against active writes.
2. **Eliminating Deadlocks between Readers and Writers**: Under pure 2PL, readers holding Shared locks frequently deadlock with writers seeking Exclusive locks. MVCC removes read locks entirely.
3. **Customizable Latency vs Consistency Trade-offs**: Applications can select lightweight isolation (`READ COMMITTED` for analytics/feeds) or strict isolation (`SERIALIZABLE` for financial balances).

---

## How It Works

### 1. MVCC Row Versioning Internal Mechanics

#### PostgreSQL Tuple Versioning (Heap-Based)
In PostgreSQL, all versions live directly in the data table (Heap):
```
                  Tuple Heap Page
┌─────────────────────────────────────────────────────────────┐
│ Header: [xmin = 100, xmax = 105, ctid = (0,2)]             │
│ Data:   id = 1, name = "Alice", balance = 500              │ (Old Version)
├─────────────────────────────────────────────────────────────┤
│ Header: [xmin = 105, xmax = 0,   ctid = (0,2)] ◄────────────┤
│ Data:   id = 1, name = "Alice", balance = 700               │ (Current Version)
└─────────────────────────────────────────────────────────────┘
```
- When $T_{105}$ executes `UPDATE balance = 700`, it writes a brand new tuple with `xmin = 105`.
- The old tuple's `xmax` is set to `105`, pointing to the new version via `ctid`.
- Unused dead tuples are later cleaned up by the **VACUUM** background process.

#### MySQL InnoDB Versioning (Rollback Segment / Undo Log)
In InnoDB, only the latest version lives in the clustered index leaf page. Older versions are reconstructed dynamically from the **Undo Log**:

```
 Clustered Index Leaf                     Undo Log Chain in Memory/Disk
┌─────────────────────────┐               ┌────────────────────────┐
│ id: 1                   │               │ Undo Record (TID: 100) │
│ balance: 700            │               │ balance = 500          │
│ DB_TRX_ID: 105          │               │ DB_ROLL_PTR: NULL      │
│ DB_ROLL_PTR: ───────────┼──────────────►└────────────────────────┘
└─────────────────────────┘
```
- If a query started before $T_{105}$ committed, it follows the `DB_ROLL_PTR` into the Undo Log to read the older value `balance = 500`.

---

### 2. The Four ANSI SQL Isolation Levels

```
Isolation Level Hierarchy:
   READ UNCOMMITTED   (Weakest, Highest Concurrency, Dirty Reads allowed)
          ▲
          │
   READ COMMITTED     (Postgres/Oracle Default, No Dirty Reads)
          ▲
          │
   REPEATABLE READ    (MySQL InnoDB Default, Snapshot Isolation)
          ▲
          │
   SERIALIZABLE       (Strongest, Full Serializability, Write-Skew Free)
```

#### Snapshot Creation Comparison
- **`READ COMMITTED`**: Takes a fresh snapshot **at the start of every single SQL statement**. If another transaction commits changes between query 1 and query 2 within the same transaction, query 2 sees the new data (**Non-Repeatable Read**).
- **`REPEATABLE READ`**: Takes a snapshot **once at the start of the first query in the transaction**. All subsequent queries throughout the transaction see the exact same snapshot, ignoring any external commits.

---

### 3. Isolation Levels vs Concurrency Anomalies

| Isolation Level | Dirty Read | Non-Repeatable Read | Phantom Read | Write Skew |
| :--- | :---: | :---: | :---: | :---: |
| **Read Uncommitted** | **Allowed** | **Allowed** | **Allowed** | **Allowed** |
| **Read Committed** | Prevented | **Allowed** | **Allowed** | **Allowed** |
| **Repeatable Read** | Prevented | Prevented | Prevented (in InnoDB/PG)* | **Allowed** |
| **Serializable** | Prevented | Prevented | Prevented | Prevented |

*\*Note: In MySQL InnoDB, Repeatable Read prevents phantoms for standard SELECT via MVCC snapshots, and for `SELECT ... FOR UPDATE` via Next-Key (Gap) Locks. However, it is still vulnerable to Write Skew!*

---

### 4. Write Skew: The Classic Repeatable Read Flaw

**The On-Call Doctor Problem**:
- *Invariant Rule*: "At all times, at least one doctor must remain on call."
- *Initial State*: Both Dr. Alice and Dr. Bob are active on call.
- Dr. Alice ($T_1$) and Dr. Bob ($T_2$) both request leave at the exact same moment.

```
T1 (Alice): SELECT COUNT(*) FROM doctors WHERE on_call = TRUE; -- Returns 2 (OK!)
T2 (Bob):   SELECT COUNT(*) FROM doctors WHERE on_call = TRUE; -- Returns 2 (OK!)

T1 (Alice): UPDATE doctors SET on_call = FALSE WHERE name = 'Alice';
T2 (Bob):   UPDATE doctors SET on_call = FALSE WHERE name = 'Bob';

T1 (Alice): COMMIT;
T2 (Bob):   COMMIT;
```
**Catastrophe**: Both transactions commit under `REPEATABLE READ` without error because neither modified the same row (Alice touched Alice's row; Bob touched Bob's row). But the invariant is broken: **0 doctors are on call!**

#### Solving Write Skew
1. **Pessimistic Locking**: Use `SELECT ... FOR UPDATE` to lock all matching rows across the condition.
2. **Serializable Isolation Level**: Use **Serializable Snapshot Isolation (SSI)** (Postgres `SET TRANSACTION ISOLATION LEVEL SERIALIZABLE`). SSI detects overlapping read-write dependencies (rw-antidependency cycles) and automatically aborts one transaction with:
   `ERROR: could not serialize access due to read/write dependencies among transactions`.

---

## Important Terminology

| Term | Definition |
| :--- | :--- |
| **Read View / Snapshot** | Logical filter defining which active transaction IDs are visible to a reader query. |
| **`xmin` / `xmax`** | System column fields in PostgreSQL tracking creation transaction ID and deletion/update transaction ID. |
| **Dead Tuple** | An obsolete version of a row no longer visible to any active transaction's snapshot. |
| **Vacuum / Purge** | Garbage collection mechanism reclaiming disk storage occupied by dead row versions. |
| **Write Skew** | Anomaly occurring under Repeatable Read where concurrent transactions read overlapping data, update disjoint data, and violate a global invariant. |
| **Serializable Snapshot Isolation (SSI)** | Modern serializability engine tracking SIREAD lock dependencies without blocking readers. |
| **Next-Key Locking** | InnoDB lock algorithm combining record lock + gap lock to prevent phantom inserts under Repeatable Read. |

---

## Example: Configuring Isolation Levels in SQL

```sql
-- Check current session isolation level (PostgreSQL)
SHOW transaction_isolation;

-- Set transaction isolation to Serializable
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;

-- Example: Safe check-and-update pattern
SELECT balance FROM accounts WHERE id = 101;

UPDATE accounts SET balance = balance - 200 WHERE id = 101;

COMMIT;
```

---

## Real-World Example: PostgreSQL VACUUM & Table Bloat
In high-throughput PostgreSQL systems (e.g., millions of updates/minute in payment tracking):
- If long-running analytical queries hold open transactions for hours:
  ```sql
  -- Analyst runs heavy query at 10:00 AM
  BEGIN;
  SELECT * FROM massive_audit_log WHERE timestamp > '2025-01-01';
  -- Session left open indefinitely!
  ```
- Because this transaction's snapshot needs old versions, PostgreSQL's **Autovacuum** cannot clean *any* dead tuples generated after 10:00 AM across the entire database!
- **Table Bloat**: Disk usage skyrockets from 10 GB to 200 GB, indexes bloat, and cache hit ratio crashes.
- *Production Rule*: Always configure `idle_in_transaction_session_timeout = '60s'` and monitor `pg_stat_activity` for stale transactions.

---

## Important Properties

| Feature | PostgreSQL MVCC | MySQL InnoDB MVCC |
| :--- | :--- | :--- |
| **Storage Location for Old Versions** | Data Heap (in-table dead tuples) | Undo Log Segments (tablespace) |
| **Update Overhead** | Writes full new row to heap + index updates (unless HOT) | Modifies row in clustered index; appends delta to undo log |
| **Garbage Collection** | `VACUUM` / `AUTOVACUUM` background worker | `Purge` background threads truncate undo segments |
| **Table Bloat Risk** | High if autovacuum lags or transactions stay idle | Low (heap does not bloat; undo log tablespace may grow) |
| **Index Traversal** | Indexes point to Heap Tuples; requires visibility checks | Clustered index primary key directly points to latest row |

---

### Common Misconceptions
1. *Misconception*: "Repeatable Read prevents all concurrency anomalies."
   - *Reality*: **False**. Repeatable Read prevents Dirty Reads, Non-Repeatable Reads, and (in most modern engines) Phantom Reads. However, it still allows **Write Skew** anomalies. Only `SERIALIZABLE` prevents all anomalies.
2. *Misconception*: "`READ COMMITTED` means a transaction reads data that will be committed in the future."
   - *Reality*: **False**. `READ COMMITTED` guarantees a query will *only* see rows whose transactions have already officially committed prior to the statement start. It never reads uncommitted dirty data.

---

### What Interviewers Usually Test
- **Postgres vs InnoDB MVCC Implementation**: Heap dead tuples + VACUUM vs Clustered Index + Undo Log pointer chain.
- **Statement-Level vs Transaction-Level Snapshots**: Why `READ COMMITTED` allows non-repeatable reads (fresh snapshot per statement) while `REPEATABLE READ` prevents it (single snapshot per transaction).
- **Explain Write Skew with an Example**: The 2-doctor on-call scenario or black/white marble inventory scenario.

---

### Common Follow-up Questions

1. How does PostgreSQL HOT (Heap-Only Tuples) optimization reduce MVCC indexing overhead?
2. Why does `COUNT(*)` without a `WHERE` clause require a full table scan in PostgreSQL under MVCC?
3. How does MySQL InnoDB prevent phantom rows under Repeatable Read without SSI?
4. What is a "Snapshot Isolation Anomaly", and how does SSI detect it without blocking reads?
5. What happens when an `UPDATE` query under `READ COMMITTED` finds a row currently locked by another transaction?

---

### Follow-up Answers

1. **Heap-Only Tuples (HOT)**: Normally, inserting a new row version requires updating all secondary indexes to point to the new physical tuple address (`ctid`). With HOT, if the update does not modify indexed columns and free space exists on the same page, PG creates an in-page pointer chain. Secondary indexes continue pointing to the root tuple, saving immense I/O.
2. **Postgres `COUNT(*)` Cost**: Because PostgreSQL stores row versions directly in the heap, a row's existence does not guarantee it is visible to the current transaction's snapshot. PG must inspect the `xmin`/`xmax` flags of every row (or check visibility maps) to determine if it is alive for that query. MySQL InnoDB avoids this only for MyISAM (which lacks MVCC/transactions).
3. **InnoDB Gap Locks / Next-Key Locks**: When executing locking reads (`SELECT ... FOR UPDATE`), InnoDB places locks not only on physical records, but also on the "gaps" (open intervals) between indexed keys. Any concurrent transaction attempting to `INSERT` a row into that gap is blocked until the locking transaction commits.
4. **SSI (Serializable Snapshot Isolation)**: SSI uses in-memory lock trackers called `SIREAD` locks. If Transaction $T_1$ reads data that Transaction $T_2$ overwrites, and $T_1$ subsequently writes data that affects $T_2$, SSI detects this dangerous directed dependency cycle ($T_1 \xrightarrow{rw} T_2 \xrightarrow{rw} T_1$) and aborts one transaction before commit.
5. **Row Lock Wait in Read Committed**: Under `READ COMMITTED`, the updating query blocks until the locking transaction finishes. Once the lock is released: if the prior transaction committed, the waiting query **re-evaluates** its `WHERE` clause against the newly committed version of that row. If the condition still matches, it updates the new version.
