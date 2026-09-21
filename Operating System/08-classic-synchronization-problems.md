# Classic Synchronization Problems

## What is it?

Producer-consumer, readers-writers, and dining philosophers are small models that expose real problems: bounded buffers, read/write coordination, resource ordering, fairness, and deadlock risk.

## Simple Explanation

They are thought experiments, like small traffic junctions, that help you learn safe rules before designing a busy API, queue consumer, or database worker pool.

## Hinglish Explanation

Producer-consumer queue bharne/empty hone ka problem hai. Readers-writers shared data ko many readers aur exclusive writer ke saath manage karta hai. Dining philosophers dikhata hai ki multiple locks galat order mein lene se deadlock ho sakta hai.

## Why Do We Need It?

Production systems use the same patterns in work queues, caches, read replicas, connection pools, and lock hierarchies. These examples make hidden failure modes visible.

## How It Works

### 1. Producer-Consumer (Bounded Buffer with Condition Variables)

#### Java Implementation
```java
import java.util.LinkedList;
import java.util.Queue;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;

public class BoundedBuffer<T> {
    private final Queue<T> queue = new LinkedList<>();
    private final int capacity;
    private final ReentrantLock lock = new ReentrantLock();
    private final Condition notFull = lock.newCondition();
    private final Condition notEmpty = lock.newCondition();

    public BoundedBuffer(int capacity) { this.capacity = capacity; }

    public void produce(T item) throws InterruptedException {
        lock.lock();
        try {
            while (queue.size() == capacity) {
                notFull.await(); // Wait while full; releases lock atomically
            }
            queue.add(item);
            notEmpty.signal();  // Notify waiting consumer
        } finally {
            lock.unlock();
        }
    }

    public T consume() throws InterruptedException {
        lock.lock();
        try {
            while (queue.isEmpty()) {
                notEmpty.await(); // Wait while empty
            }
            T item = queue.poll();
            notFull.signal();   // Notify waiting producer
            return item;
        } finally {
            lock.unlock();
        }
    }
}
```

#### Python Implementation
```python
import threading
from collections import deque

class BoundedBuffer:
    def __init__(self, capacity: int):
        self.capacity = capacity
        self.buffer = deque()
        self.lock = threading.Lock()
        self.not_full = threading.Condition(self.lock)
        self.not_empty = threading.Condition(self.lock)

    def produce(self, item):
        with self.not_full:
            while len(self.buffer) == self.capacity:
                self.not_full.wait()
            self.buffer.append(item)
            self.not_empty.notify()

    def consume(self):
        with self.not_empty:
            while len(self.buffer) == 0:
                self.not_empty.wait()
            item = self.buffer.popleft()
            self.not_full.notify()
            return item
```

---

### 2. Readers-Writers Problem

Multiple readers can read concurrently, but a writer requires exclusive access:

```java
import java.util.concurrent.locks.ReentrantReadWriteLock;

public class SharedCache {
    private String data = "";
    private final ReentrantReadWriteLock rwLock = new ReentrantReadWriteLock(true); // Fair mode

    public String readData() {
        rwLock.readLock().lock();
        try {
            return this.data; // Multiple threads can read concurrently
        } finally {
            rwLock.readLock().unlock();
        }
    }

    public void writeData(String newData) {
        rwLock.writeLock().lock(); // Exclusive lock: blocks all readers and writers
        try {
            this.data = newData;
        } finally {
            rwLock.writeLock().unlock();
        }
    }
}
```

---

### 3. Dining Philosophers (Deadlock Prevention via Lock Ordering)

Deadlock occurs if all 5 philosophers pick up their left fork simultaneously (Circular Wait).
**Solution**: Break circular wait by enforcing strict ascending lock ordering (or picking up the lower-numbered fork first).

```python
import threading

class DiningPhilosopher(threading.Thread):
    def __init__(self, philosopher_id: int, left_fork: threading.Lock, right_fork: threading.Lock):
        super().__init__()
        self.id = philosopher_id
        # Always acquire the lower-indexed fork first!
        self.first_fork, self.second_fork = sorted([left_fork, right_fork], key=id)

    def run(self):
        # Strict lock ordering breaks circular wait condition
        with self.first_fork:
            with self.second_fork:
                print(f"Philosopher {self.id} is eating.")
```

## Important Terminology

| Term | Meaning |
|---|---|
| Bounded buffer | Queue with finite capacity |
| Reader preference | Readers may delay writers |
| Writer preference | Writers may delay readers |
| Fairness | Neither side waits indefinitely |
| Lock ordering | Global order for acquiring multiple locks |

## Example

A worker queue uses a bounded channel: producers block/backpressure when memory limit is reached, consumers remove jobs, and shutdown wakes all waiters.

## Real-World Example

A read-heavy configuration cache may use a read-write lock, but high writer frequency or long reads can make it worse than simpler concurrent data structures. Measure contention.

## Important Properties

| Problem | Main lesson | Typical fix |
|---|---|---|
| Producer-consumer | Capacity + ordering | Mutex + conditions/semaphores |
| Readers-writers | Shared reads, exclusive writes | RW lock/fair policy |
| Dining philosophers | Multiple resource deadlock | Lock ordering/arbitrator |

### Common Misconceptions

* More reader concurrency is not always faster under write pressure.
* Semaphores alone do not guarantee fair scheduling.
* Lock ordering prevents one class of deadlock, not all liveness bugs.

### What Interviewers Usually Test

Whether you can write safe pseudocode, identify invariants, and discuss fairness and shutdown—not just recite the named problem.

### Common Follow-up Questions

1. Why need both mutex and semaphores in bounded buffer?
2. How prevent writer starvation?
3. How solve dining philosophers with ordering?
4. What is backpressure?
5. What is a read-write lock?

### Follow-up Answers

1. Semaphores track available slots/items; the mutex protects the actual shared queue structure during modification.
2. Use a fair queue, writer preference, or admission rule that stops new readers once a writer is waiting.
3. Require every philosopher to acquire forks in the same global order, breaking circular wait.
4. Backpressure slows producers when consumers or buffers cannot keep up, preventing unlimited queue growth.
5. It permits concurrent readers but grants writers exclusive access, with policy-dependent fairness.
