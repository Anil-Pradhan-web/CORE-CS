# 20. Crash Recovery, WAL & ARIES

## What is it?
**Crash Recovery** is the database subsystem that restores a database to a consistent, durable state following a hardware crash, power outage, or operating system failure. The industry-standard recovery architecture is built upon the **Write-Ahead Logging (WAL)** protocol and the **ARIES (Algorithms for Recovery and Isolation Exploiting Semantics)** algorithm. ARIES guarantees that all changes from committed transactions are preserved (Redo) and all changes from uncommitted in-flight transactions are completely undone (Undo).

---

## Simple Explanation
Imagine an accountant keeping a physical ledger:
- If the accountant writes numbers directly into the master ledger with permanent ink and the building loses power midway, the master ledger has half-baked, untraceable errors.
- Instead, the accountant writes down every intention into a chronological spiral notebook first: `"Entry #402: Subtracted ₹500 from Account 10, added ₹500 to Account 20"` and snaps a photo of it (**Write-Ahead Log**).
- After power returns:
  1. The accountant checks the notebook from the last marked bookmark (**Analysis Phase**).
  2. Re-applies every single completed transaction to the master ledger (**Redo Phase - Repeating History**).
  3. Cancels and crosses out any entries that were left halfway done when the lights went out (**Undo Phase**).

---

## Hinglish Explanation
Database RAM mein bohot fast kaam karta hai, lekin RAM volatile hoti hai (power cut hote hi memory saaf).
Agar server crash ho jaye, do badi problems hoti hain:
1. Kuch transactions `COMMIT` ho chuki thi, par unke dirty pages abhi RAM ke buffer pool mein the aur disk par flush nahi huye the (**Durability violated**).
2. Kuch transactions crash ke waqt *in-progress* (uncommitted) thi, par buffer pool ne unke dirty pages pehle hi disk par write kar diye the (**Atomicity violated**).

Database isko kaise solve karta hai?
- **WAL Protocol**: Disk pe koi bhi actual data page write karne se pehle, uska log record disk ke append-only log file mein flush hona hi chahiye.
- **ARIES Algorithm**: Crash ke baad restart hone par 3 phases chalte hain:
  1. **Analysis**: Checkpoint se shuru karke pata lagata hai ki crash ke time kaunsi transactions active thi ("Losers") aur kaunse pages dirty the.
  2. **Redo**: Sab kuch waisa replay karta hai jaisa crash se pehle hua tha ("Repeating History"), chahe commit hua ho ya abort.
  3. **Undo**: Jo transactions crash ke waqt adhoori thi ("Losers"), unke changes ko reverse (undo) karta hai aur **CLR (Compensation Log Records)** likhta hai taaki agar recovery ke dauran dobara crash ho jaye, toh engine loop mein na phase!

---

## Why Do We Need It?
1. **Zero Data Loss for Committed Transactions**: Guarantees Durability without forcing slow, synchronous random disk writes on every single SQL commit.
2. **Crash-Safe Self-Healing**: Enables production databases to restart automatically after power failures in seconds without human manual ledger reconstruction.
3. **Idempotent Recovery Execution**: If a server crashes *while recovering from a previous crash*, ARIES safely resumes without corrupting data or duplicating log steps.

---

## How It Works

### 1. Buffer Pool Policies: Steal vs Force Matrix

Database storage engines face two architectural design decisions regarding RAM buffer frames and disk:

```
                  ┌──────────────────────┬──────────────────────┐
                  │       NO-FORCE       │        FORCE         │
┌─────────────────┼──────────────────────┼──────────────────────┤
│      STEAL      │ Modern RDBMS Default │ Rare / Impractical   │
│                 │ (MySQL, Postgres)    │                      │
│                 │ Fast Commit, Fast I/O│ High I/O overhead    │
│                 │ Needs: REDO + UNDO   │ Needs: UNDO only     │
├─────────────────┼──────────────────────┼──────────────────────┤
│    NO-STEAL     │ High RAM requirement │ Toy / Primitive DBs  │
│                 │ Needs: REDO only     │ No REDO, No UNDO     │
│                 │                      │ Horrible throughput  │
└─────────────────┴──────────────────────┴──────────────────────┘
```

- **STEAL**: Buffer pool manager is permitted to write a dirty page belonging to an *uncommitted* transaction to disk if RAM is tight. (Requires **UNDO logging** to revert disk pages if the transaction later aborts).
- **NO-FORCE**: Buffer pool manager does *not* need to flush all modified data pages to disk at the time of commit. (Requires **REDO logging** to reapply committed modifications from log after a crash).
- Modern engines choose **STEAL + NO-FORCE** because it delivers the highest transaction throughput and lowest commit latency.

---

### 2. WAL Log Sequence Numbers (LSN) Mechanics

Every log record is assigned a strictly increasing integer: **LSN (Log Sequence Number)**.

```
Log Record on Disk/RAM:
┌──────┬─────────┬────────┬───────┬────────┬───────────┬──────────┐
│ LSN  │ PrevLSN │ Trx_ID │ Type  │ PageID │ Undo Data │ Redo Data│
└──────┴─────────┴────────┴───────┴────────┴───────────┴──────────┘
```

- **PageLSN**: Every database page header contains the LSN of the most recent log record that updated it.
- **FlushedLSN**: The largest LSN safely flushed to persistent storage.
- **The WAL Rule**:
  $$\text{PageLSN} \le \text{FlushedLSN}$$
  *A dirty page cannot be written to disk until all log records up to that page's PageLSN have been flushed to disk!*

---

### 3. Checkpointing & Fuzzy Checkpoints
A simple checkpoint stops all transactions, flushes all dirty pages, and marks a clean point. This causes massive latency spikes (system freeze).
Modern databases use **Non-Quiescent (Fuzzy) Checkpoints**:
1. Take a snapshot of in-memory state without stopping user transactions.
2. Write a `CHECKPOINT_BEGIN` log record.
3. Write two internal tables into a `CHECKPOINT_END` record:
   - **Transaction Table (TT)**: List of all active transactions and their `lastLSN`.
   - **Dirty Page Table (DPT)**: List of all dirty pages in memory and their `recLSN` (the earliest LSN that dirtied the page).

---

### 4. The Three Phases of ARIES Recovery

```
Log File on Disk:
──────┬──────────────────────┬─────────────────────────┬──────────────► [CRASH]
      │                      │                         │
Checkpoint Begin/End    Oldest recLSN              Crash Point
                             │
                             ├───────────────► 1. ANALYSIS PHASE
                             │                 (Rebuilds TT and DPT)
                             │
                             ├───────────────────────────────────────► 2. REDO PHASE
                             │                                         (Repeats History from oldest recLSN)
                             │
                             ◄──────────────────────────────────────── 3. UNDO PHASE
                                                                       (Rolls back Loser Transactions)
```

#### Phase 1: Analysis Phase
- Scans forward through the log starting from the last valid checkpoint.
- Reconstructs the **Transaction Table (TT)** and **Dirty Page Table (DPT)**.
- Identifies **"Loser Transactions"** (transactions active at the moment of crash with no `COMMIT` or `ABORT` record).

#### Phase 2: Redo Phase ("Repeating History")
- Scans forward starting from the minimum `recLSN` found in the DPT up to the very end of the log.
- Re-executes all logged operations (both committed and uncommitted transactions!).
- **Optimization**: Skips disk page write if $\text{PageLSN} \ge \text{Log LSN}$ (meaning the change was already on disk before the crash).

#### Phase 3: Undo Phase (Rolling Back Losers)
- Scans backward from the end of the log, undoing the operations of all Loser transactions.
- For every reversed operation, writes a **Compensation Log Record (CLR)**.
- Each CLR contains an `UndoNextLSN` pointer pointing to the next operation that must be undone for that transaction.
- **CLR Safety**: CLRs are *never undone*. If the database crashes midway through the Undo phase, it restarts and avoids repeating already completed undo operations!

---

## Important Terminology

| Term | Definition |
| :--- | :--- |
| **LSN (Log Sequence Number)** | Monotonically increasing 64-bit integer uniquely identifying every log record byte offset. |
| **PageLSN** | Metadata field in a physical disk page header recording the latest LSN applied to that page. |
| **`recLSN` (Recovery LSN)** | Earliest LSN whose modifications have not yet been flushed to the physical disk page. |
| **Dirty Page Table (DPT)** | In-memory table tracking all modified pages in the buffer pool and their `recLSN`. |
| **Transaction Table (TT)** | In-memory table tracking all active transactions, status, and their `lastLSN`. |
| **CLR (Compensation Log Record)**| Log record written during transaction rollback recording an undo action; prevents undo loops on crash. |
| **Loser Transaction** | A transaction active at the time of crash that had neither committed nor completed rollback. |

---

## Example: Step-by-Step ARIES Trace

Suppose the log contains the following sequence:
```
LSN 10: T1 modifies Page P1 (recLSN for P1 = 10)
LSN 20: T2 modifies Page P2 (recLSN for P2 = 20)
LSN 30: CHECKPOINT (TT: [T1, T2], DPT: [P1: 10, P2: 20])
LSN 40: T1 modifies Page P3 (recLSN for P3 = 40)
LSN 50: T2 COMMITS (Flushed to disk!)
LSN 60: T3 modifies Page P2
-- [SYSTEM CRASHES HERE!] --
```

### Recovery Steps:
1. **Analysis Phase**:
   - Reads Checkpoint at LSN 30.
   - Reaches LSN 50: Removes T2 from TT (T2 committed).
   - Reaches LSN 60: Adds T3 to TT.
   - At crash end: **Losers = {T1, T3}**. **DPT = {P1: 10, P2: 20, P3: 40}**.
2. **Redo Phase**:
   - Smallest `recLSN` is 10 (Page P1).
   - Scans forward from LSN 10 to 60. Replays changes for LSN 10, 20, 40, 50, 60. (Full history restored).
3. **Undo Phase**:
   - Rollback Losers {T1, T3}.
   - Undoes LSN 60 (T3), writes CLR with `UndoNextLSN = 0`.
   - Undoes LSN 40 (T1), writes CLR with `UndoNextLSN = 10`.
   - Undoes LSN 10 (T1), writes CLR with `UndoNextLSN = 0`.
   - Writes `ABORT / END` records for T1 and T3. Database is now 100% clean and consistent!

---

## Real-World Example: PostgreSQL WAL & Point-in-Time Recovery (PITR)
PostgreSQL's crash recovery engine powers both crash survival and enterprise backup:
1. **WAL Files (16MB segments)**: Stored under `pg_wal/`.
2. **Archiving**: In production, `archive_command` continuously uploads completed 16MB WAL segments to cloud object storage (Amazon S3 / Google Cloud Storage).
3. **PITR Restore**:
   - If an engineer accidentally runs `DROP TABLE customers;` at 02:14:32 PM, the company restores the base backup from midnight.
   - PostgreSQL replays WAL segments forward and stops recovery at:
     ```
     recovery_target_time = '2026-09-22 14:14:30'
     ```
   - All transactions up to 2 seconds before the disastrous drop are restored without loss!

---

## Important Properties

| Feature | Analysis Phase | Redo Phase | Undo Phase |
| :--- | :--- | :--- | :--- |
| **Scan Direction** | Forward | Forward | **Backward** |
| **Starting Point** | Last Checkpoint record | Oldest `recLSN` in DPT | Highest LSN among loser transactions |
| **Actions Taken** | Rebuilds TT and DPT | Replays all operations ("repeats history") | Reverses changes of loser transactions |
| **Log Records Written** | None | None | **Compensation Log Records (CLRs)** |
| **Idempotent?** | Yes | Yes (checking PageLSN avoids duplicates) | Yes (guided by CLR `UndoNextLSN` pointers) |

---

### Common Misconceptions
1. *Misconception*: "During the Redo phase, the database only re-executes changes of committed transactions."
   - *Reality*: **False**. ARIES adheres to the philosophy of **Repeating History**. It redoes the actions of *ALL* transactions—both committed AND uncommitted losers—up to the crash point. The uncommitted loser actions are cleanly rolled back later in the Undo phase.
2. *Misconception*: "Compensation Log Records (CLRs) can be undone if the system crashes again."
   - *Reality*: **False**. CLRs are *never* undone. They record an undo action that already succeeded. Undoing a CLR would re-apply the original faulty operation, causing an infinite recovery loop.

---

### What Interviewers Usually Test
- **Steal / No-Force Policy Rationale**: Why modern systems choose Steal/No-Force despite needing both Redo and Undo logging.
- **The WAL Invariant**: $\text{PageLSN} \le \text{FlushedLSN}$.
- **Purpose of Compensation Log Records (CLRs)**: How the `UndoNextLSN` field guarantees bounded recovery time even under cascading re-crashes.

---

### Common Follow-up Questions

1. Why does ARIES repeat history for uncommitted transactions during the Redo phase instead of skipping directly to committed ones?
2. What would happen if a dirty page was flushed to disk before its corresponding WAL record hit disk?
3. How does the database know if a page on disk already contains a logged modification during Redo?
4. What is the difference between physical logging, logical logging, and physiological logging?
5. Why does a Fuzzy Checkpoint record both the Transaction Table and the Dirty Page Table?

---

### Follow-up Answers

1. **Why Repeat History for Losers**: In modern engines with dynamic locking, uncommitted transactions hold page latches and produce physical page structure changes (e.g., B+ Tree node splits). By repeating history, the exact physical storage layout is recreated before logical undo operations execute. This cleanly separates concurrency control from recovery logic.
2. **Page Flushed Before WAL (WAL Violation)**: If the system crashes after the dirty page is written but before the log is flushed, the disk contains uncommitted changes with *no record in the log* describing what was changed or how to undo it. Atomicity is permanently violated and the database is corrupted.
3. **Checking `PageLSN`**: During Redo, the recovery engine inspects the `PageLSN` stamped inside the target disk page's header. If $\text{PageLSN} \ge \text{LogRecord.LSN}$, the change is already reflected on disk, and the engine skips the redo operation.
4. **Physiological Logging**: Pure physical logging records before/after byte images (huge log volume). Pure logical logging records SQL statements like `UPDATE emp SET sal = sal * 1.1` (difficult to guarantee idempotency during partial crashes). **Physiological logging** (used by ARIES) logs physical page addresses with logical operation descriptions inside that page, combining compact size with exact recovery determinism.
5. **Checkpoint Tables**: The **DPT** tells the recovery manager the earliest point in the log where Redo must begin ($\min(\text{recLSN})$), avoiding having to replay the log from the beginning of time. The **TT** provides the starting set of active transactions so Analysis does not need to scan backwards.
