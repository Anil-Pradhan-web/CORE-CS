# 16. Transactions & ACID Properties

## What is it?
A **Transaction** is a logical unit of database work containing one or more SQL operations (read, insert, update, delete) executed as an indivisible sequence. The **ACID properties** (**Atomicity**, **Consistency**, **Isolation**, **Durability**) represent the core guarantees that relational database management systems provide to preserve data correctness in the face of concurrent access, system failures, and power outages.

---

## Simple Explanation
Imagine transferring ₹1,000 from Alice's bank account to Bob's bank account:
1. Deduct ₹1,000 from Alice ($A = A - 1000$).
2. Add ₹1,000 to Bob ($B = B + 1000$).

If the server crashes right after Step 1, Alice loses ₹1,000 but Bob never receives it. The money vanishes into thin air!
A **Transaction** bundles both steps together.
- **Atomicity**: Either both steps finish, or neither does (the ₹1,000 deduction is rolled back if step 2 fails).
- **Consistency**: Total money ($A + B$) remains invariant before and after the transfer.
- **Isolation**: If Charlie queries Alice's balance in the middle of the transfer, he does not see partial, half-deducted money.
- **Durability**: Once the transaction says "Transfer Successful", a sudden power cut 1 millisecond later will not erase the transfer.

---

## Hinglish Explanation
Transaction ek aisi boundary hoti hai jiske andar saare database queries ek single logical unit of work bante hain:
- **Atomicity ("Sab kuch ya kuch bhi nahi")**: Agar ek transaction mein 10 updates hain aur 9th update par error aa jaye, toh pehle ke 8 updates bhi automatically **Rollback** (undo) ho jayenge. Database waisi state mein aa jayega jaise transaction kabhi shuru hi na hua ho.
- **Consistency ("Rules kabhi nahi tootenge")**: Database state hamesha valid honi chahiye. Balance kabhi negative nahi ho sakta (agar constraint hai), foreign key invalid nahi ho sakti. Agar transaction koi constraint violate karega, toh rollback ho jayega.
- **Isolation ("Main akele chal raha hoon")**: Jab hazaron users ek saath ek hi table pe transactions run karte hain, tab bhi har transaction ko lagna chahiye ki poore database pe sirf wahi akele execute ho raha hai. Intermediate uncommitted changes doosre transactions ko nahi dikhte.
- **Durability ("Commit ho gaya toh pathar ki lakeer")**: Ek baar user ko `COMMIT OK` response mil gaya, uske baad chahe server ka power plug nikal jaye, hard drive crash ho jaye (with backup), server restart hone par transaction ka data safely disk pe committed milega via WAL (Write-Ahead Log).

---

## Why Do We Need It?
1. **Preventing Financial Inconsistencies**: Without atomicity, partial execution destroys ledger integrity, causing unrecoverable revenue loss or duplicate disbursements.
2. **Abstracting Failure Handling from Application Code**: Without ACID, application developers would have to write manual, bug-prone compensatory rollback logic for every possible network timeout, disk full error, or process crash.
3. **Safe High-Concurrency Processing**: ACID allows thousands of checkout requests, flight bookings, and stock market orders to hit the same tables simultaneously without corrupting underlying shared memory.

---

## How It Works

### 1. Transaction State Diagram

```
                       ┌─────────────────────────┐
                       │         ACTIVE          │
                       │ (Executing Operations)  │
                       └────────────┬────────────┘
                                    │
               Normal Execution     │     Error / Crash / Abort
               Finishes Successfully│     Detected
                                    ▼
       ┌────────────────────────┐        ┌────────────────────────┐
       │   PARTIALLY COMMITTED  │        │         FAILED         │
       │ (Final Operation Done, │        │ (Error encountered,    │
       │  Flushing WAL to disk) │        │  Cannot proceed)       │
       └───────────┬────────────┘        └───────────┬────────────┘
                   │                                 │
         WAL fsync │           WAL Flush Failure     │
        Successful │          ┌──────────────────────┘
                   │          │
                   ▼          ▼
            ┌──────────────┐  ┌──────────────────────────────────┐
            │  COMMITTED   │  │             ABORTED              │
            │ (Successful, │  │ (Undo Log applied, Data restored │
            │  Permanent)  │  │  to original pre-tx state)       │
            └──────────────┘  └────────────────┬─────────────────┘
                                               │
                                               ▼
                                      ┌───────────────────┐
                                      │    TERMINATED     │
                                      │  (Connection/Tx   │
                                      │   Cleanup Done)   │
                                      └───────────────────┘
```

---

### 2. How the DBMS Enforces Each ACID Property

```
┌──────────────┬─────────────────────────────────────────────────────────────┐
│ ACID Pillar  │ Primary Under-the-Hood Engine Mechanism                     │
├──────────────┼─────────────────────────────────────────────────────────────┤
│ Atomicity    │ Undo Log / Rollback Segment in WAL (reverses uncommitted ops)│
├──────────────┼─────────────────────────────────────────────────────────────┤
│ Consistency  │ Schema Constraints (PRIMARY KEY, CHECK, FK) + App Invariants│
├──────────────┼─────────────────────────────────────────────────────────────┤
│ Isolation    │ Concurrency Control Protocols (Locks, 2PL, MVCC Read Views) │
├──────────────┼─────────────────────────────────────────────────────────────┤
│ Durability   │ Write-Ahead Logging (Redo Log) + OS `fsync()` system calls  │
└──────────────┴─────────────────────────────────────────────────────────────┘
```

#### Atomicity & The Undo Log
When a transaction runs:
```sql
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
```
Before the in-memory page is modified, the database appends an **Undo Record** containing the *old value* (e.g., `id=1, old_balance=500`) to the undo log buffer. If the transaction aborts or fails midway, the recovery manager scans the undo records backward and restores every modified row to its pre-transaction state.

#### Durability & The Write-Ahead Log (WAL)
Writing entire 16KB data pages to random disk sectors during every commit is too slow. Instead:
1. Modifications are recorded as sequential, append-only **Redo Log entries** in memory.
2. At `COMMIT`, the DBMS forces an `fsync()` of only the Redo Log buffer to disk.
3. Once the log bytes are safely on disk, `COMMIT` returns success to the client. If power fails immediately after, the dirty pages can be reconstructed from the redo log during restart.

---

## Important Terminology

| Term | Definition |
| :--- | :--- |
| **Transaction ID (TID / XID)** | Unique monotonically increasing integer assigned to each transaction. |
| **Commit** | Finalizing transaction operations, persisting redo logs, and making changes visible to other transactions. |
| **Rollback / Abort** | Canceling transaction operations and applying undo logs to revert all modified records. |
| **Savepoint** | A marked point within a transaction allowing partial rollback without terminating the outer transaction. |
| **Undo Log** | Log records capturing previous values used to undo uncommitted changes during rollback or crash recovery. |
| **Redo Log** | Log records capturing new values used to replay committed changes during crash recovery. |
| **In-Flight Transaction** | An active transaction that has neither committed nor aborted. |

---

## Example: Production Transaction with Savepoints

### SQL Implementation
```sql
BEGIN TRANSACTION;

-- Step 1: Deduct from customer wallet
UPDATE wallets 
SET balance = balance - 500 
WHERE user_id = 42 AND balance >= 500;

-- Check if wallet had sufficient balance
-- (In application code: if rows_affected == 0, ROLLBACK)

-- Step 2: Establish savepoint before third-party reward coupon
SAVEPOINT before_reward;

-- Step 3: Attempt optional promotional coupon
UPDATE coupons 
SET used = TRUE 
WHERE coupon_code = 'FESTIVE50' AND used = FALSE;

-- If coupon expired or invalid, rollback to savepoint only!
ROLLBACK TO SAVEPOINT before_reward;

-- Step 4: Finalize core purchase
INSERT INTO orders (user_id, amount, status) 
VALUES (42, 500, 'PAID');

COMMIT;
```

### Python (Psycopg2 / SQLAlchemy Pattern)
```python
import psycopg2

def execute_money_transfer(conn, sender_id: int, receiver_id: int, amount: float):
    try:
        with conn.cursor() as cur:
            # Atomic transaction begins automatically with context manager
            cur.execute(
                "UPDATE accounts SET balance = balance - %s WHERE id = %s AND balance >= %s",
                (amount, sender_id, amount)
            )
            if cur.rowcount == 0:
                raise ValueError("Insufficient balance or sender not found")

            cur.execute(
                "UPDATE accounts SET balance = balance + %s WHERE id = %s",
                (amount, receiver_id)
            )
            if cur.rowcount == 0:
                raise ValueError("Receiver account not found")

            # Persist transfer audit log
            cur.execute(
                "INSERT INTO transfer_logs (sender, receiver, amount) VALUES (%s, %s, %s)",
                (sender_id, receiver_id, amount)
            )

        # Commit on successful block exit
        conn.commit()
        print("Transfer completed and committed successfully.")
    except Exception as e:
        # Atomic rollback on any runtime or database exception
        conn.rollback()
        print(f"Transaction failed, rolled back completely: {e}")
```

---

## Real-World Example: UPI / Payment Gateway Ledger System
When an Indian UPI transaction occurs:
1. Payer PSP initiates deduction from Bank A.
2. NPCI acts as central switch.
3. Payee Bank B credits beneficiary account.

At the database level within Bank A:
- If debit succeeds, but NPCI network times out, Bank A creates a **Reverse Transaction** or applies a **Reconciliation Job** based on idempotency keys.
- Internally, Bank A's core banking database uses **Strict Two-Phase Locking (SS2PL) + WAL** to guarantee that debit operations and audit ledger entries are committed atomically. At no point can a user's balance drop without a corresponding immutable ledger entry.

---

## Important Properties

| ACID Property | Target Failure Addressed | Key Enforcing Mechanism | Performance Trade-off |
| :--- | :--- | :--- | :--- |
| **Atomicity** | Mid-transaction crashes, exceptions, disk-full errors | Undo Log / Rollback segments | Undo storage overhead and rollback CPU cycles |
| **Consistency** | Corrupted data models, negative balances, orphaned keys | Constraints, Foreign Keys, Application assertions | CPU validation checks on every INSERT/UPDATE |
| **Isolation** | Race conditions, dirty reads, concurrent overwrites | Locks (2PL), MVCC snapshots, latching | Concurrency limits, lock contention, lock wait timeouts |
| **Durability** | Power outages, OS panics, disk drive resets | Redo Log, Sequential WAL flush via `fsync` | Disk I/O bottleneck during high commit rates |

---

### Common Misconceptions
1. *Misconception*: "Consistency in ACID means the same thing as Consistency in the CAP Theorem."
   - *Reality*: **NO!** ACID Consistency ($C$) means *application-defined database invariants and integrity constraints* (e.g., balance $\ge 0$). CAP Consistency ($C$) means *Linearizability/Single-copy serializability across distributed network nodes* (every read receives the most recent write).
2. *Misconception*: "Calling `COMMIT` immediately writes all modified table pages to the hard drive."
   - *Reality*: Table data pages remain dirty in RAM. `COMMIT` only guarantees that the sequential Redo Log entries have been written and flushed (`fsync`) to disk. Data pages are lazily flushed later.

---

### What Interviewers Usually Test
- **Transaction Lifecycle States**: Difference between `PARTIALLY COMMITTED` and `COMMITTED` (partially committed means in-memory execution finished; committed means WAL has physically synced to disk).
- **ACID Component Roles**: Which specific mechanism handles which property (WAL Undo for A, Constraints for C, Locks/MVCC for I, WAL Redo + fsync for D).
- **Nested Transactions vs Savepoints**: Why most RDBMSs implement savepoints rather than true independent nested transactions.

---

### Common Follow-up Questions

1. Why does the DBMS flush the WAL before writing dirty data pages to disk (The WAL Protocol)?
2. What is the difference between a `ROLLBACK` and a `COMPENSATING TRANSACTION`?
3. How does `innodb_flush_log_at_trx_commit = 1` differ from `= 0` or `= 2` in MySQL?
4. Can an application achieve Durability without calling `fsync()`?
5. What happens to open transactions when a client connection drops unexpectedly?

---

### Follow-up Answers

1. **Write-Ahead Logging Rule**: If a dirty data page were written to disk before its corresponding WAL log record, and the system crashed during the write, disk data would be updated with no log record to undo it. The WAL rule states: *An undo log record must hit disk before the data page it describes hits disk.*
2. **Rollback vs Compensating Transaction**: A `ROLLBACK` is an internal engine operation reverting uncommitted changes of an active transaction. A **Compensating Transaction** is a brand new, forward-committing transaction designed to undo the business effect of a *previously committed* transaction in distributed architectures (e.g., Sagas).
3. **InnoDB Flush Modes**:
   - `1` (Full ACID): Flushes and `fsync`s the log buffer to disk on every single commit (100% durable, highest I/O).
   - `0`: Log buffer is written and synced to disk only once per second (can lose up to 1 second of transactions on MySQL crash or OS crash).
   - `2`: Log buffer is written to OS cache on every commit, but `fsync`ed only once per second (survives MySQL process crash, but loses data on OS/power failure).
4. **Durability Requires OS Sync**: No. Writing to a file descriptor simply copies bytes into the OS page cache. If the machine loses power, volatile RAM cache is lost. Durability strictly requires forcing hardware disk controllers via `fsync()`, `fdatasync()`, or `O_SYNC`.
5. **Connection Drop Cleanup**: The database server detects a broken TCP socket (keep-alive probe or EOF). It immediately signals the transaction context, sets state to `FAILED`, rolls back all uncommitted operations using the undo log, releases all held table/row locks, and terminates the session.
