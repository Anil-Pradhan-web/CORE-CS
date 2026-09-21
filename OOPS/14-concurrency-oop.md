# Concurrency and Object-Oriented Design

## What is it?

Concurrency in OOP deals with designing classes and objects whose state remains correct and consistent when accessed and manipulated by multiple threads simultaneously. It combines object encapsulation with thread-synchronization primitives.

## Simple Explanation

Imagine a shared bank account with ₹1,000 balance.
- If two people swipe their cards at two different ATMs at the exact same microsecond for ₹800 each:
  - Without concurrency control, both ATMs check the balance (`1000 >= 800`), both dispense cash, and the account drops to `-₹600` (**Race Condition**).
  - With concurrency control, the first ATM acquires a lock on the account object, completes the withdrawal, updates balance to ₹200, and releases the lock. The second ATM then sees the new balance of ₹200 and safely rejects the transaction.

## Hinglish Explanation

Multithreading environment mein jab multiple threads ek hi object ke state ko read aur update karte hain, toh data corrupt ho sakta hai jisko **Race Condition** kehte hain. OOP mein thread-safety achieve karne ke teen main tareeqe hain:
1. **Immutability**: Object banne ke baad uski state change hi nahi ho sakti (zero locks needed).
2. **Synchronization**: Critical section ko lock se protect karna (Java mein `synchronized` / `ReentrantLock`, Python mein `threading.Lock`).
3. **Thread-safe atomic abstractions**: Java ke `AtomicInteger` ya Python ke `queue.Queue`.
Python mein **GIL (Global Interpreter Lock)** hone ke bawajood race conditions ho sakti hain!

## Why Do We Need It?

1. **Prevent Data Inconsistency**: Avoid corrupted bank balances, lost counter increments, and invalid internal object states.
2. **Prevent Deadlocks**: Improper locking orders cause threads to freeze forever waiting on each other.
3. **Maximize Throughput**: Fine-grained locking and lock-free concurrency allow multi-core processors to process thousands of requests concurrently.

## How It Works

### The Golden Rule: Immutability is Inherently Thread-Safe

If state cannot change, threads can read it concurrently without any locks or coordination.

#### Java Immutable Class
```java
public final class ImmutableAccount {
    private final String id;
    private final double balance;

    public ImmutableAccount(String id, double balance) {
        this.id = id;
        this.balance = balance;
    }
    // Return a new object rather than mutating existing state
    public ImmutableAccount withDeposit(double amount) {
        return new ImmutableAccount(this.id, this.balance + amount);
    }
}
```

### Mutex Synchronization: Java vs. Python

#### Java: `synchronized` and Atomic Types
```java
import java.util.concurrent.atomic.AtomicInteger;

public class ThreadSafeCounter {
    private int count = 0;
    private final AtomicInteger atomicCount = new AtomicInteger(0);

    // 1. Intrinsic Lock (Monitor lock on "this")
    public synchronized void incrementSync() {
        count++; // Atomically read, increment, and write back
    }

    // 2. Lock-free Atomic using Hardware CAS (Compare-And-Swap)
    public void incrementAtomic() {
        atomicCount.incrementAndGet();
    }
}
```

#### Python: `threading.Lock`
```python
import threading

class ThreadSafeCounter:
    def __init__(self):
        self._count = 0
        self._lock = threading.Lock()

    def increment(self):
        # GIL does NOT prevent race conditions during bytecode switching!
        with self._lock: # acquire and release automatically
            self._count += 1

    @property
    def value(self) -> int:
        with self._lock:
            return self._count
```

## Important Terminology

| Term | Java | Python | Meaning |
|---|---|---|---|
| **Race Condition** | Flawed interleaved execution | Interleaved execution | Bug where output depends on unpredictable thread execution timing |
| **Critical Section** | Synchronized block | `with lock:` block | Code block accessing shared mutable state |
| **Deadlock** | Mutual thread blockage | Mutual thread blockage | Two or more threads waiting on locks held by each other |
| **Volatile** | `volatile` keyword | N/A | Guarantees CPU cache visibility and disables instruction reordering |
| **GIL** | No GIL (true OS threads) | Global Interpreter Lock | CPython lock allowing only one thread to execute Python bytecode at a time |
| **CAS** | Compare-And-Swap | `multiprocessing` / C-ext | Hardware atomic CPU instruction replacing locks with optimistic retries |

## Example: The Python GIL Myth

> **Myth**: *"Python has a GIL, so I don't need locks for thread safety!"*
> **Reality**: The GIL only protects the Python interpreter's internal memory; it switches threads every 5 milliseconds or 100 bytecodes.
> An operation like `self.count += 1` compiles to 4 bytecodes:
> 1. `LOAD_FAST`
> 2. `LOAD_ATTR`
> 3. `BINARY_ADD`
> 4. `STORE_ATTR`
> If the GIL switches threads between step 2 and 4, two threads overwrite each other, causing lost updates. You **still must lock** shared state in Python!

## Real-World Example

In high-volume e-commerce inventory management:
Instead of synchronizing the entire `Warehouse` object (which serializes all shoppers and destroys throughput), Java uses `ConcurrentHashMap` with striped bucket locks, and atomic integers per SKU (`ConcurrentMap<String, AtomicInteger>`).

## Important Properties

| Mechanism | Strengths | Trade-offs / Risks |
|---|---|---|
| **Immutability** | 100% thread-safe; zero lock overhead | High allocation churn if state updates constantly |
| **Coarse Locks (`synchronized`)**| Simple to reason about | Bottleneck; degrades concurrent throughput |
| **Fine-Grained Locks** | High concurrent scalability | High risk of deadlock if lock acquisition order is inconsistent |
| **Lock-Free CAS (Atomics)** | Extremely fast under low/moderate contention | Busy-spin loop under massive contention |

### Common Misconceptions

* `volatile` in Java does NOT make compound operations atomic: `volatile int count; count++;` is still not thread-safe because increment involves read-modify-write.
* Starting 10 threads in Python does NOT achieve 10x CPU speedup on multi-core machines due to the GIL; CPU-bound tasks in Python must use `multiprocessing`, not `threading`.
* Deadlocks are not random; they happen when 4 conditions (Coffman conditions) hold, primarily **circular wait**. Always acquire multiple locks in a globally consistent order to prevent deadlocks.

### What Interviewers Usually Test

1. Why `volatile` does not guarantee thread-safety for `count++` in Java.
2. Explaining why Python still needs thread synchronization despite the GIL.
3. How to detect and prevent deadlocks (Lock Ordering).
4. `ConcurrentHashMap` vs. `Collections.synchronizedMap()` in Java.
5. Designing an immutable class and explaining its concurrency benefits.

### Common Follow-up Questions

1. What are the 4 Coffman conditions required for a deadlock?
2. What is the difference between `synchronized` method and `ReentrantLock` in Java?
3. How does `ConcurrentHashMap` achieve thread-safety without locking the entire map?
4. How do CPU caches affect multithreaded variable visibility?
5. How should you handle CPU-bound vs. I/O-bound concurrency in Python?

### Follow-up Answers

1. Mutual Exclusion, Hold and Wait, No Preemption, and Circular Wait. Breaking any one condition (e.g., eliminating circular wait via consistent lock ordering) prevents deadlocks entirely.
2. `synchronized` is block-scoped and automatically unlocks; `ReentrantLock` provides advanced capabilities: `tryLock()` with timeout, interruptible lock acquisition, and fair lock queuing.
3. In Java 8+, `ConcurrentHashMap` uses lock-free Compare-And-Swap (CAS) for inserting into empty buckets, and locks only the individual bucket's head node during collisions/updates, allowing thousands of threads to update different buckets concurrently.
4. Each CPU core has L1/L2 caches. A thread updating a variable in its core's cache might not flush to main RAM immediately, so a thread on another core reads stale data. `volatile` in Java flushes writes to main memory and establishes a "happens-before" relationship.
5. For I/O-bound tasks (network, disk, DB), use Python `threading` or `asyncio` (GIL is released during I/O waits). For CPU-bound tasks (data crunching, image processing), use `multiprocessing` to bypass the GIL across separate processes.
