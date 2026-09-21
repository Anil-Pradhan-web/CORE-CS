# 15. Hashing & Buffer Pool Management

## What is it?
**Hashing** in DBMS provides constant-time $O(1)$ point-lookup access to disk blocks or in-memory records using hash functions. Unlike static hashing (which suffers from fixed buckets and overflow chains), dynamic hashing schemes (**Extendible Hashing** and **Linear Hashing**) dynamically expand and shrink buckets without full table re-indexing.
**Buffer Pool Management** is the database subsystem responsible for mediating between volatile, low-latency main memory (RAM) and non-volatile, high-latency secondary storage (SSD/HDD). It caches database disk pages in RAM frames, minimizing disk I/O through smart page replacement algorithms (LRU, Clock, LRU-2) and background dirty page flushing.

---

## Simple Explanation
Think of your computer disk like a massive central library warehouse, and RAM as your personal study desk:
- **Buffer Pool**: You cannot read a book directly inside the closed warehouse shelf. You must fetch the book (disk page) onto your study desk (buffer pool frame). If your desk is full, you must decide which book to put back on the shelf (page eviction).
- **Dirty Page**: If you scribbled notes into a book on your desk, you cannot just throw it away—you must write your scribbles back to the warehouse shelf before clearing the desk space.
- **Hashing**: A lightning-fast index card system telling you: "Book ID #4592 is on Shelf 7, Row 3" directly in one step, without searching through a tree of cards.

---

## Hinglish Explanation
Database ka data permanent storage (SSD/HDD) mein blocks/pages (usually 4KB, 8KB ya 16KB) ke roop mein store rehta hai. Lekin CPU directly disk pe computation nahi kar sakta; usko data RAM mein chahiye.
- **Buffer Pool** database engine ka apna dedicated memory cache hota hai jo disk pages ko RAM frames mein hold karta hai. DBMS OS ke page cache pe trust nahi karta kyunki DBMS ko pata hota hai kaunsi query kis order mein pages access karegi.
- Jab koi query page maangti hai, DBMS **Buffer Pool Table (Page Table)** mein check karta hai:
  - Agar page RAM mein hai $\rightarrow$ **Cache Hit** (microseconds).
  - Agar RAM mein nahi hai $\rightarrow$ **Cache Miss** $\rightarrow$ disk se page read karo, free frame mein daalo. Agar frame khali nahi hai, toh kisi purane page ko **Evict** karo.
- **Extendible Hashing**: Jab static hash table full ho jata hai, overflow chain banne lagti hai jo $O(N)$ ban jati hai. Extendible hashing mein ek in-memory **Directory** hoti hai jo pointers rakhti hai buckets ke upar. Jab bucket overflow hoti hai, sirf wahi bucket split hoti hai aur global depth badhakar directory double ho jati hai—poora table rehash nahi karna padta.

---

## Why Do We Need It?
1. **Bridging the Storage Latency Gap**: L1/L2 cache (~1 ns), RAM (~50-100 ns), NVMe SSD (~10-50 $\mu s$), Mechanical HDD (~5-10 ms). Direct disk access kills database throughput; a 99% buffer cache hit ratio keeps queries sub-millisecond.
2. **Predictable Query Execution vs OS Swapping**: The OS file cache does not know transaction semantics (WAL order, locks, dirty page flush dependencies). DBMS-managed buffer pools prevent premature evictions and optimize I/O via asynchronous page flushing.
3. **Handling High-Concurrency Primary Key Lookups**: Hash indexing enables $O(1)$ point queries for key-value stores and memory-optimized relational tables without the $O(\log_B N)$ tree traversal overhead.

---

## How It Works

### 1. Buffer Pool Architecture

```
                       User Queries (SQL Engine)
                                  │
                          Page ID requested
                                  ▼
                 ┌─────────────────────────────────┐
                 │    Buffer Pool Page Table       │
                 │   (Hash Table: PageID -> Frame) │
                 └────────────────┬────────────────┘
                                  │
                  ┌───────────────┴───────────────┐
             Hit  │                               │ Miss
                  ▼                               ▼
       ┌─────────────────────┐       ┌────────────────────────┐
       │ Return Frame in RAM │       │ Allocate Frame Slot    │
       │ (Increment Pin/Ref) │       │ (Run Eviction if full) │
       └─────────────────────┘       └───────────┬────────────┘
                                                 │ Read Page
                                                 ▼
                                     ┌────────────────────────┐
                                     │  Disk Storage (SSD)    │
                                     │   [Page 0][Page 1]...  │
                                     └────────────────────────┘
```

#### Frame Descriptors & Metadata
Every frame in the buffer pool maintains metadata:
- **Frame ID**: Physical index in the allocated RAM array.
- **Page ID**: Disk page currently mapped to this frame.
- **Pin Count (Reference Count)**: Number of concurrent query threads currently reading/writing this frame. A frame with $\text{Pin Count} > 0$ **cannot be evicted**.
- **Dirty Bit (Boolean)**: `True` if data was modified in RAM but not yet synced to disk; requires flushing to disk before eviction.

---

### 2. Extendible Hashing Mechanics
Extendible hashing uses two depth metrics:
- **Global Depth ($d$)**: The number of prefix/suffix bits used to index the directory ($2^d$ directory pointers).
- **Local Depth ($p$)**: The number of bits used by an individual bucket.

```
 Global Depth = 2
 Directory (2^2 = 4 entries)
 ┌─────┬──────────┐
 │ 00  │ ───┐     │
 ├─────┼─── │ ────┤
 │ 01  │ ── │ ─┐  │
 ├─────┼─── │ ─│──┤
 │ 10  │ ───┘  │  │
 ├─────┼───────│──┤
 │ 11  │ ──────┘  │
 └─────┴──────────┘
          │        └──► ┌──────────────────────┐
          │             │ Local Depth = 1      │  Bucket B (ends in 1)
          │             │ Records: [5, 9, 13]  │
          │             └──────────────────────┘
          ▼
 ┌──────────────────────┐
 │ Local Depth = 1      │  Bucket A (ends in 0)
 │ Records: [4, 8, 12]  │
 └──────────────────────┘

Case 1: Insert key 16 (binary 10000 -> ends in 00). Bucket A is FULL.
Condition: Local Depth (1) < Global Depth (2).
Result: Bucket A splits into Bucket A0 (ends in 00) and Bucket A1 (ends in 10).
        Local depth becomes 2. Directory does NOT double.

Case 2: Bucket A0 overflows when Local Depth == Global Depth.
Result: Directory doubles (Global Depth becomes 3, size 2^3 = 8).
```

---

### 3. Page Eviction Algorithms

#### A. Clock / Second-Chance Eviction (Low Overhead LRU Approximation)
- Frames arranged logically in a circular buffer with a `Clock Hand`.
- Each frame has a 1-bit `Reference Bit` (set to 1 when accessed).
- **Eviction Step**:
  1. If `Reference Bit == 1`, reset to `0`, advance hand.
  2. If `Reference Bit == 0` and `Pin Count == 0`:
     - If dirty, queue for asynchronous disk write.
     - If clean, evict immediately and load new page.

```
       Frame 0 [Ref=0] ◄── Hand points here -> EVICT!
     /        \
Frame 3 [Ref=1]  Frame 1 [Ref=1]
     \        /
       Frame 2 [Ref=0]
```

#### B. LRU-K / 2Q Algorithm (Scan Resistance)
Classic LRU suffers from **Sequential Scan Pollution**: A single `SELECT * FROM big_table` reads thousands of cold pages once and clears out entire hot working sets.
- **LRU-2**: Tracks the time of the *last 2 accesses* to every page.
- Pages accessed only once stay in a probationary FIFO queue.
- Only pages accessed at least twice within a time threshold enter the hot LRU queue.

---

## Important Terminology

| Term | Definition |
| :--- | :--- |
| **Buffer Pool** | In-memory RAM array divided into fixed-size frames holding disk pages. |
| **Frame** | Fixed-size memory slot (e.g., 16KB in MySQL InnoDB, 8KB in PostgreSQL). |
| **Pin Count / Latch** | In-memory concurrency primitive protecting a frame from eviction during active query read/write. |
| **Dirty Page** | An in-memory page that has received modifications not yet flushed to disk. |
| **Flusher Thread** | Background thread (e.g., `buf_flush_page_cleaner` in InnoDB) flushing dirty pages to maintain free frames. |
| **Checkpoint** | Safe-point where all dirty pages modified before an LSN are guaranteed flushed to disk. |
| **Extendible Hashing**| Dynamic hashing using an in-memory directory of pointers that doubles only when local depth equals global depth. |
| **Linear Hashing** | Dynamic hashing that splits buckets sequentially one-by-one using two alternating hash functions without a centralized directory. |

---

## Example: Clock Eviction Simulator in Python

```python
class ClockBufferPool:
    def __init__(self, capacity: int):
        self.capacity = capacity
        self.frames = [None] * capacity  # page_ids
        self.ref_bits = [0] * capacity
        self.pinned = [False] * capacity
        self.hand = 0
        self.page_table = {}  # page_id -> frame_index

    def access_page(self, page_id: int):
        if page_id in self.page_table:
            frame_idx = self.page_table[page_id]
            self.ref_bits[frame_idx] = 1
            print(f"[HIT] Page {page_id} in Frame {frame_idx}")
            return frame_idx

        print(f"[MISS] Page {page_id} not in memory. Evicting...")
        frame_idx = self._evict()
        old_page = self.frames[frame_idx]
        if old_page is not None:
            del self.page_table[old_page]

        self.frames[frame_idx] = page_id
        self.ref_bits[frame_idx] = 1
        self.page_table[page_id] = frame_idx
        print(f"Loaded Page {page_id} into Frame {frame_idx}")
        return frame_idx

    def _evict(self) -> int:
        while True:
            if not self.pinned[self.hand]:
                if self.ref_bits[self.hand] == 1:
                    self.ref_bits[self.hand] = 0
                else:
                    evicted_frame = self.hand
                    self.hand = (self.hand + 1) % self.capacity
                    return evicted_frame
            self.hand = (self.hand + 1) % self.capacity

pool = ClockBufferPool(capacity=3)
for p in [10, 20, 30, 10, 40]:
    pool.access_page(p)
```

---

## Real-World Example: MySQL InnoDB Midpoint Insertion Strategy
To prevent sequential table scans (`mysqldump`, large analytical queries) from destroying the buffer pool:
- InnoDB divides its LRU list into **two sub-lists**:
  - **Young / New Sublist** (default: top 63% of list).
  - **Old Sublist** (default: tail 37% of list).
- Newly loaded pages are inserted **at the midpoint** (the head of the Old list), NOT at the very top of the LRU!
- If the page is accessed again after `innodb_old_blocks_time` (default: 1000ms), it is promoted to the Young list.
- Single-pass batch scans touch the page only once and leave it to quickly age out at the tail of the Old sublist without evicting critical transactional cache.

---

## Important Properties

| Feature | Static Hashing | Extendible Hashing | Linear Hashing |
| :--- | :--- | :--- | :--- |
| **Directory Required** | No | Yes (grows by powers of 2) | No directory needed |
| **Splitting Granularity** | Overflows into chained lists | Single overflowing bucket splits | Sequential bucket split via pointer |
| **Worst-Case Lookup** | $O(N)$ (chain degradation) | $O(1)$ (direct directory pointer) | $O(1)$ average, short overflow chains |
| **Reorganization Cost** | Full table rehash | Cheap: 1 bucket allocation + pointer copies | Uniform: incremental single bucket split |

---

### Common Misconceptions
1. *Misconception*: "When an in-memory page is modified, it is immediately written to disk."
   - *Reality*: Immediate write-through kills disk IOPS. Modern DBMSs write modifications to the in-memory log buffer, append to the Write-Ahead Log (WAL) on disk, and mark the buffer frame **dirty**. The dirty page is asynchronously written back to disk much later by background flusher threads.
2. *Misconception*: "Hash indexing is better than B+ Tree indexing because it is $O(1)$."
   - *Reality*: Hash indexes cannot do range scans (`WHERE age BETWEEN 20 AND 30`) or prefix lookups (`WHERE name LIKE 'A%'`). Every range query degrades to a full table scan. B+ Trees remain the default choice for general database engines.

---

### What Interviewers Usually Test
- **Sequential Scan Buffer Pollution**: Why standard LRU fails under full table scans and how engines solve it (Midpoint insertion, 2Q, LRU-K).
- **Pinning vs Latching vs Locking**: Pins prevent frame eviction; Latches protect in-memory data structures from race conditions; Locks protect logical rows/tables across transaction duration.
- **Extendible Hashing Bit Math**: Given directory depth and bucket capacity, calculating directory doubling and pointer redistribution.

---

### Common Follow-up Questions

1. What happens if a transaction modifies a page, marks it dirty, and the database crashes before the buffer pool flushes that frame to disk?
2. What is the difference between a database "Pin" and a database "Lock"?
3. Why does extendible hashing experience performance hiccups when directory doubling occurs?
4. How does the Buffer Pool interact with the Operating System's `fsync` system call?
5. Why do modern DBMS engines (PostgreSQL, MySQL) open disk files with `O_DIRECT` instead of using the Linux Page Cache?

---

### Follow-up Answers

1. **Recovery via WAL**: Before the transaction committed, its changes were appended to the disk-resident Write-Ahead Log (WAL / redo log). During post-crash recovery, the ARIES redo phase reads the log and replays the modifications to bring the page to its committed state.
2. **Pin vs Lock**: A **Pin (or Page Latch)** is a physical, short-lived in-memory counter preventing the buffer manager from evicting a frame while an internal thread reads memory addresses. A **Lock** is a logical, transaction-level concurrency control mechanism held until `COMMIT` or `ROLLBACK` to guarantee ACID isolation.
3. **Directory Doubling Overhead**: When local depth equals global depth and an overflow occurs, the directory array must double in size ($2^d \rightarrow 2^{d+1}$). If the directory is large, allocating contiguous memory and copying pointer references causes a transient latency spike.
4. **`fsync` Durability**: Buffer pool flushes write dirty bytes to OS page buffers via `write()`. Calling `fsync(fd)` forces the OS kernel to flush its storage controllers and drive caches to physical non-volatile storage, guaranteeing durability.
5. **Double Buffering Avoidance (`O_DIRECT`)**: Using standard OS I/O caches pages twice: once in the DBMS Buffer Pool and once in the Linux Page Cache, wasting 50% of available RAM. Furthermore, the OS kernel's background writeback ignores database transaction dependencies and WAL checkpoint ordering. `O_DIRECT` bypasses kernel caching, giving the DBMS total control over I/O scheduling.
