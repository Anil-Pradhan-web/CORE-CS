# Indexing, B-Trees, and B+ Trees

## What is it?

An index is an auxiliary data structure that enables the DBMS query engine to locate specific records in $O(\log N)$ or $O(1)$ time rather than performing an expensive $O(N)$ Full Table Scan. The **B+ Tree** is the undisputed industry standard data structure powering relational database indexing (MySQL InnoDB, PostgreSQL, Oracle, SQL Server).

## Simple Explanation

- **No Index**: Searching for a recipe in a 1,000-page cookbook with no table of contents or index. You must turn and read every single page from page 1 to page 1,000 (**Full Table Scan**).
- **Clustered Index**: The cookbook chapters are physically arranged chronologically by year of creation. You can have only one physical arrangement.
- **Secondary (Non-Clustered) Index**: An alphabetical recipe list in the back: "Tandoori Chicken ➔ See Page 312".
- **B+ Tree**: A multi-level signpost system in a giant library. You read 3 signposts at hallway intersections and arrive directly at the exact shelf.

## Hinglish Explanation

Index database ka search shortcut hai. Agar 10 lakh rows hain aur index nahi hai, toh database disk se saare 10 lakh rows read karega. Index lagane par B+ Tree structure use hota hai jisse sirf 3 ya 4 disk reads mein exact row mil jaati hai:
- **Clustered Index**: Data rows disk par physically index ke order mein store hoti hain (leaf nodes mein actual row data hota hai; ek table mein sirf **1** clustered index ho sakta hai).
- **Secondary Index**: Alag B+ tree hota hai jiske leaf nodes mein pointer (Postgres mein `ctid`, InnoDB mein `Primary Key`) hota hai.
- **B+ Tree vs B-Tree**: B-Tree mein data har node mein hota hai. B+ Tree mein internal nodes mein sirf routing keys hoti hain (high fan-out), aur saare data records leaf nodes mein hote hain jo doubly linked list se connected hote hain (super fast range scans).

## Why Do We Need It?

1. **Eliminates Disk Latency**: Reading 1,000,000 disk pages takes minutes. A B+ tree with fanout of 100 searches 1,000,000 records in $\log_{100}(1000000) = 3$ page reads (~15 milliseconds)!
2. **Range Query Efficiency**: Queries like `WHERE age BETWEEN 20 AND 30` traverse down to 20, then scan horizontally across leaf nodes via pointers.

## How It Works

### 1. B-Tree vs. B+ Tree Architecture

```text
B-Tree (Data in Internal & Leaf Nodes):
                 [ 50 : DataPtr ]
                /                \
      [ 20 : DataPtr ]      [ 70 : DataPtr ]
     *Poor range scans; internal nodes waste space with data pointers*

B+ Tree (Data ONLY in Leaves; Leaves are Doubly Linked):
                     [ 30  |  70 ]           ◄── Internal Node (Keys Only, Routing)
                    /      |      \
         ┌─────────┘       │       └──────────┐
         ▼                 ▼                  ▼
     [ 10 | 20 ]       [ 30 | 50 ]        [ 70 | 90 ]  ◄── Leaf Nodes (Contain Records/Pointers)
         ▲                 ▲                  ▲
         └─── Pointer ────►└─── Pointer ─────►└─── Doubly Linked List!
```
*Because internal nodes store NO data payloads, B+ Tree internal nodes have massive **Fan-out** (often 100 to 1,000 children per node), keeping tree height at just 3 to 4 levels!*

---

### 2. Clustered Index vs. Secondary Index Lookup in MySQL InnoDB

```text
Secondary Index Lookup (e.g., INDEX on email):
  idx_email B+ Tree ──► finds email = 'anil@test.com' ──► yields PK = 42
                                                               │
                                  ┌────────────────────────────┘
                                  ▼
Clustered Index Lookup (Primary Key B+ Tree - Double Lookup / Bookmark Lookup):
  PK B+ Tree ──► traverses root to leaf ──► finds PK = 42 ──► Returns Full Row Payload!
```

---

### 3. The Leftmost Prefix Rule in Composite Indexes

Suppose you create a composite index: `CREATE INDEX idx_user ON users(country, state, city);`

| Query `WHERE` Clause | Uses Index? | Why? |
|---|---|---|
| `WHERE country = 'India'` | **YES** | Matches leftmost prefix (`country`) |
| `WHERE country = 'India' AND state = 'MH'` | **YES** | Matches leftmost 2 columns |
| `WHERE country = 'India' AND city = 'Pune'` | **Partially** | Uses index for `country`, filters `city` in memory |
| `WHERE state = 'MH' AND city = 'Pune'` | **NO (Full Scan)**| Missing leftmost column (`country`)! |
| `WHERE city = 'Pune'` | **NO (Full Scan)**| Missing leftmost column |

---

### 4. Covering Index (Index-Only Scan)

If a query requests **only** columns present inside the index itself:
```sql
-- Index exists on (emp_id, email)
SELECT emp_id, email FROM employees WHERE emp_id = 101;
```
The DBMS satisfies the query directly from the secondary index tree without ever visiting the table heap or clustered index! This is called an **Index-Only Scan** or **Covering Index**.

## Important Terminology

| Term | Meaning |
|---|---|
| **Clustered Index** | Organizes physical table storage on disk; leaf nodes store actual row data |
| **Secondary Index** | Auxiliary index; leaf nodes store index keys and references to base rows |
| **Fan-out** | The number of child pointers an internal node can hold (typically hundreds) |
| **Dense Index** | Contains an index entry for every single search key in the table |
| **Sparse Index** | Contains index entries for only some records (used in clustered storage) |
| **Covering Index** | An index containing all columns requested by a query (zero table visits) |
| **Page Split** | When an insertion exceeds node capacity, splitting the node into two 50% full nodes |

## Example: The Math of B+ Tree Height

Assume:
- Page size = 16 KiB (16,384 bytes).
- Primary Key (BIGINT) + child pointer = 8 + 8 = 16 bytes.
- Fan-out per node = $16,384 / 16 \approx 1,000$ pointers per node.
- Average leaf row size = 200 bytes $\implies$ ~80 rows per leaf page.

Capacity by Height:
- **Height 1 (Root only)**: 80 rows.
- **Height 2**: $1,000 \times 80 = \mathbf{80,000\text{ rows}}$.
- **Height 3**: $1,000 \times 1,000 \times 80 = \mathbf{80,000,000\text{ rows}}$ (80 Million rows!).
- **Height 4**: $\mathbf{80\text{ Billion rows}}$!
*With the top 2 levels cached in RAM buffer pool, searching 80 million rows requires only **1 single physical disk read**!*

## Real-World Example

In high-scale database optimization:
- Using auto-increment sequential integers for Primary Keys ensures new inserts append to the rightmost leaf node of the B+ Tree.
- Using random UUIDv4 causes random insertions into historical leaf pages, triggering frequent **B+ Tree Page Splits**, doubling I/O, and fragmenting disk blocks.

## Important Properties

| Feature | B-Tree | B+ Tree |
|---|---|---|
| **Data Pointers Location** | Stored in internal AND leaf nodes | **Stored strictly in Leaf nodes** |
| **Internal Node Capacity** | Lower (data pointers take space) | **Much Higher (pure routing keys)** |
| **Fan-out & Tree Height** | Lower fan-out, taller tree | **Higher fan-out, shallower tree** |
| **Range Queries** | Inefficient (requires in-order tree traversal)| **Extremely Fast (linear leaf list scan)**|
| **Search Time** | Best case $O(1)$ at root, worst $O(\log N)$ | Uniform search time ($O(\log N)$ to leaf) |

### Common Misconceptions

* A table can have multiple clustered indexes (False: a table can have only **one** clustered index because physical rows can only be sorted on disk in one order).
* Indexes always speed up database systems (False: indexes add write amplification; every `INSERT` or `DELETE` must update the base table AND all secondary B+ Trees).
* `SELECT * WHERE b = 5` will use a composite index `(a, b)` (False: violates the Leftmost Prefix rule).

### What Interviewers Usually Test

1. Differences between B-Trees and B+ Trees (why databases chose B+ Trees).
2. Clustered vs. Non-Clustered indexes and the Bookmark Lookup penalty.
3. The Leftmost Prefix Rule in composite indexes.
4. Designing a Covering Index to optimize hot queries.
5. Why random UUIDs degrade B+ Tree write performance.

### Common Follow-up Questions

1. Why are B+ Trees preferred over Binary Search Trees (BST / AVL / Red-Black) in databases?
2. What is a Bookmark Lookup (Double Lookup) in secondary indexes?
3. How do doubly-linked leaf nodes optimize range queries?
4. What happens during a B+ Tree Page Split on insertion?
5. How does a Covering Index eliminate table access?

### Follow-up Answers

1. BSTs have a fan-out of only 2, creating trees with depth 20–30 for 1 million records (requiring 20–30 disk I/Os). B+ Trees have a fanout of 500–1,000, matching disk block sizes and limiting depth to 3–4.
2. In secondary indexes, the leaf node contains the indexed key and the row's Primary Key. The engine must perform a second search on the clustered index to fetch the rest of the columns.
3. Once the search traverses down to the start of a range (e.g., `WHERE age >= 25`), it simply follows the leaf node's next-pointers sequentially without walking back up the tree.
4. When a leaf page fills up (100% capacity) and a new key arrives, the engine allocates a new page, moves 50% of the keys to the new page, and inserts a routing key into the parent node.
5. If all columns in the `SELECT` and `WHERE` clauses exist within the secondary index, the engine returns data directly from the index leaf without consulting the primary table pages.
