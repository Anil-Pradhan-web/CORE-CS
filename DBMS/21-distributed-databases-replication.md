# 21. Distributed Databases & Replication

## What is it?
**Replication** in distributed database systems is the process of storing redundant copies of the same data across multiple physically separate machines connected via a network. Replication achieves high availability (surviving server failures), fault tolerance, reduced read latency (serving queries from geographic locations closer to users), and increased read throughput. The primary replication architectures are **Single-Leader**, **Multi-Leader**, and **Leaderless**.

---

## Simple Explanation
Imagine a newspaper company publishing news across the country:
- **Single-Leader (Leader-Follower)**: There is one chief editor in the central office in Mumbai. Only the chief editor can write or approve articles (**Writes**). Once published, copies are distributed to printing presses in Delhi, Bangalore, and Kolkata (**Replicas**). Readers across the country read from their local press (**Reads**). If the Delhi press breaks down, readers can read from Bangalore.
- **Multi-Leader**: Editors in both Mumbai and New York can publish articles independently. If both publish conflicting breaking news about the same event at the exact same minute, they need a merge protocol to resolve the conflict.
- **Leaderless**: No editor exists. Any journalist can submit an article directly to multiple servers. When a reader wants the truth, they poll 5 different servers and trust whichever version the majority agrees upon (**Quorum consensus**).

---

## Hinglish Explanation
Jab ek single database server traffic handle nahi kar pata ya hardware crash ka risk hota hai, hum **Replication** use karte hain:
- **Single-Leader Architecture**: Ek **Primary (Leader)** node hota hai aur multiple **Read Replicas (Followers)** hote hain. Saare `INSERT`, `UPDATE`, `DELETE` sirf Primary pe hote hain. Primary apne WAL (Write-Ahead Log) changes ko Followers ko bhejta hai. Reads ko hum Followers par distribute kar dete hain taaki Primary par load kam ho.
- **Sync vs Async Replication**:
  - **Synchronous**: Primary wait karta hai jab tak replica confirm na kar de ki data disk pe likh gaya. Zero data loss, lekin write latency badh jati hai aur agar replica down hua toh write block ho jayega.
  - **Asynchronous**: Primary client ko turant `OK` bol deta hai aur background mein replica ko data bhejta hai. Super fast, lekin agar primary crash ho gaya replication complete hone se pehle, toh un-replicated data humesha ke liye udd jayega (**Replication Lag Data Loss**).
- **Replication Lag Anomalies**: Async replication mein user apna hi post submit karke jab refresh karta hai, toh kabhi-kabhi post gayab dikhta hai kyunki read query slow replica par chali gayi (**Read-After-Write inconsistency**).

---

## Why Do We Need It?
1. **High Availability (HA)**: If a database server burns out, a standby replica can be promoted to leader in seconds, minimizing downtime to nearly 99.999% ("five nines").
2. **Horizontal Read Scaling**: A single write leader can feed 10+ read replicas, allowing applications to serve hundreds of thousands of concurrent read queries per second.
3. **Geographic Proximity / Edge Latency**: Placing read replicas in US-East, EU-Central, and AP-South ensures global users experience sub-20ms database queries.

---

## How It Works

### 1. The Three Replication Topologies

```
1. SINGLE-LEADER (Primary-Replica)
           Client Writes               Client Reads
                 │                          ▲
                 ▼                          │
         ┌───────────────┐          ┌───────────────┐
         │ Primary Node  ├─────────►│ Replica Node  │
         │   (Leader)    │  Replication Log │
         └───────────────┘          └───────────────┘

2. MULTI-LEADER (Active-Active)
         ┌───────────────┐          ┌───────────────┐
   ◄────►│ Leader Node A │◄────────►│ Leader Node B │◄────►
         │  (DataCenter 1)│  Bi-dir  │  (DataCenter 2)│
         └───────────────┘  Replication └─────────────┘

3. LEADERLESS (Dynamo-Style Quorum)
                 Client Write/Read Request
                   ┌───────┼───────┐
                   ▼       ▼       ▼
                ┌─────┐ ┌─────┐ ┌─────┐
                │ N1  │ │ N2  │ │ N3  │  (Writes to W nodes,
                └─────┘ └─────┘ └─────┘   Reads from R nodes)
```

---

### 2. Synchronous, Asynchronous & Semi-Synchronous Replication

```
Client         Leader              Follower 1 (Sync)    Follower 2 (Async)
  │               │                        │                    │
  │── Write ─────►│                        │                    │
  │               │── Write to WAL ───────►│                    │
  │               │                        │── Ack ────────────►│ (Background)
  │               │◄─ Ack (Page Written) ──│                    │
  │◄─ Ok (Commit)─│                        │                    │
  │               │── Async WAL Stream ────────────────────────►│
```

- **Synchronous**: Leader waits for follower ACK before replying to client. Guaranteed zero data loss, but one slow/unreachable replica freezes all writes globally.
- **Asynchronous**: Leader replies immediately after writing to its local WAL. High throughput, but primary crash before replication causes data loss during failover.
- **Semi-Synchronous (Industry Standard - MySQL/Postgres)**: Leader blocks until **at least one** follower acknowledges receipt of the log record. Ensures at least two nodes have the data without waiting on every slow follower.

---

### 3. Leaderless Quorums (Dynamo-Style: Cassandra, ScyllaDB)
Parameters:
- $N$: Total number of replicas storing the data.
- $W$: Minimum number of successful replica acknowledgments required for a **Write** to succeed.
- $R$: Minimum number of replicas queried for a **Read** to succeed.

#### The Quorum Condition:
$$W + R > N$$

```
Example: N = 3 replicas [Node 1, Node 2, Node 3]
Set W = 2, R = 2 (W + R = 4 > 3)

Write (W = 2):
- Writes successfully to [Node 1, Node 2] with Timestamp T2.
- Node 3 was unreachable, still has old version T1.

Read (R = 2):
- Reads from [Node 2, Node 3].
- Node 2 returns T2; Node 3 returns T1.
- Because W + R > N, the read set and write set MUST overlap by at least 1 node!
- The client sees T2 > T1, adopts the newest value (T2), and triggers "Read Repair" to update Node 3.
```

---

### 4. Replication Lag Anomalies & Solutions

```
┌───────────────────────────┬───────────────────────────────────┬───────────────────────────────────────┐
│ Anomaly                   │ Symptom                           │ Production Solution                   │
├───────────────────────────┼───────────────────────────────────┼───────────────────────────────────────┤
│ **Read-Your-Writes Lag**  │ User updates profile, refreshes   │ Route reads of user's own profile to  │
│                           │ page, and sees old profile photo! │ Leader for 5 seconds after any write. │
├───────────────────────────┼───────────────────────────────────┼───────────────────────────────────────┤
│ **Monotonic Reads Lag**   │ User refreshes page: Query 1 hits │ Hash user ID to route all reads from  │
│                           │ replica A (fresh); Query 2 hits   │ a specific user to the same replica   │
│                           │ replica B (lagging) $\implies$    │ throughout their session.             │
│                           │ time travels backwards!           │                                       │
├───────────────────────────┼───────────────────────────────────┼───────────────────────────────────────┤
│ **Consistent Prefix**     │ Comments appear out of order:     │ Route causally related messages to    │
│ **Reads Lag**             │ Answer is displayed *before* the  │ the same partition or track Lamport   │
│                           │ question is visible!              │ logical clocks / causality tokens.    │
└───────────────────────────┴───────────────────────────────────┴───────────────────────────────────────┘
```

---

### 5. Automatic Failover & Split-Brain Hazard
When a Primary node stops responding:
1. **Heartbeat Failure Detection**: Replicas detect Primary has missed heartbeats for 10 seconds.
2. **Leader Election**: Replicas run a consensus vote (e.g., Raft protocol or ZooKeeper/Etcd leader lease) to elect the replica with the most up-to-date WAL LSN.
3. **Reconfiguration**: DNS / Virtual IP switches write traffic to the new leader.
4. **Split-Brain Disaster**: If the old leader was only temporarily unresponsive (e.g., network partition / stop-the-world GC pause) and wakes up thinking it is still the leader:
   - Both old and new leaders accept writes simultaneously.
   - **Resolution**: **Fencing / STONITH ("Shoot The Other Node In The Head")**. The consensus cluster cuts the power or network interface of the old primary before promoting the new leader.

---

## Important Terminology

| Term | Definition |
| :--- | :--- |
| **Primary / Leader** | Central node authorized to process write operations and generate replication stream. |
| **Replica / Follower** | Node that ingests replication stream from leader to maintain a read-only mirror of data. |
| **Replication Lag** | The time delay between a write committing on the leader and reflecting on an async replica. |
| **Read-Your-Writes Consistency**| Guarantee that a user will always see their own recent updates immediately after submitting. |
| **Split-Brain** | Catastrophic failure where network partition causes two nodes to believe they are both the active leader. |
| **Quorum ($W + R > N$)** | Minimum number of nodes that must agree for read/write operations to overlap in leaderless systems. |
| **Read Repair** | In leaderless systems, when a reader notices an out-of-date replica during a read, it pushes the newer value to it. |
| **Last-Write-Wins (LWW)**| Conflict resolution strategy discarding older writes based on wall-clock timestamps (risks data loss due to clock drift). |

---

## Example: Production Read-Write Splitting (Java / Spring Data)

```java
// Dynamic routing DataSource switching between Primary and Replicas
public class ReplicationRoutingDataSource extends AbstractRoutingDataSource {
    @Override
    protected Object determineCurrentLookupKey() {
        // If the current transaction is marked readOnly, route to replica pool
        boolean isReadOnly = TransactionSynchronizationManager.isCurrentTransactionReadOnly();
        if (isReadOnly) {
            return "REPLICA";
        }
        return "PRIMARY";
    }
}

// Service layer implementation
@Service
public class OrderService {

    @Transactional(readOnly = false) // Routes strictly to Primary Leader
    public void createOrder(OrderRequest request) {
        orderRepository.save(new Order(request));
    }

    @Transactional(readOnly = true)  // Routes to load-balanced Read Replicas
    public List<Order> getOrderHistory(Long userId) {
        return orderRepository.findByUserId(userId);
    }
}
```

---

## Real-World Example: GitHub Database Outage (Split-Brain)
In 2012 and 2018, major tech platforms suffered prolonged outages due to MySQL split-brain:
- A brief 100ms network partition isolated the active MySQL primary.
- Orchestrator promoted a replica in a secondary data center as the new primary.
- The old primary continued accepting writes from app servers in its local network that had not received the partition alert.
- Result: Two different primary databases diverged, writing conflicting auto-increment IDs and foreign keys. Engineers had to spend days manually diffing binary logs and writing bespoke reconciler scripts to rescue orphaned user commits.

---

## Important Properties

| Dimension | Single-Leader | Multi-Leader | Leaderless |
| :--- | :--- | :--- | :--- |
| **Write Target** | Exactly 1 Leader node | Any Leader (1 per DataCenter) | Any $W$ nodes out of $N$ |
| **Read Target** | Any Replica or Leader | Any Replica or Leader | Any $R$ nodes out of $N$ |
| **Write Conflict Risk** | Impossible (Single serialization point) | **High** (Requires merge/CRDTs) | Handled at read time via version vectors |
| **Failover Complexity** | Moderate (Requires election / consensus) | Complex multi-cluster sync | None (inherently masterless) |
| **Typical Systems** | MySQL, PostgreSQL, MongoDB | DynamoDB Global, CouchDB | Apache Cassandra, ScyllaDB, Amazon Dynamo |

---

### Common Misconceptions
1. *Misconception*: "Asynchronous replication provides Durability in distributed systems."
   - *Reality*: **False**. Async replication guarantees durability only *on the local machine*. If the primary loses power before sending its WAL to replicas, the newly promoted replica will permanently drop the un-replicated commits.
2. *Misconception*: "Quorums guarantee linearizable strong consistency."
   - *Reality*: **Not always**. Even if $W + R > N$, edge cases like concurrent writes, network partitions, sloppy quorums with hinted handoff, and failed writes that partially touched nodes can still produce stale or inconsistent reads without additional consensus layers like Paxos.

---

### What Interviewers Usually Test
- **Explaining the Quorum Equation**: Why $W + R > N$ ensures read-write overlap, and calculating $W$ and $R$ values for read-heavy vs write-heavy workloads.
- **Solving Read-After-Write Consistency**: How to design an application so users do not see stale data immediately after updating their profiles.
- **Split-Brain Mitigation**: What STONITH/fencing tokens are and how distributed consensus (Raft/Paxos) prevents two masters.

---

### Common Follow-up Questions

1. In a leaderless system with $N=5$, how would you configure $W$ and $R$ for a read-heavy system?
2. What is "Replication Lag", and how can you monitor it in PostgreSQL and MySQL?
3. What is the difference between Statement-Based Replication (SBR) and Row-Based Replication (RBR)?
4. Why is "Last-Write-Wins" (LWW) dangerous when using system wall-clock time?
5. What are CRDTs (Conflict-free Replicated Data Types) and where are they used in multi-leader databases?

---

### Follow-up Answers

1. **Read-Heavy Quorum Configuration ($N=5$)**:
   - Set $W = 5$ and $R = 1$ (or $W=4, R=2$).
   - Because reads require querying only 1 node ($R=1$), read latency is ultra-low. However, writes are slower because they must reach all nodes ($W=5$), fitting a read-dominant profile.
2. **Monitoring Replication Lag**:
   - In MySQL: Run `SHOW REPLICA STATUS` and check `Seconds_Behind_Source`.
   - In PostgreSQL: Query `pg_stat_replication` and measure byte difference between `sent_lsn`, `write_lsn`, and `replay_lsn`.
3. **SBR vs RBR**:
   - **Statement-Based Replication (SBR)**: Primary ships raw SQL statements (`UPDATE users SET age = age + 1`). Vulnerable to non-deterministic functions (`NOW()`, `RAND()`, `UUID()`).
   - **Row-Based Replication (RBR)**: Primary ships exact binary diffs of modified bytes/rows. Completely deterministic, avoids function drift, but generates higher network log volume.
4. **Clock Drift with LWW**: Physical hardware quartz clocks drift by milliseconds every day (NTP clock skew / leap seconds). If Node A's clock runs 50ms fast, its writes will perpetually overwrite and overwrite legitimate, newer writes made on Node B whose clock is accurate, causing silent data destruction.
5. **CRDTs**: Conflict-free Replicated Data Types are mathematical data structures (e.g., G-Counter, PN-Counter, Observed-Remove Set) whose internal states can be concurrently modified across multiple leaders and merged deterministically using mathematically proven commutative, associative, and idempotent union operations without central coordination.
