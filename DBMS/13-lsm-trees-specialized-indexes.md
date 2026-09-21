# LSM Trees and Specialized Indexes

## What is it?

While B+ Trees are optimized for read-heavy workloads with in-place updates, **Log-Structured Merge (LSM) Trees** are designed for high-throughput write-heavy workloads by converting random disk writes into sequential append-only writes. Additionally, modern databases leverage **Hash Indexes**, **Bitmap Indexes**, **Inverted Indexes**, and **Spatial (R-Tree/GiST) Indexes** for specialized query patterns.

## Simple Explanation

- **B+ Tree (In-Place Update)**: Like an encyclopedia where you erase and rewrite sentences directly on page 412 whenever information changes. Excellent for readers; tedious and slow for writers.
- **LSM Tree (Append-Only Write)**: Like an accountant's ledger notebook. You never erase past lines; you simply append new entries and corrections at the end of the book. At the end of the day, a clerk reviews the notebook and merges all entries into a clean summary (**Compaction**).
- **Bloom Filter**: A bouncer at the club door with a checklist who can instantly tell you with 100% certainty if a person is **NOT** inside the club, saving you from searching every room.

## Hinglish Explanation

B+ Tree random disk writes karta hai jisse write-heavy workloads (jaise high-speed logs, time-series, IoT) slow ho jaate hain. **LSM Tree** writes ko pehle RAM mein (**MemTable**) append karta hai aur background mein immutable **SSTables (Sorted String Tables)** disk par sequentially dump karta hai:
- **Compaction**: Old SSTables ko merge karke duplicate keys aur deleted items (tombstones) ko saaf karta hai.
- **Bloom Filters**: Fast $O(1)$ probabilistic check karta hai taaki disk par faltu SSTables na padhni padein.
- **Specialized Indexes**: Hash index ($O(1)$ exact match), Bitmap index (low cardinality columns jaise gender/status), Inverted index (Elasticsearch text search), aur R-Tree (geospatial coordinates).

## Why Do We Need It?

1. **Massive Write Throughput**: Sequential disk writes on NVMe SSDs are 10x to 50x faster than random in-place page overwrites. LSM trees turn all writes into sequential operations.
2. **Specialized Query Optimization**: Searching for text keywords ("machine learning") or spatial radius ("restaurants within 2 km") cannot be efficiently performed by standard B+ Trees.

## How It Works

### 1. LSM Tree Architecture & Write Flow

```text
Incoming WRITE ──┬──► In-Memory MemTable (SkipList/Tree)  [RAM - Instant Write]
                 │
                 └──► Write-Ahead Log (WAL) on Disk      [Append-Only - Durability]

When MemTable fills (e.g. 64 MB):
  MemTable freezes and flushes sequentially to Disk as an immutable SSTable!

Disk Structure:
Level 0: [ SSTable 1 ]  [ SSTable 2 ]  [ SSTable 3 ]
             │               │              │
             └───────────────┼──────────────┘
                             ▼ Compaction (Merge-Sort)
Level 1:      [ Merged SSTable A ]   [ Merged SSTable B ]
                             ▼
Level 2:                 [ Larger SSTables ... ]
```

---

### 2. SSTables and Bloom Filters

- **SSTable (Sorted String Table)**: A file of key-value pairs sorted strictly by key on disk.
- **Bloom Filter**: An in-memory bit array with $k$ hash functions.
  - If Bloom filter says **NO**: Key is **definitely NOT present** (bypasses disk read!).
  - If Bloom filter says **YES**: Key is **probably present** (performs disk search; tiny false-positive rate).

---

### 3. Compaction Strategies

1. **Size-Tiered Compaction**: Merges SSTables of similar sizes into larger SSTables (common in write-intensive Cassandra).
2. **Leveled Compaction**: Organizes SSTables into fixed levels ($L_0, L_1, L_2...$). Each level has 10x capacity of previous level; keys do not overlap within levels (used in RocksDB, LevelDB).

---

### 4. Overview of Specialized Index Types

```text
┌───────────────────────┬───────────────────────┬─────────────────────────────┐
│      Index Type       │    Lookup Strategy    │          Best For           │
├───────────────────────┼───────────────────────┼─────────────────────────────┤
│ B+ Tree               │ M-way Balanced Tree   │ General OLTP, Range scans   │
│ LSM Tree              │ MemTable + SSTables   │ Write-heavy, Time-series    │
│ Hash Index            │ Hash Table ($O(1)$)   │ Exact equality point search │
│ Bitmap Index          │ Bit arrays (0s & 1s)  │ Low cardinality, OLAP filters│
│ Inverted Index        │ Token ➔ Doc_IDs map   │ Full-text search (Lucene)   │
│ R-Tree / GiST         │ Bounding boxes        │ Spatial GIS (Lat/Long maps) │
└───────────────────────┴───────────────────────┴─────────────────────────────┘
```

## Important Terminology

| Term | Meaning |
|---|---|
| **MemTable** | In-memory sorted buffer (SkipList or Red-Black Tree) holding recent writes |
| **SSTable** | Sorted String Table; immutable, sorted key-value file on disk |
| **Compaction** | Background process merging sorted SSTables and removing overwritten data |
| **Tombstone** | A deletion marker written to indicate a key has been deleted |
| **Write Amplification**| Ratio of total bytes written to storage vs. bytes requested by user |
| **Read Amplification** | Number of disk reads required to answer a single query |

## Example: Bitmap Index in Action

Suppose a table of 10 million rows has attributes `Gender` (`M`/`F`) and `Status` (`Active`/`Inactive`):

```text
RowID:      1  2  3  4  5
Gender='M': 1  0  1  1  0
Active:     1  1  0  1  0

Query: WHERE Gender='M' AND Status='Active'
Perform bitwise AND on bit vectors in single CPU instruction:
  1 0 1 1 0
& 1 1 0 1 0
  ─────────
= 1 0 0 1 0  ==> Rows 1 and 4 match instantly!
```

## Real-World Example

- **NoSQL & Distributed Systems**: Apache Cassandra, ScyllaDB, RocksDB, and CockroachDB use LSM Trees to sustain hundreds of thousands of concurrent writes per second.
- **Full-Text Search**: Elasticsearch and Apache Solr tokenize document strings into an **Inverted Index**:
  `"database"` ➔ `[Doc 1, Doc 4, Doc 10]`. Searching for two words performs a fast set intersection of document ID lists.

## Important Properties

| Feature | B+ Tree | LSM Tree |
|---|---|---|
| **Write Pattern** | Random in-place page writes | **Sequential append-only writes** |
| **Write Latency** | Higher (page splits, random I/O) | **Extremely Low (writes to RAM)** |
| **Read Latency** | **Fast ($O(\log N)$)** | Slower (checks MemTable + multiple SSTables)|
| **Storage Engine Example**| MySQL InnoDB, PostgreSQL Heap | RocksDB, Cassandra, LevelDB |
| **Deletions** | In-place removal or page merge | Writes **Tombstone** marker; cleaned during compaction |

### Common Misconceptions

* Hash Indexes are NOT always better than B+ Trees; Hash indexes support only exact matches (`WHERE id = 5`), and **cannot** perform range queries (`WHERE age > 30`) or prefix searches.
* Deleting a record in an LSM tree does NOT free disk space immediately; it writes a "Tombstone" marker that reclaims space only when background compaction runs.
* A Bloom filter never produces False Negatives: if an element was inserted, the Bloom filter will **never** say "No".

### What Interviewers Usually Test

1. Differences between B+ Trees and LSM Trees (Write-heavy vs. Read-heavy trade-offs).
2. How Bloom filters eliminate unnecessary disk reads in LSM trees.
3. Why Inverted Indexes are used in full-text search instead of B+ Trees.
4. How Bitmap indexes execute complex boolean filters at hardware speeds.

### Common Follow-up Questions

1. What is Read Amplification and Space Amplification in LSM Trees?
2. Why can't Hash Indexes support range queries?
3. What is a Tombstone in LSM trees?
4. How does a Bloom Filter work?
5. Why are Bitmap indexes poor choices for write-heavy OLTP databases?

### Follow-up Answers

1. Read Amplification is the number of physical disk reads needed to satisfy one logical read (checking multiple SSTables). Space Amplification is the extra disk space consumed by un-compacted, overwritten versions of records.
2. Because hash functions distribute keys randomly across buckets; adjacent keys (like 10 and 11) do not sit in adjacent memory locations.
3. A special deletion marker record appended to the MemTable; when encountered during reads, it indicates the key has been deleted, and removes the key permanently during compaction.
4. It hashes a key with $k$ independent hash functions to set $k$ bit positions in a bit array. To query, it checks if all $k$ bits are 1. If any bit is 0, the item is definitively not in the set.
5. Updating a single row requires locking the entire bitmap vector for that attribute value, which serializes concurrent updates and causes extreme lock contention.
