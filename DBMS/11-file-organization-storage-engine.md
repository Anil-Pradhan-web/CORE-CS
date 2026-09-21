# File Organization and Storage Engine Internals

## What is it?

Storage engines are the foundational subsystems of a DBMS responsible for mapping abstract logical relations (tables and rows) to physical hardware bytes (disk blocks, SSD pages, and memory buffers). Understanding page layouts, slotted-page architectures, and row-vs-columnar formats reveals how databases minimize expensive I/O operations.

## Simple Explanation

A library doesn't scatter books randomly on the floor:
- It groups pages into bound books of fixed dimensions (Database **Pages**, typically 4 KiB, 8 KiB, or 16 KiB).
- Each page has a table of contents at the top pointing to where chapters begin (**Slotted-Page Architecture**).
- When looking for a specific sentence, an index points directly to page 240, slot 4, avoiding reading the whole library.

## Hinglish Explanation

Database tables physical disk par direct raw rows ki tarah nahi save hote. Disk I/O hamesha fixed-size **Pages / Blocks** (e.g., InnoDB 16KB, Postgres 8KB) mein hota hai:
- **Slotted Page Architecture**: Page ke top par slot directory hoti hai jo niche grow karti hai, aur actual record data bottom se upar grow karta hai.
- **Row-Oriented (OLTP)**: Poora row ek sath store hota hai (Fast write, fast single-row fetch).
- **Column-Oriented (OLAP)**: Har column alag file/block mein store hota hai (Huge compression, super fast aggregation like `SUM(salary)`).
- **Storage Engines**: MySQL ka InnoDB (row-level lock + clustered index) vs PostgreSQL (heap tables + MVCC tuple headers).

## Why Do We Need It?

1. **Minimizing Disk I/O**: Accessing RAM takes ~100 nanoseconds, while reading from NVMe SSD takes ~50 microseconds and magnetic disk takes ~10 milliseconds (100,000x slower). Storage engines structure pages to pack maximum relevant data into each single read.
2. **Variable-Length Record Handling**: VARCHARs, TEXT, and JSON fields change size upon update; fixed offsets would cause massive internal fragmentation.

## How It Works

### 1. Slotted-Page Architecture (The Industry Standard)

Used by PostgreSQL, SQLite, MySQL InnoDB, and Oracle to store variable-length records inside fixed-size pages:

```text
┌───────────────────────────────────────────────────────────┐
│ Page Header: Checksum, LSN, Free space pointer, Slot count │
├───────────────────────────────────────────────────────────┤
│ Slot 0 (Offset: 4010, Length: 86)                         │ ──┐
│ Slot 1 (Offset: 3880, Length: 130)                        │ ──┼─┐
│ Slot 2 (Offset: 3750, Length: 130)                        │ ──┼─┼─┐
│  ▼ (Slot directory grows DOWNWARD)                        │   │ │ │
│                                                           │   │ │ │
│                     [ Free Space ]                        │   │ │ │
│                                                           │   │ │ │
│  ▲ (Record payloads grow UPWARD)                          │   │ │ │
│ Tuple 2 Payload (Offset 3750) ◄───────────────────────────┘   │ │
│ Tuple 1 Payload (Offset 3880) ◄───────────────────────────────┘ │
│ Tuple 0 Payload (Offset 4010) ◄─────────────────────────────────┘
└───────────────────────────────────────────────────────────┘
```
- **Tuple Identifier (TID / Record ID)**: Represented as `(Page_Number, Slot_Number)` (e.g., `ctid` in Postgres).
- If a row is updated and changes size, the pointer in the slot changes, but the tuple's external `(Page, Slot)` address remains identical—preventing cascading index updates!

---

### 2. Row-Store (NSM) vs. Column-Store (DSM)

```text
Relational Data:
Row 1: [ID: 1, Name: Anil, Age: 25, Country: India]
Row 2: [ID: 2, Name: Bob,  Age: 30, Country: USA]

Row-Oriented Layout (OLTP - e.g., Postgres, InnoDB):
Page 1: [1, Anil, 25, India] [2, Bob, 30, USA]
*Ideal for: INSERT, UPDATE, and SELECT * WHERE ID = 1*

Column-Oriented Layout (OLAP - e.g., ClickHouse, Snowflake, Parquet):
Page 1 (ID):      [1, 2]
Page 2 (Name):    [Anil, Bob]
Page 3 (Age):     [25, 30]
Page 4 (Country): [India, USA]
*Ideal for: SELECT AVG(Age) FROM users; (Reads ONLY Page 3! Saves 75% I/O!)*
```

---

### 3. Big Columns: TOAST in PostgreSQL

What happens when a single row has a 5 MB image or JSON document, but a page is only 8 KiB?
PostgreSQL uses **TOAST (The Oversized-Attribute Storage Technique)**:
1. Compresses large attributes using LZ4 or pglz.
2. If still > 2 KiB, breaks data into multiple chunks and moves them into a separate auxiliary TOAST table.
3. Leaves a tiny 18-byte pointer in the main table row.

## Important Terminology

| Term | Meaning |
|---|---|
| **Page / Block** | The atomic unit of transfer between disk and DBMS memory (typically 4KB to 16KB) |
| **Slotted Page** | Page architecture combining a top slot array with bottom record payloads |
| **TID / ctid / RowID** | Physical address of a record on disk: `(Page_ID, Slot_Index)` |
| **Row-Store (NSM)** | N-ary Storage Model; stores all attributes of a single row contiguously |
| **Column-Store (DSM)**| Decomposition Storage Model; stores all values of a single column contiguously |
| **Run-Length Encoding**| Compression technique in column stores encoding consecutive duplicate values |

## Real-World Example: MySQL InnoDB vs. MyISAM

| Feature | InnoDB | MyISAM (Legacy) |
|---|---|---|
| **Storage Structure** | **Clustered Index (B+ Tree)** | Heap table + separate index files |
| **Transactions & ACID**| **Full ACID support (Undo/Redo logs)**| None (non-transactional) |
| **Locking Granularity**| **Row-level locking (MVCC)** | Table-level locking only |
| **Crash Recovery** | Automatic crash recovery via WAL | Requires slow `REPAIR TABLE` |
| **Foreign Keys** | Fully supported | Not supported |

## Important Properties

| Dimension | Row-Store (OLTP) | Column-Store (OLAP) |
|---|---|---|
| **Best Query Type** | Point lookups (`SELECT * WHERE ID=?`)| Aggregations (`SELECT SUM(x), AVG(y)`)|
| **Write Performance** | **Very High** (appends full tuple)| Slower (must write to multiple column files)|
| **Compression Ratio** | Low (heterogeneous data types in row)| **Extreme (10x-50x)** (homogeneous column types)|
| **I/O Efficiency** | Reads unused columns in wide tables | Reads **strictly** required columns |

### Common Misconceptions

* A database does not read bytes individually from SSD; the OS and storage engine always transfer entire physical **Pages** (e.g., reading 1 column of 1 row loads the entire 16 KiB page into the buffer pool).
* In PostgreSQL, tables are Heap files (unordered), and all indexes point to the physical tuple ID (`ctid`). In MySQL InnoDB, the table IS a clustered B+ Tree organized by Primary Key.
* Columnar storage is not suitable for high-frequency transactional point updates (e.g., changing an order status 1,000 times a second).

### What Interviewers Usually Test

1. How Slotted-Page architecture handles variable-length records and updates.
2. Trade-offs between Row-Oriented and Column-Oriented storage.
3. Differences between MySQL InnoDB and MyISAM storage engines.
4. How databases store oversized records (e.g., Postgres TOAST).

### Common Follow-up Questions

1. Why do storage engines use Slotted Pages instead of storing records at fixed offsets?
2. What is Run-Length Encoding (RLE) in columnar databases?
3. What is the difference between physical RowID and logical Primary Key?
4. How does in-place record updating handle an attribute growing larger than available page free space?
5. Why are column-oriented databases able to achieve significantly higher compression ratios?

### Follow-up Answers

1. Because variable-length attributes (`VARCHAR`, `TEXT`) make fixed offsets impossible. Slotted pages allow records to shift during updates while keeping the slot pointer index identical, preventing index invalidation.
2. A compression algorithm replacing repeating values with a count and value pair (e.g., storing `[USA, USA, USA, USA]` as `(4, USA)`), saving massive storage in sorted column blocks.
3. A RowID/TID represents physical disk coordinates `(Block, Slot)`. A Primary Key is a logical unique business identifier that remains constant even if the row moves to a different physical page.
4. If the updated record no longer fits in the free space of the current page, the storage engine moves the tuple to a new page and leaves a forwarding pointer (or updates index references) to avoid breaking referential consistency.
5. Because values stored together in a columnar page share the exact same data type and domain characteristics (e.g., all integers or all timestamps), making them ideal for bit-packing, delta encoding, and dictionary compression.
