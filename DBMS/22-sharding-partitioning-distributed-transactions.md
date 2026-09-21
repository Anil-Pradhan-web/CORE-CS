# 22. Sharding, Partitioning & Distributed Transactions

## What is it?
**Database Partitioning and Sharding** is the architectural technique of splitting large database tables into smaller, distinct subsets called **shards** and distributing them across independent physical machines. While partitioning traditionally refers to splitting data within a single database instance, **Sharding** distributes data across multiple independent database nodes (**Shared-Nothing Architecture**).
When transactions must read or write data spanning multiple independent shards, the database must execute **Distributed Transactions** using atomic consensus protocols such as **Two-Phase Commit (2PC)** or application-level **Sagas**.

---

## Simple Explanation
Imagine a dictionary with 10 million words:
- Instead of printing one 100-kilogram mega-book that no person can lift, you print 26 smaller volumes: Volume A, Volume B, ..., Volume Z (**Range-Based Sharding**).
- If someone wants to look up "Apple", they go directly to Book A without opening any other volume (**Targeted Shard Query**).
- But if you want to find "Every English word borrowed from French", you must open all 26 volumes simultaneously, search each, and merge the results (**Scatter-Gather Cross-Shard Query**).
- If an operation requires modifying words in both Book A and Book Z simultaneously, you need a two-step voting agreement (**Two-Phase Commit**) so that both books update or neither does.

---

## Hinglish Explanation
Jab ek database table itna bada ho jata hai (e.g., 50 TB, millions of writes per second) ki duniya ka sabse powerful machine bhi usko sambhal nahi sakta, hum **Horizontal Sharding** karte hain:
- **Shard Key**: Table ki ek specific column (jaise `user_id`) chunte hain jiske basis par decide hota hai ki kaunsa row kis physical server pe jayega.
- **Hash-Based Sharding & Consistent Hashing**: `hash(user_id) % num_shards` karke data evenly distribute karte hain. Consistent hashing with virtual nodes se naye shard servers add karne par data rebalancing bohot minimal hoti hai.
- **Distributed Transactions (2PC)**: Agar ek transaction Shard A aur Shard B dono pe update karta hai, standard commit kaam nahi karega. Ek **Coordinator** hota hai jo:
  1. **Phase 1 (Prepare)**: Dono shards se puchta hai "Kya tum commit karne ke liye ready ho?". Dono resources lock karke "YES" bolte hain.
  2. **Phase 2 (Commit)**: Coordinator dono ko bolta hai "GLOBAL COMMIT".
- **2PC Blocking Problem**: Agar Phase 1 ke baad Coordinator crash ho jaye, toh saare participant shards locks pakad ke permanently freeze ho jate hain! Isliye modern microservices 2PC ki jagah **Saga Pattern (Compensating Transactions)** use karte hain.

---

## Why Do We Need It?
1. **Breaking Hardware Limits (Horizontal Scale-Out)**: A single PostgreSQL instance hits practical ceiling limits at 5-10 TB and ~50k writes/sec. Sharding across 100 nodes scales storage to petabytes and throughput to millions of operations/sec.
2. **Eliminating Single Points of Contention**: Sharding separates hot tables across independent CPU cores, RAM buses, and disk controllers.
3. **Cross-Partition Consistency Guarantees**: When business workflows span independent accounts or microservices, distributed transaction algorithms guarantee atomic execution without data corruption.

---

## How It Works

### 1. Sharding Strategies Comparison

```
1. RANGE-BASED SHARDING
   [A - D] ──► Shard 1
   [E - M] ──► Shard 2
   [N - Z] ──► Shard 3
   Risk: Hotspotting! If most users have names starting with 'S', Shard 3 melts.

2. HASH-BASED SHARDING
   hash(user_id) % 3 == 0 ──► Shard 1
   hash(user_id) % 3 == 1 ──► Shard 2
   hash(user_id) % 3 == 2 ──► Shard 3
   Risk: Changing shard count from 3 to 4 forces re-hashing and moving 75% of all data!

3. CONSISTENT HASHING RING (With Virtual Nodes)
               ┌───────── Shard Node 0 ────────┐
          Node 2_v1                           Node 1_v1
         /                                             \
     Node 1_v2                                       Node 0_v2
        │                 [ 2^32 - 1 Hash Ring ]        │
     Node 0_v1                                       Node 2_v2
         \                                             /
          Node 2_v3                           Node 1_v3
               └───────── Shard Node 2 ────────┘
   Benefit: Adding a new shard node moves only K/N keys. Virtual nodes ensure uniform load distribution.
```

---

### 2. Choosing a Good Shard Key

Selecting a poor shard key is the #1 reason distributed databases fail in production:

| Anti-Pattern Key | Problem Caused | Production Failure Scenario |
| :--- | :--- | :--- |
| **Monotonically Increasing (`auto_increment_id`, `created_at`)** | **Write Hotspotting** | All new writes at 02:00 PM hit only the single latest shard; other 99 shards sit idle. |
| **Low Cardinality (`country_code`, `status`)** | **Lumpy Shards** | Shard for 'US' or 'IN' holds 80% of all data; shards for smaller countries hold 0.01%. Cannot be split. |
| **Random UUID without grouping** | **Scatter-Gather Floods** | Every query searching for a user's recent orders must broadcast to all 100 shards and join in memory. |
| **Ideal Shard Key** | High Cardinality + Uniform Distribution + Natural Query Affinity (e.g., `company_id` for multi-tenant B2B, or `user_id` for consumer apps). |

---

### 3. Two-Phase Commit (2PC) Protocol

```
    COORDINATOR                               PARTICIPANTS (Shard A, Shard B)
         │                                                   │
         │─────── PHASE 1: PREPARE (Can you commit?) ───────►│
         │                                                   │ (Acquire Locks,
         │                                                   │  Write Undo/Redo Log,
         │                                                   │  Flush to Disk)
         │◄────── VOTE COMMIT ("YES") / VOTE ABORT ("NO") ───│
         │                                                   │
   (All voted YES?)                                          │
         │                                                   │
         │─────── PHASE 2: GLOBAL COMMIT / ABORT ───────────►│
         │                                                   │ (Release Locks,
         │                                                   │  Finalize Changes)
         │◄────── ACKNOWLEDGMENT ────────────────────────────│
         │                                                   │
   Transaction Complete!
```

#### The Fatal Flaw of 2PC: The Blocking Coordinator Problem
If the Coordinator sends `PREPARE`, receives `YES` from all participants, and **crashes before sending `GLOBAL COMMIT`**:
- Participants are in an **Uncertain (In-Doubt) State**.
- They hold exclusive row locks on their local data.
- They *cannot abort* unilaterally (because the coordinator might have already told other participants to commit).
- They *cannot commit* unilaterally (because someone else might have voted NO).
- **Result**: Data items remain locked indefinitely until the coordinator recovers or an operator intervenes manually.

---

### 4. Sagas vs Two-Phase Commit (2PC)

To avoid blocking locks across microservices, modern architectures use **Sagas** (Eventual Consistency):

```
                       SAGA EXECUTION FLOW
Step 1: Debit Wallet (Local Tx 1) ────────► Success
                                               │
Step 2: Reserve Flight (Local Tx 2) ───────► Success
                                               │
Step 3: Book Hotel (Local Tx 3) ───────────► FAILS! (Out of rooms)
                                               │
                          ◄────────────────────┘
               COMPENSATING TRANSACTIONS
Compensate 2: Cancel Flight Reservation ◄── (Undo business effect)
Compensate 1: Refund Money to Wallet    ◄── (Undo business effect)
```

| Dimension | Two-Phase Commit (2PC) | Sagas Pattern |
| :--- | :--- | :--- |
| **Consistency Model** | ACID (Strict Immediate Consistency) | BASE (Eventual Consistency) |
| **Isolation** | High (Holds distributed locks) | **No Isolation** (Intermediate states visible to world) |
| **Failure Recovery** | Automatic rollback via undo log | Explicit **Compensating Transactions** |
| **Scalability** | Poor (Latency of slowest node + lock contention) | High (Asynchronous message-driven execution) |

---

## Important Terminology

| Term | Definition |
| :--- | :--- |
| **Horizontal Sharding** | Partitioning table rows across separate physical database instances. |
| **Vertical Partitioning**| Splitting table columns into separate tables (e.g., separating wide text blobs from hot integer metadata). |
| **Shard Key** | The column value determining which shard owns a given row. |
| **Consistent Hashing** | Circular hash ring mapping both nodes and keys to an integer space, minimizing key relocation during node churn. |
| **Scatter-Gather Query**| Query that must broadcast to all shards because the `WHERE` clause does not include the shard key. |
| **2PC (Two-Phase Commit)** | Distributed atomic commit consensus protocol involving a coordinator and participant nodes. |
| **In-Doubt Transaction** | A 2PC participant state where it has voted to commit but hasn't received the final coordinator verdict. |
| **Compensating Transaction**| A distinct forward transaction designed to reverse the side effects of a previously committed local saga step. |

---

## Example: Consistent Hashing Ring in Python

```python
import hashlib
import bisect

class ConsistentHashRing:
    def __init__(self, nodes=None, replicas=3):
        self.replicas = replicas  # Virtual nodes per physical server
        self.ring = []            # Sorted virtual node hashes
        self.node_map = {}        # Hash -> physical node name
        if nodes:
            for node in nodes:
                self.add_node(node)

    def _hash(self, key: str) -> int:
        return int(hashlib.md5(key.encode('utf-8')).hexdigest(), 16)

    def add_node(self, node: str):
        for i in range(self.replicas):
            vnode_key = f"{node}#vnode{i}"
            h = self._hash(vnode_key)
            bisect.insort(self.ring, h)
            self.node_map[h] = node

    def get_node(self, key: str) -> str:
        if not self.ring:
            return None
        h = self._hash(key)
        idx = bisect.bisect_right(self.ring, h)
        if idx == len(self.ring):
            idx = 0  # Wrap around ring
        return self.node_map[self.ring[idx]]

ring = ConsistentHashRing(nodes=["db-shard-1", "db-shard-2", "db-shard-3"], replicas=100)
print("User 4592 routed to:", ring.get_node("user_4592"))
print("User 9821 routed to:", ring.get_node("user_9821"))
```

---

## Real-World Example: Instagram Sharding Strategy
When Instagram scaled beyond a single PostgreSQL instance:
- They designed a custom **64-bit ID generator**:
  - `41 bits`: Milliseconds since custom epoch (gives 41 years of IDs).
  - `13 bits`: Logical Shard ID (supports up to $2^{13} = 8192$ logical shards).
  - `10 bits`: Auto-incrementing sequence number within the millisecond ($2^{10} = 1024$ unique IDs/ms/shard).
- **Why this is brilliant**: Given *any* photo ID or comment ID, the application router extracts the 13-bit logical shard ID directly from the integer bits and routes the query to the exact database server with zero lookup tables or cross-shard searches!

---

## Important Properties

| Protocol / Pattern | Concurrency | Latency Overhead | Fault Tolerance | Implementation Complexity |
| :--- | :--- | :--- | :--- | :--- |
| **Local Transactions** | Ultra High | None | Hardware dependent | Trivial |
| **2PC (Two-Phase Commit)** | Very Low | High (2 round-trips + disk flushes) | Vulnerable to coordinator crash | Moderate (Built into some engines) |
| **3PC (Three-Phase Commit)** | Low | Very High (3 round-trips) | Non-blocking under crash-stop (fails under network partitions) | High |
| **Sagas (Orchestrated)** | Very High | Asynchronous / Eventual | High (Retry & Compensation) | High (Requires state machine engine like Temporal) |

---

### Common Misconceptions
1. *Misconception*: "Consistent Hashing prevents all data movement when nodes change."
   - *Reality*: **False**. Consistent hashing minimizes data movement ($K/N$ keys move instead of $100\%$ of keys when adding the $N$-th node), but the subset of keys belonging to the newly inserted node's hash ranges must still be physically transferred across the network.
2. *Misconception*: "Sagas provide ACID isolation."
   - *Reality*: **False**. Sagas guarantee **ACD (Atomicity, Consistency, Durability)**, but completely lack **Isolation** ($I$). Intermediate steps are committed to real database tables immediately and visible to external users before the entire saga completes.

---

### What Interviewers Usually Test
- **Consistent Hashing Mechanics**: Why virtual nodes are necessary (to prevent uneven hash clustering and hot spots).
- **The 2PC Blocking Dilemma**: What happens when the coordinator fails after the prepare phase and how participants get stuck.
- **Shard Key Selection Criteria**: Why sharding on `created_at` kills write performance and how compound keys (`tenant_id + created_at`) fix it.

---

### Common Follow-up Questions

1. What is the difference between a "Logical Shard" and a "Physical Shard"?
2. Why is Three-Phase Commit (3PC) rarely implemented in real-world distributed production systems?
3. How do you execute a cross-shard join (`SELECT * FROM users u JOIN orders o ON u.id = o.user_id`) efficiently?
4. What is the "Dual Write" problem when coordinating database updates with a message queue (Kafka)?
5. What is the Transactional Outbox Pattern?

---

### Follow-up Answers

1. **Logical vs Physical Shards**: A physical shard is a physical server or virtual machine instance. A logical shard is an individual database schema/table residing on that machine. By assigning multiple logical shards (e.g., 1024 logical shards) across a few physical servers (e.g., 8 servers), rebalancing becomes trivial: moving a logical shard to a new server requires copying its schema file without re-hashing individual rows.
2. **3PC Partition Vulnerability**: 3PC introduces a `Pre-Commit` state to eliminate coordinator blocking under crash-stop assumptions. However, in real networks subject to asynchronous delays and network partitions, 3PC can easily partition into sub-clusters where one side commits and the other aborts, violating safety.
3. **Cross-Shard Join Strategies**:
   - **Co-location**: Shard both `users` and `orders` on the exact same shard key (`user_id`). This ensures a user's data and their orders live on the same physical database instance, allowing fast local joins.
   - **Application-Level Join**: Query shards in parallel, stream results to an application service, and join in application memory.
4. **The Dual Write Problem**: If an application writes to a database and then publishes an event to Kafka, a crash between the two steps causes state divergence: the DB wrote the order, but Kafka never notified shipping; or Kafka sent the event, but the DB rolled back.
5. **Transactional Outbox Pattern**: Solves the dual write problem by writing the message into an `outbox` table *inside the same local ACID database transaction* as the business update. A background polling worker or CDC process (like Debezium reading the database WAL) reliably streams events from the outbox table to Kafka with at-least-once delivery guarantees.
