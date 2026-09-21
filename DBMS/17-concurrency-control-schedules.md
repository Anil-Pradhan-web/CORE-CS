# 17. Concurrency Control & Schedules

## What is it?
**Concurrency Control** is the database management subsystem that coordinates simultaneous execution of transactions on a shared database. A **Schedule** (or History) is the chronological order in which the interleaved operations (Read, Write, Commit, Abort) of multiple concurrent transactions are executed. Concurrency control ensures that interleaved schedules preserve consistency without introducing execution anomalies like dirty reads, lost updates, or cascading aborts.

---

## Simple Explanation
Suppose two people share a single Google Doc document:
- If User 1 edits line 5 while User 2 also edits line 5 at the exact same second without coordination, User 1's work will be overwritten and erased (**Lost Update**).
- If User 1 drafts a paragraph, User 2 reads it and tweets it, but User 1 decides to delete it 2 seconds later, User 2 has broadcasted private, non-existent draft content (**Dirty Read**).

Concurrency control acts as an automated traffic controller that interleaves operations for speed while guaranteeing the final outcome is identical to running User 1's entire transaction first, followed by User 2's entire transaction (**Serializability**).

---

## Hinglish Explanation
Jab multiple transactions parallel mein chalte hain CPU aur disk utilization badhane ke liye, unke operations ek doosre ke beech interleave hote hain:
- **Serial Schedule**: Transaction $T_1$ poora khatam hoga, tab $T_2$ start hoga. Yeh 100% safe hai, par performance bohot slow hoti hai (zero concurrency).
- **Non-Serial Concurrent Schedule**: $T_1$ aur $T_2$ ke reads aur writes aapas mein mix hote hain. Lekin agar mix galat ho gaya, toh anomalies create hoti hain (Lost Update, Dirty Read, Phantom Read).
- **Conflict Serializability**: Hum check karte hain ki kya concurrent schedule ko non-conflicting operations swap karke ek serial schedule mein badla ja sakta hai? Isko test karne ke liye **Precedence Graph (Serialization Graph)** banate hain. Agar graph mein cycle nahi hai (Directed Acyclic Graph - DAG), toh schedule **Conflict Serializable** hai aur 100% mathematically safe hai!

---

## Why Do We Need It?
1. **Preventing Catastrophic Anomalies**: Without concurrency control, e-commerce stores oversell inventory, banking ledgers drop customer credits, and duplicate flight seats are issued.
2. **Maximizing Resource Utilization & Throughput**: If the database forced all transactions to run serially one by one, high-end 64-core servers would sit idle while waiting for single-threaded disk I/O.
3. **Formal Mathematical Proof of Correctness**: Conflict and View serializability provide provable standards allowing database query optimizers to interleave disk I/O and CPU bursts safely.

---

## How It Works

### 1. The Four Major Concurrency Anomalies

```
1. DIRTY READ (Uncommitted Dependency)
   T1: Write(X) ───────────────► Abort (Rollback!)
            \
   T2:       Read(X) ──► Uses fake uncommitted data!

2. NON-REPEATABLE READ (Fuzzy Read)
   T1: Read(X=100) ──────────────────────────► Read(X=200) [Value changed midway!]
                          ▲
   T2:           Write(X=200) ──► Commit

3. PHANTOM READ
   T1: Read (COUNT rows WHERE age > 30) = 5 ─► Read again = 6 [New ghost row appeared!]
                                  ▲
   T2:                   INSERT INTO users (age=35) ──► Commit

4. LOST UPDATE (Blind Overwrite)
   T1: Read(X=100) ───────────────► Write(X = 100 + 50 = 150)
   T2:       Read(X=100) ──────────────────────────────────► Write(X = 100 + 20 = 120)
   Result: T1's addition of 50 is completely lost! Final X = 120 instead of 170.
```

---

### 2. Conflicting Operations
Two operations $I_i$ and $I_j$ belonging to different transactions $T_i$ and $T_j$ are in **Conflict** if and only if:
1. They access the **same data item** $X$.
2. At least **one of them is a Write** operation ($W(X)$).

| Operation 1 ($T_i$) | Operation 2 ($T_j$) | Conflict Status | Reason |
| :--- | :--- | :--- | :--- |
| $\text{Read}(X)$ | $\text{Read}(X)$ | **No Conflict** | Reading shared data concurrently causes no state change. |
| $\text{Read}(X)$ | $\text{Write}(X)$ | **CONFLICT** | Read-Write conflict (Unrepeatable / Inconsistent read). |
| $\text{Write}(X)$ | $\text{Read}(X)$ | **CONFLICT** | Write-Read conflict (Dirty read hazard). |
| $\text{Write}(X)$ | $\text{Write}(X)$ | **CONFLICT** | Write-Write conflict (Lost update hazard). |

---

### 3. Precedence Graph (Serialization Graph) Algorithm
To test if a concurrent schedule $S$ is **Conflict Serializable**:
1. Create a node for every active transaction $T_1, T_2, \dots, T_n$.
2. For any two conflicting operations accessing item $X$ where operation of $T_i$ executes *before* operation of $T_j$, draw a directed edge:
   $$T_i \longrightarrow T_j$$
3. **Theorem**: A schedule $S$ is conflict serializable if and only if its precedence graph contains **NO CYCLES** (is a DAG).
4. The equivalent serial schedule order is obtained by **Topological Sorting** of the DAG.

```
Example Schedule:
S = R1(A); W1(A); R2(A); W2(A); R1(B); W1(B); R2(B); W2(B);

Conflicts on A:
- W1(A) happens before R2(A)  ==> Edge: T1 -> T2
- W1(A) happens before W2(A)  ==> Edge: T1 -> T2

Conflicts on B:
- W1(B) happens before R2(B)  ==> Edge: T1 -> T2
- W1(B) happens before W2(B)  ==> Edge: T1 -> T2

Precedence Graph:
       ┌────┐         ┌────┐
       │ T1 │ ──────► │ T2 │
       └────┘         └────┘
Cycle Detected? NO!
Conclusion: Conflict Serializable! Equivalent Serial Schedule: T1 -> T2.
```

---

### 4. Schedule Taxonomy: Recoverability & Cascadelessness

```
                               All Schedules
                                     │
                        ┌────────────┴────────────┐
                        │                         │
                  Serializable               Non-Serializable
                        │
                  ┌─────┴────────────────┐
                  │                      │
             Recoverable           Non-Recoverable (Dangerous!)
                  │
             Cascadeless (Avoids Cascading Aborts - ACA)
                  │
                Strict Schedules
```

#### Recoverable Schedule
A schedule is **Recoverable** if: Whenever transaction $T_j$ reads data written by $T_i$ ($T_i \xrightarrow{W(X)} T_j \xrightarrow{R(X)}$), transaction $T_i$ **must commit before** $T_j$ commits.
*Why?* If $T_j$ commits first, and then $T_i$ aborts, $T_j$ cannot be rolled back (it already committed durable data to user), leaving the database permanently corrupted.

#### Cascadeless Schedule
A schedule avoids cascading aborts if transactions read **only committed data**:
$$T_j \text{ reads } X \text{ only after } T_i \text{ has already COMMITTED } X$$
*Benefit*: If $T_1$ aborts, no other active transaction has read its uncommitted dirty writes, preventing a domino-effect rollback of 100 dependent transactions.

---

## Important Terminology

| Term | Definition |
| :--- | :--- |
| **Serial Schedule** | Schedule where transactions run non-interleaved; one completes fully before the next starts. |
| **Serializable Schedule** | A concurrent interleaved schedule whose final database state is identical to some serial schedule. |
| **Conflict Equivalence** | Two schedules are conflict equivalent if one can be transformed into the other by swapping adjacent non-conflicting operations. |
| **Precedence Graph** | Directed graph testing conflict serializability; vertices are transactions, edges represent conflict precedence. |
| **Blind Write** | A transaction writing to item $X$ without having read $X$ first ($W(X)$ without prior $R(X)$). |
| **View Serializability** | A weaker form of serializability allowing blind writes where read/write views match a serial schedule. Testing is NP-Complete. |
| **Cascading Abort** | A failure cascade where aborting one transaction forces aborting multiple other transactions that read its dirty data. |

---

## Example: Cycle Detection in Precedence Graph

Consider Schedule $S_2$:
$$S_2 = R_1(X); \; W_2(X); \; W_1(X); \; C_1; \; C_2;$$

Let us analyze conflicts:
1. $R_1(X)$ precedes $W_2(X)$ $\implies$ Conflicting pair ($R-W$). Edge: **$T_1 \longrightarrow T_2$**.
2. $W_2(X)$ precedes $W_1(X)$ $\implies$ Conflicting pair ($W-W$). Edge: **$T_2 \longrightarrow T_1$**.

```
Precedence Graph:
         ┌──────┐
    ┌──► │  T1  │ ───┐
    │    └──────┘    │
    │                ▼
    │    ┌──────┐    │
    └─── │  T2  │ ◄──┘
         └──────┘
    CYCLE DETECTED: T1 -> T2 -> T1!
```
**Result**: Schedule $S_2$ is **NOT Conflict Serializable**. If executed, this schedule produces a Lost Update anomaly.

---

## Real-World Example: Airline Reservation Seat Sniping
Consider two concurrent customers, Alice ($T_1$) and Bob ($T_2$), booking Flight AI-202:
1. Alice queries seat 14B: `SELECT status FROM seats WHERE id = '14B'` (returns 'FREE').
2. Bob queries seat 14B: `SELECT status FROM seats WHERE id = '14B'` (returns 'FREE').
3. Alice executes: `UPDATE seats SET status = 'BOOKED', user = 'Alice' WHERE id = '14B'`.
4. Bob executes: `UPDATE seats SET status = 'BOOKED', user = 'Bob' WHERE id = '14B'`.
5. Both transactions commit.

Bob's update blindly overwrote Alice's reservation (**Lost Update**). Alice arrives at the airport with a confirmed boarding pass for a seat Bob now owns. Concurrency control engines stop this by forcing serialized row locks (`SELECT FOR UPDATE`) or MVCC conflict aborts.

---

## Important Properties

| Schedule Category | Guarantees Serial Execution Outcome? | Avoids Cascading Aborts? | Allows Non-Recoverable States? | Verification Complexity |
| :--- | :--- | :--- | :--- | :--- |
| **Conflict Serializable** | Yes | Not necessarily (depends on commit order) | Only if not recoverable | $O(V + E)$ cycle check |
| **View Serializable** | Yes (includes blind writes) | Not necessarily | Only if not recoverable | NP-Complete |
| **Cascadeless Schedule** | Only if serializable | **Guaranteed** | No (Strictly Recoverable) | Easy runtime rule enforcement |
| **Strict Schedule** | Only if serializable | **Guaranteed** (cannot read or overwrite uncommitted writes) | Impossible | Standard in production RDBMS |

---

### Common Misconceptions
1. *Misconception*: "Every View Serializable schedule is also Conflict Serializable."
   - *Reality*: **False**. Conflict Serializable is a strict *subset* of View Serializable. Every Conflict Serializable schedule is View Serializable, but some View Serializable schedules (involving blind writes) are NOT conflict serializable.
2. *Misconception*: "If a schedule is Conflict Serializable, it is guaranteed to avoid cascading aborts."
   - *Reality*: **False**. Serializability only guarantees *equivalence to a serial schedule*. A schedule can be perfectly conflict serializable but still suffer from cascading rollbacks if an uncommitted dirty read occurs before an abort.

---

### What Interviewers Usually Test
- **Drawing and Solving Precedence Graphs**: Given a sequence of $R/W$ operations across 2 or 3 transactions, identify conflicts, construct the graph, and determine topological order.
- **Differentiating Conflict vs View Serializability**: When do blind writes allow view serializability without conflict serializability.
- **Recoverability vs Cascadelessness Conditions**: Exact mathematical rule for commit ordering ($Commit_1 < Commit_2$) vs read ordering ($Commit_1 < Read_2$).

---

### Common Follow-up Questions

1. How do you construct an example of a schedule that is View Serializable but NOT Conflict Serializable?
2. What is the difference between a "Strict Schedule" and a "Cascadeless Schedule"?
3. Why do production databases enforce Conflict Serializability rather than View Serializability?
4. If a schedule has no cycles in its precedence graph, can it still deadlock?
5. How does a database enforce recoverability at runtime?

---

### Follow-up Answers

1. **View Serializable but Not Conflict Serializable**:
   $$S = R_1(X); \; W_2(X); \; W_1(X); \; W_3(X)$$
   - Precedence graph has $T_1 \to T_2$ (from $R_1(X), W_2(X)$) and $T_2 \to T_1$ (from $W_2(X), W_1(X)$) $\implies$ Cycle exists (Not Conflict Serializable).
   - However, $T_3$ performs a final blind write overwriting $X$. The final state and read view match serial schedule $T_1 \to T_2 \to T_3$. Hence it is View Serializable.
2. **Strict vs Cascadeless**:
   - **Cascadeless**: $T_j$ cannot *read* $X$ until $T_i$ commits ($W_1(X) \dots C_1 \dots R_2(X)$).
   - **Strict**: $T_j$ can neither *read* NOR *write* $X$ until $T_i$ commits ($W_1(X) \dots C_1 \dots W_2(X)$). Strict prevents dirty writes and simplifies recovery to copying pre-images.
3. **NP-Completeness of View Serializability**: Determining whether an arbitrary schedule is View Serializable is an NP-Complete problem ($O(2^N)$), whereas Conflict Serializability is verified in polynomial time $O(V + E)$ using topological sorting. Databases use protocols like 2PL and MVCC that automatically produce conflict serializable schedules.
4. **Graph Cycles vs Deadlocks**: A precedence graph tracks past execution history to check serializability. A **Wait-For Graph** tracks active transactions waiting on locks. A schedule can be conflict serializable without deadlocks; conversely, a deadlock halts execution before the schedule can even finish.
5. **Enforcing Recoverability**: The lock manager holds all exclusive write locks until the transaction issues `COMMIT` or `ABORT` (Strict 2PL). Because no concurrent transaction can read or commit on uncommitted writes, non-recoverable schedules are structurally impossible.
