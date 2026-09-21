# 18. Lock-Based & Timestamp Protocols

## What is it?
**Lock-Based Protocols** and **Timestamp Protocols** are the two foundational concurrency control families used by database engines to guarantee serializability without manual intervention. Lock-based protocols use concurrency tokens (**Shared Locks** for reading, **Exclusive Locks** for writing) governed by the **Two-Phase Locking (2PL)** rule. Timestamp protocols assign monotonically increasing timestamps to transactions, ordering access by logical age.

---

## Simple Explanation
Imagine a conference room whiteboard:
- **Shared Lock (S-Lock / Read Lock)**: Multiple people can stand together and read what is on the board simultaneously. No one is allowed to erase or write while anyone is reading.
- **Exclusive Lock (X-Lock / Write Lock)**: If one person wants to erase or write new equations, everyone else must step out of the room. Only one writer can hold the marker at a time.
- **Two-Phase Locking (2PL)**: Once you put down even a single lock, you can never pick up another lock! You acquire everything you need (Growing Phase), reach your peak, and then gradually release them (Shrinking Phase). This mathematical rule prevents interleaved operations from creating circular conflicts.

---

## Hinglish Explanation
Jab multiple queries ek hi record ko change ya read karna chahti hain, engine locks lagata hai:
- **Shared Lock (S)**: Read karne ke liye. Ek record par multiple transactions ek saath S-lock le sakte hain.
- **Exclusive Lock (X)**: Write/Update/Delete karne ke liye. Jab tak X-lock kisi ke paas hai, koi doosra na toh padh sakta hai na likh sakta hai.
- **2PL (Two-Phase Locking)**:
  1. **Growing Phase**: Transaction sirf naye locks acquire kar sakta hai; koi bhi lock release nahi kar sakta.
  2. **Shrinking Phase**: Transaction locks release karna shuru karta hai; ek baar pehla lock chhod diya, toh ab koi naya lock nahi le sakta!
- **Strict 2PL (S2PL)**: Real-world databases (MySQL, Postgres) basic 2PL use nahi karte kyunki basic 2PL mein cascading aborts hote hain. Strict 2PL mein **saare Exclusive locks transaction ke commit/abort hone tak hold kiye jaate hain**.

---

## Why Do We Need It?
1. **Enforcing Conflict Serializability Dynamically**: Instead of running complex $O(V+E)$ precedence graph algorithms after execution, 2PL guarantees serializability purely through runtime local lock rules.
2. **Preventing Dirty Overwrites**: Exclusive locks ensure no two transactions modify the same row simultaneously, eliminating lost updates.
3. **Multi-Level Locking Efficiency**: Multiple Granularity Locking (Intention Locks) allows locking an entire 10-million-row table in $O(1)$ time without scanning and locking every single individual row.

---

## How It Works

### 1. Lock Compatibility Matrix

```
                 Lock Requested by T2
                 ┌──────────┬──────────┐
                 │ Shared S │ Excl X   │
┌──────────────┬─┼──────────┼──────────┤
│ Lock Held    │S│  GRANT   │  DENY    │
│ by T1        ├─┼──────────┼──────────┤
│              │X│  DENY    │  DENY    │
└──────────────┴─┴──────────┴──────────┘
```

- **S and S**: Compatible. Multiple readers read concurrently.
- **S and X**: Incompatible. Writer waits for readers to finish; readers wait for writer.
- **X and X**: Incompatible. Writers execute sequentially one at a time.

---

### 2. Two-Phase Locking (2PL) Protocols Comparison

```
Basic 2PL:
  Locks Held ▲        /───────\  <-- Lock Point (Peak)
             │       /         \
             │      /           \
             │     / (Growing)   \ (Shrinking)
             └────┴───────────────┴──────────────► Time
                  Cannot acquire locks after first release!

Strict 2PL (Avoids Cascading Aborts):
  Locks Held ▲        /───────────┐
             │       /            │
             │      /             │ All X-locks held
             │     / (Growing)    │ until COMMIT/ABORT!
             └────┴───────────────┴──────────────► Time
                                COMMIT

Rigorous 2PL / SS2PL (Strong Strict 2PL):
  - BOTH S-locks and X-locks are held until COMMIT/ABORT.
  - Serialization order matches transaction COMMIT order exactly.
```

---

### 3. Multiple Granularity Locking (MGL) & Intention Locks
When locking a table, checking every row for existing locks is $O(N)$. MGL introduces a hierarchy with **Intention Locks**:

```
                       Database (DB)
                            │
                       Table (T)
                            │
                       Page (P)
                            │
                       Row / Tuple (R)
```

#### Intention Lock Semantics
- **IS (Intention Shared)**: Explicit locking will be done at a lower level with Shared locks.
- **IX (Intention Exclusive)**: Explicit locking will be done at a lower level with Exclusive locks.
- **SIX (Shared + Intention Exclusive)**: The entire node is locked in Shared mode, with explicit writes occurring at lower levels.

#### MGL Compatibility Matrix
| | IS | IX | S | SIX | X |
| :---: | :---: | :---: | :---: | :---: | :---: |
| **IS** | Grant | Grant | Grant | Grant | Deny |
| **IX** | Grant | Grant | Deny | Deny | Deny |
| **S** | Grant | Deny | Grant | Deny | Deny |
| **SIX**| Grant | Deny | Deny | Deny | Deny |
| **X** | Deny | Deny | Deny | Deny | Deny |

*Rule to Lock a Row with X-lock*:
1. Acquire `IX` on Database.
2. Acquire `IX` on Table.
3. Acquire `IX` on Page.
4. Acquire `X` on Row.
*If another transaction wants to `LOCK TABLE t WRITE` (X-lock)*, it sees `IX` on the Table and immediately blocks without inspecting individual rows!

---

### 4. Deadlock Handling Schemes

```
Transaction T1 (holds Row A, wants Row B) ◄────┐
                    │                          │ Circular Wait!
                    ▼                          │
Transaction T2 (holds Row B, wants Row A) ─────┘
```

#### A. Deadlock Detection via Wait-For Graph (WFG)
- Background daemon checks WFG periodically (e.g., every 1 second in InnoDB).
- Directed edge $T_1 \to T_2$ means $T_1$ is blocked waiting for a lock held by $T_2$.
- If a cycle is detected, the DBMS chooses a **Victim Transaction** (usually the one with lowest undo log volume or newest timestamp) and executes an automatic `ROLLBACK`.

#### B. Deadlock Prevention: Wait-Die vs Wound-Wait
Based on transaction start timestamps ($TS(T_i)$). Older transaction has smaller timestamp.

```
Scenario: Older transaction T_old requests lock held by Younger transaction T_young

1. WAIT-DIE (Non-preemptive):
   - Old requests Young  ==> Old WAITS.
   - Young requests Old  ==> Young DIES (Aborts & restarts with original timestamp).
   ("Old can wait; Young must die")

2. WOUND-WAIT (Preemptive):
   - Old requests Young  ==> Old WOUNDS Young (Young is aborted and preempted).
   - Young requests Old  ==> Young WAITS.
   ("Old wounds young; Young waits for old")
   *Note: Wound-Wait causes far fewer aborts in practice than Wait-Die!
```

---

### 5. Timestamp-Based Ordering & Thomas Write Rule
Each transaction $T_i$ is assigned a unique timestamp $TS(T_i)$. Each data item $Q$ maintains:
- $W\text{-timestamp}(Q)$: Largest timestamp of any transaction that wrote $Q$.
- $R\text{-timestamp}(Q)$: Largest timestamp of any transaction that read $Q$.

#### Basic Timestamp Ordering Protocol
1. **$T_i$ issues $Read(Q)$**:
   - If $TS(T_i) < W\text{-timestamp}(Q)$, $T_i$ is reading an overwritten value $\implies$ **Abort $T_i$**.
   - Otherwise, allow read and set $R\text{-timestamp}(Q) = \max(R\text{-timestamp}(Q), TS(T_i))$.
2. **$T_i$ issues $Write(Q)$**:
   - If $TS(T_i) < R\text{-timestamp}(Q)$, a younger transaction already read the older value $\implies$ **Abort $T_i$**.
   - If $TS(T_i) < W\text{-timestamp}(Q)$, $T_i$ is attempting to write an obsolete value.
     - Under **Basic Timestamp Ordering**: **Abort $T_i$**.
     - Under **Thomas Write Rule**: Simply **ignore the write** and proceed! (Since a younger transaction has already updated $Q$, this blind write is obsolete).

---

## Important Terminology

| Term | Definition |
| :--- | :--- |
| **Lock Point** | The exact moment a transaction acquires its final lock before entering the shrinking phase. |
| **Strict 2PL** | 2PL protocol where all exclusive (X) locks are held until transaction termination (`COMMIT` or `ABORT`). |
| **Rigorous 2PL (SS2PL)**| 2PL protocol where all shared (S) and exclusive (X) locks are held until termination. |
| **Intention Lock** | A lock placed on an ancestor node (table/page) signaling that child records hold fine-grained locks. |
| **Wait-For Graph (WFG)**| Directed graph where nodes are transactions and edges represent active lock-wait dependencies. |
| **Victim Selection** | The algorithm choosing which transaction to abort when breaking a detected deadlock cycle. |
| **Thomas Write Rule** | Optimization allowing obsolete writes ($TS < W\text{-TS}$) to be ignored without aborting the transaction. |

---

## Example: SQL Explicit Row Locking

```sql
-- Transaction 1: Alice holds exclusive lock on Row ID=100
BEGIN;
SELECT * FROM products WHERE product_id = 100 FOR UPDATE;
-- Engine places IX lock on 'products' table, and X lock on row 100

-- Transaction 2: Bob attempts to read same row for update
BEGIN;
SELECT * FROM products WHERE product_id = 100 FOR UPDATE;
-- BLOCKED! Bob waits in lock queue until Transaction 1 issues COMMIT or ROLLBACK.

-- Transaction 3: Charlie performs non-locking read (MVCC)
SELECT * FROM products WHERE product_id = 100;
-- SUCCEEDS IMMEDIATELY! Plain SELECT uses snapshot read, ignoring locks.
```

---

## Real-World Example: MySQL InnoDB Deadlock Log Analysis
When an InnoDB transaction fails with `ERROR 1213 (40001): Deadlock found when trying to get lock; try restarting transaction`, developers run:
```sql
SHOW ENGINE INNODB STATUS;
```
The output reveals the exact lock graph:
```
*** (1) TRANSACTION:
TRANSACTION 2841, ACTIVE 2 sec starting index read
mysql tables in use 1, locked 1
LOCK WAIT 2 lock struct(s), heap size 1136, 1 row lock(s)
MySQL thread id 12, OS thread handle 12345, query id 456 UPDATE users SET balance=balance-10 WHERE id=1;

*** (2) TRANSACTION:
TRANSACTION 2842, ACTIVE 1 sec starting index read
mysql tables in use 1, locked 1
2 lock struct(s), heap size 1136, 2 row lock(s)
MySQL thread id 14, query id 457 UPDATE users SET balance=balance+10 WHERE id=2;
*** WE ROLL BACK TRANSACTION (2)
```
InnoDB picked Transaction 2 as the victim because it had fewer modified rows, rolling it back to release locks for Transaction 1.

---

## Important Properties

| Protocol | Guarantees Serializability? | Susceptible to Deadlocks? | Avoids Cascading Aborts? | Concurrency Throughput |
| :--- | :--- | :--- | :--- | :--- |
| **Basic 2PL** | Yes | Yes | **No** (Cascading aborts possible) | High |
| **Strict 2PL** | Yes | Yes | **Yes** (Standard RDBMS choice) | Moderate |
| **Rigorous 2PL** | Yes | Yes | **Yes** (Strict schedule) | Moderate-Low |
| **Conservative 2PL** | Yes | **No** (Pre-claims all locks) | Yes | Very Poor (Starvation risk) |
| **Timestamp Ordering** | Yes | **No** (Uses abort & restart) | Only with commit buffering | High read throughput |

---

### Common Misconceptions
1. *Misconception*: "2PL prevents deadlocks from occurring."
   - *Reality*: **False**. 2PL guarantees *serializability*, NOT deadlock freedom! Transactions obeying 2PL can easily reach deadlocks if they acquire locks in opposing orders. Only Conservative 2PL prevents deadlocks.
2. *Misconception*: "Intention locks (IS/IX) lock actual data rows."
   - *Reality*: **False**. Intention locks exist strictly at coarse-grained container levels (Tables, Partitions, Pages) as high-level flags to prevent full-table lock operations from having to inspect millions of row locks.

---

### What Interviewers Usually Test
- **2PL vs Strict 2PL vs Rigorous 2PL**: Exact difference in lock release timing (Basic releases in shrinking phase; Strict holds X until commit; Rigorous holds both S and X until commit).
- **Intention Lock Compatibility**: Why `IX` and `IX` are compatible (multiple transactions can write to different rows in the same table simultaneously).
- **Deadlock Resolution Strategies**: Wait-Die vs Wound-Wait behavior based on transaction age.

---

### Common Follow-up Questions

1. Why is Conservative 2PL rarely used in production databases?
2. What is the difference between an Intent Shared (`IS`) lock and a Shared (`S`) lock on a table?
3. How does `SELECT ... FOR UPDATE` differ from `SELECT ... LOCK IN SHARE MODE` (or `FOR SHARE`)?
4. In Wound-Wait, why does the older transaction "wound" the younger instead of dying?
5. Why does the Thomas Write Rule violate conflict serializability yet preserve view serializability?

---

### Follow-up Answers

1. **Conservative 2PL Drawbacks**: In practical web applications, a transaction cannot know in advance every single row it will ever touch (rows accessed depend on dynamic user input and branch conditions). Furthermore, locking all data items upfront causes severe lock contention and starvation.
2. **IS vs S Lock**: An `S` lock locks the *entire table* in read mode; no writer can write to *any* row in the table. An `IS` lock merely states: "I intend to read specific rows deep inside this table." Multiple transactions can hold `IS` and `IX` simultaneously as long as their individual target rows do not conflict.
3. **`FOR UPDATE` vs `FOR SHARE`**: `FOR UPDATE` requests an **Exclusive (X)** row lock, preventing others from reading with lock or modifying the row. `FOR SHARE` requests a **Shared (S)** row lock, allowing other transactions to read concurrently but preventing updates or deletes until the lock is released.
4. **Wound-Wait Philosophy**: Older transactions have already consumed significant CPU and I/O resources. Aborting an older transaction wastes more work than aborting a young transaction. In Wound-Wait, when an older transaction needs a lock, it preempts ("wounds") the younger transaction, minimizing total wasted system computation.
5. **Thomas Write Rule**: Thomas Write Rule ignores a write if an already younger write exists ($TS(T_i) < W\text{-TS}(Q)$). While this breaks conflict serializability (swapping conflicting writes would change state), the ignored write is obsolete and completely masked by the later write. The final view and output remain identical to a serial schedule, making it View Serializable.
