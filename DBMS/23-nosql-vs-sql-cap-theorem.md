# 23. NoSQL vs SQL, CAP & PACELC Theorems

## What is it?
The **CAP Theorem** and **PACELC Theorem** are foundational theoretical frameworks governing distributed data storage trade-offs. The CAP theorem states that a distributed database can guarantee at most two of three properties: **Consistency (Linearizability)**, **Availability**, and **Partition Tolerance**. Because network partitions ($P$) are inevitable in physical infrastructure, systems must choose between **CP** and **AP**.
The **NoSQL** movement arose to satisfy high-throughput horizontal scaling and unstructured data models by trading strict ACID guarantees for **BASE (Basically Available, Soft state, Eventual consistency)** across four primary families: **Key-Value**, **Document**, **Wide-Column**, and **Graph** databases.

---

## Simple Explanation
Imagine two bank branches on opposite sides of a river connected by a telephone line:
- If a storm cuts the telephone wire (**Network Partition $P$**), the branches cannot talk.
- A customer walks into Branch 1 and deposits ₹10,000.
- Another customer walks into Branch 2 and asks: "What is the total account balance?"
  - **Choice 1 (CP - Consistency Priority)**: Branch 2 says: "Sorry, phone line is cut, I refuse to answer until connection is restored." The system remains strictly accurate (Consistent), but Branch 2 is down for reads (**Unavailable**).
  - **Choice 2 (AP - Availability Priority)**: Branch 2 says: "Here is your last known balance (without the new ₹10,000)." The system answers immediately (**Available**), but the data is stale and out-of-sync (**Inconsistent**).

You **cannot** choose "CA" because you cannot stop physical cables from being cut by backhoes!

---

## Hinglish Explanation
Traditional RDBMS (MySQL, PostgreSQL) tabular structures aur ACID transactions ke liye best hote hain. Lekin jab data petabytes mein ho aur har second 500,000 writes handle karni ho, tab relational databases slow ho jate hain.
- **CAP Theorem**:
  - **C (Consistency)**: Har read request par sabse latest write milegi ya error milega.
  - **A (Availability)**: Har non-failing node se response aayega (error nahi aayega), chahe data thoda purana hi kyun na ho.
  - **P (Partition Tolerance)**: Agar network drop ho jaye ya servers aapas mein baat na kar sakein, system crash nahi karega.
  - *Reality*: Distributed world mein network partition unavoidable hai. Isliye real choice sirf **CP** (e.g., MongoDB, HBase, Spanner) ya **AP** (e.g., Cassandra, DynamoDB, CouchDB) ke beech hoti hai.
- **PACELC Theorem**: CAP theorem sirf tab baat karta hai jab network toot-ta hai ($P$). Lekin normal time par jab network sahi chal raha hota hai tab kya? PACELC kehta hai: **If Partition ($P$), choose Availability ($A$) or Consistency ($C$); ELSE ($E$), choose Latency ($L$) or Consistency ($C$)**.
- **ACID vs BASE**:
  - **ACID**: Strict, immediate consistency, bank ledgers.
  - **BASE**: Basically Available (system accessible), Soft State (data updates ho rahi hain), Eventual Consistency (kuch seconds baad sabhi nodes sync ho jayenge; e.g., YouTube video view count).

---

## Why Do We Need It?
1. **Navigating Distributed Trade-Offs Without Illusion**: Prevents engineers from attempting to design impossible "zero-latency, strongly consistent, 100% available" systems.
2. **Matching Storage Engine to Data Topology**: Storing deeply nested JSON documents in a flat relational table requires painful joins; storing complex social network graphs in SQL requires slow recursive CTEs.
3. **Handling Extreme Write Throughput**: Wide-column LSM stores (Cassandra) handle millions of write operations per second where B+ Tree relational databases bottleneck on disk page locking.

---

## How It Works

### 1. The CAP Theorem Triangle

```
                         Consistency (C)
                         (Linearizability)
                               / \
                              /   \
                             /     \
                       CP   /       \   CA
                    Systems/         \ Systems
                          /           \ (Single-node RDBMS;
                         /             \  impossible across
                        /               \  real networks!)
                       /                 \
                      /═══════════════════\
       Availability (A)        AP       Partition Tolerance (P)
     (Non-failing nodes     Systems      (Survives dropped network
       always respond)                    packets between nodes)
```

- **CP Systems** (e.g., Spanner, HBase, ZooKeeper, etcd): Prefer to fail or timeout a request rather than serve stale or diverging data during a partition.
- **AP Systems** (e.g., Cassandra, CouchDB, DynamoDB): Accept writes and reads on both sides of a network partition; sync and resolve conflicts later via vector clocks or CRDTs.
- **Why "CA" is a Myth in Distributed Systems**: A system cannot "reject" partitions. Network switches drop packets, cables get severed, and cloud instances experience pause pauses. A distributed system that does not tolerate partitions simply dies on the first network hiccup.

---

### 2. The PACELC Theorem

Formulated by Daniel Abadi to address the 99.9% of time when the network is **NOT** partitioned:

$$\text{If } \mathbf{P} \implies (\mathbf{A} \lor \mathbf{C}) \quad \mathbf{E}\text{lse} \implies (\mathbf{L} \lor \mathbf{C})$$

```
                        PACELC CLASSIFICATION
                 ┌────────────────────────────────┐
                 │ Is there a Network Partition?  │
                 └──────────────┬─────────────────┘
                   Yes /              \ No (Normal Ops)
                      ▼                ▼
             ┌────────────────┐   ┌────────────────┐
             │  Choose A or C │   │ Choose L or C  │
             └────────────────┘   └────────────────┘
```

#### PACELC System Matrix:
- **PC/EC** (e.g., Bigtable, HBase, Spanner): Under partition, choose Consistency. During normal ops, choose Consistency over Latency (waits for quorum sync).
- **PA/EL** (e.g., Cassandra, DynamoDB, Riak): Under partition, choose Availability. During normal ops, choose Latency over Consistency (serves reads immediately from local node).
- **PA/EC** (e.g., MongoDB with primary write concern): Under partition, available for reads. In normal ops, waits for majority commit (Consistency).

---

### 3. ACID vs BASE Comparison

```
┌──────────────────────────────────────┬──────────────────────────────────────┐
│           ACID (Relational)          │             BASE (NoSQL)             │
├──────────────────────────────────────┼──────────────────────────────────────┤
│ Atomicity (All or nothing)           │ Basically Available (System stays up)│
│ Consistency (Invariants maintained)  │ Soft State (Values float over time)  │
│ Isolation (Transactions isolated)    │ Eventual Consistency (Syncs later)   │
│ Durability (Committed data survives) │ Optimistic scalability               │
└──────────────────────────────────────┴──────────────────────────────────────┘
```

---

### 4. The Four NoSQL Database Families

```
                               NoSQL FAMILIES
                                     │
         ┌───────────────────┬───────┴───────────┬───────────────────┐
         ▼                   ▼                   ▼                   ▼
    KEY-VALUE            DOCUMENT           WIDE-COLUMN            GRAPH
   (Redis, Memcached,   (MongoDB, Couchbase) (Cassandra, ScyllaDB, (Neo4j, Amazon
    AWS DynamoDB)                             HBase)                Neptune)
         │                   │                   │                   │
   O(1) Hash Map        Hierarchical JSON    Tabular Map-of-Maps  Nodes, Edges &
   by primary key       documents with       with dynamic sparse  Index-Free
                        nested arrays        columns (LSM Tree)   Adjacency
```

#### A. Key-Value Stores (Redis)
- **Structure**: Dictionary mapping string keys to arbitrary values (strings, hashes, lists, sets, sorted sets).
- **Ideal For**: In-memory caching, session storage, rate limiting, pub/sub queues, leaderboards.

#### B. Document Stores (MongoDB)
- **Structure**: Stores records as semi-structured JSON/BSON documents. Documents are self-describing; schema can vary from row to row.
- **Ideal For**: Content management systems, e-commerce product catalogs with heterogeneous attributes, user profiles.

#### C. Wide-Column Stores (Cassandra)
- **Structure**: Multi-dimensional sorted map: `Table[RowKey][ColumnFamily][ColumnKey][Timestamp] -> Value`. Built on LSM Trees and SSTables.
- **Ideal For**: High-throughput telemetry/IoT time-series, log ingestion, chat message history, fraud audit tracking.

#### D. Graph Databases (Neo4j)
- **Structure**: Stores data as **Nodes** (entities), **Edges** (relationships), and **Properties** (key-value pairs). Features **Index-Free Adjacency** (each node holds direct physical memory pointers to its neighboring nodes).
- **Ideal For**: Social networks ("Friends of friends"), fraud rings, recommendation engines, knowledge graphs.

---

## Important Terminology

| Term | Definition |
| :--- | :--- |
| **Linearizability (CAP-C)**| Strongest consistency model where all operations appear to execute instantaneously on a single copy of data. |
| **Eventual Consistency** | Weak consistency guarantee that if no new updates are made, all replicas will eventually converge to the same value. |
| **Index-Free Adjacency** | Graph database architecture where each node stores direct memory references to neighboring nodes, making edge traversal $O(1)$. |
| **BSON** | Binary JSON format used by MongoDB supporting additional data types (Date, Int64, Binary) and fast parsing. |
| **Column Family** | In wide-column databases, a logical grouping of related columns stored together on disk. |
| **Read Repair** | Client or server pushing updated data to a stale replica detected during a read quorum operation. |
| **Tombstone** | A soft-delete marker in LSM-based stores (Cassandra) recording that a record has been deleted without immediate disk overwrite. |

---

## Example: Graph Query (Cypher) vs SQL Recursive CTE

### Problem: Find all friends-of-friends of Alice up to 3 degrees of separation

#### SQL (Relational Recursive Common Table Expression)
```sql
WITH RECURSIVE FriendGraph AS (
    -- Anchor: 1st degree friends
    SELECT friend_id, 1 AS depth
    FROM friendships
    WHERE user_id = 'Alice'
    
    UNION
    
    -- Recursive member: Traverse next degree
    SELECT f.friend_id, fg.depth + 1
    FROM friendships f
    JOIN FriendGraph fg ON f.user_id = fg.friend_id
    WHERE fg.depth < 3
)
SELECT DISTINCT friend_id FROM FriendGraph;
-- Extremely expensive! Multiple self-joins and table scans on multi-million row table.
```

#### Neo4j (Cypher Graph Query Language)
```cypher
MATCH (alice:User {name: 'Alice'})-[:FRIEND*1..3]->(fof:User)
RETURN DISTINCT fof.name;
-- Ultra fast! Traverses direct memory pointers (index-free adjacency) without scanning join tables.
```

---

## Real-World Example: Uber Storage Evolution
When Uber started:
1. **Phase 1 (PostgreSQL)**: Scaled early dispatch logic on single Postgres instances. Ran into connection limits, replication lag, and table bloat.
2. **Phase 2 (Schemaless / MySQL Shards)**: Built an append-only, fault-tolerant datastore on top of partitioned MySQL instances (using MySQL purely as a fast B+ tree key-value storage engine).
3. **Phase 3 (Cassandra & Docstore)**: Used Apache Cassandra for real-time driver GPS telemetry pings (millions of writes per second with high write availability).
4. **Phase 4 (Specialized Engines)**: Integrated Redis for sub-millisecond driver geohash caching and Elasticsearch for rider receipt searching.

---

## Important Properties: Architectural Selection Decision Matrix

| Workload Requirement | Recommended Database Type | Example Technologies | Primary Justification |
| :--- | :--- | :--- | :--- |
| **Financial Ledger / ACID Invariants** | Relational (RDBMS) | PostgreSQL, MySQL, CockroachDB | Strict ACID, foreign key constraints, 2PL/MVCC serializability. |
| **Sub-Millisecond Caching & Sessions** | In-Memory Key-Value | Redis, Memcached | In-memory RAM storage, $O(1)$ operations, rich data structures. |
| **Dynamic E-Commerce Catalog** | Document Store | MongoDB, AWS DocumentDB | Nested schemas, schema evolution per product category. |
| **High-Volume Time-Series / Telemetry** | Wide-Column / LSM | Apache Cassandra, ScyllaDB | Append-only SSTable writes, no read-before-write, horizontal scale. |
| **Complex Relationship Traversal** | Graph Database | Neo4j, Amazon Neptune | Index-free adjacency, traverses relationships without join explosion. |
| **Full-Text Inverted Search** | Search Engine | Elasticsearch, OpenSearch | BM25 text relevance, tokenizers, n-gram faceted fuzzy search. |

---

### Common Misconceptions
1. *Misconception*: "NoSQL databases do not support ACID transactions."
   - *Reality*: **Outdated**. Modern NoSQL engines have evolved: MongoDB introduced multi-document ACID transactions in v4.0; CockroachDB and Google Spanner provide distributed serializable ACID transactions across NoSQL horizontal architectures (NewSQL).
2. *Misconception*: "You can build a 'CA' distributed database by choosing strong hardware."
   - *Reality*: **Impossible**. "CA" means a system that is Consistent and Available, but does NOT tolerate network partitions. Because network partitions are physical phenomena of the universe (hardware switches fail, optic cables break, cloud packets drop), a distributed system *cannot choose to ignore partitions*. Under partition, it *must* either reject requests (CP) or serve stale data (AP).

---

### What Interviewers Usually Test
- **Explaining the PACELC Theorem**: Extending CAP to normal (non-partitioned) operations (Latency vs Consistency).
- **CAP Classification of Famous Databases**: Categorizing Cassandra (AP), HBase/MongoDB (CP), and Spanner (CP).
- **Index-Free Adjacency**: Why graph databases outperform SQL joins for deep network hops.

---

### Common Follow-up Questions

1. How can Google Spanner claim "External Consistency" (Linearizability) while offering 99.999% Availability?
2. What is the difference between Strong Consistency, Eventual Consistency, and Causal Consistency?
3. Why does Cassandra excel at high-volume write operations compared to MySQL?
4. What is a "Tombstone" in Cassandra, and what performance problem does it cause if records are deleted frequently?
5. When should you use PostgreSQL's native `JSONB` column instead of spinning up a separate MongoDB cluster?

---

### Follow-up Answers

1. **Google Spanner's "TrueTime"**: Spanner is technically a **CP system** under CAP (if all data centers partition, it sacrifices availability for consistency). However, it achieves 99.999% availability in practice because Google owns private redundant fiber networks, and uses GPS receivers and atomic clocks in every data center (**TrueTime API**) to bound clock uncertainty to $\le 7$ms, enabling lock-free distributed read transactions.
2. **Consistency Spectrum**:
   - **Strong (Linearizable)**: Immediate visibility globally; every read returns the most recent write.
   - **Causal**: Operations that are causally related are seen by every node in the same order; concurrent operations can be observed in different orders.
   - **Eventual**: Lowest guarantee; if writes stop, all nodes will converge to the same value at an unspecified future time.
3. **Cassandra Write Superiority**: Cassandra appends writes sequentially to an in-memory MemTable and an append-only CommitLog on disk. It performs **zero in-place page reads or updates** and does not manage B+ Tree balance or page locks during writes.
4. **Cassandra Tombstones**: In Cassandra's LSM model, deletes write a marker called a **Tombstone** with a timestamp. The old data is not erased until major compaction occurs. If a table receives millions of deletes, range scans must read and filter through millions of tombstones, causing `TombstoneOverwhelmingException` and query timeouts.
5. **Postgres `JSONB` vs MongoDB**: If 80% of your application needs relational tables and ACID transactions, but a few entities (user metadata, config JSONs) require schema flexibility, PostgreSQL's `JSONB` data type (with GIN index support) provides document storage inside a rock-solid relational engine. A separate MongoDB cluster adds operational overhead and eliminates cross-table joins.
