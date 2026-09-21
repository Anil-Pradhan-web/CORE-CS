# Object Memory and Lifecycle

## What is it?

Object memory and lifecycle describes how objects are allocated in memory, how their state is maintained during execution, and how their resources are reclaimed when they are no longer reachable. Understanding these internals in Java and Python prevents performance bottlenecks and memory leaks.

## Simple Explanation

- **Stack**: A sticky notepad on your desk. Fast, strictly ordered, used for active function calls and local primitive variables. Cleaned automatically when the function returns.
- **Heap**: A large shared warehouse. When you order an object, space is allocated in the warehouse, and your notepad just holds the warehouse tracking number (reference).
- **Garbage Collection (GC)**: The automated janitor who periodically checks the warehouse and shreds any inventory that no longer has any active tracking slips pointing to it.

## Hinglish Explanation

Jab bhi aap object create karte ho, memory do jagah divide hoti hai: Stack aur Heap. Primitive variables aur function references Stack mein rehte hain jo fast hota hai. Actual object data Heap memory mein store hota hai. Java mein JVM automatic Garbage Collector use karta hai (Generational GC: Young, Old generation). Python primary cleanup ke liye **Reference Counting** use karta hai aur circular references (jo reference counting se clear nahi ho sakte) unke liye cyclic garbage collector use karta hai.

## Why Do We Need It?

Without automatic memory management, developers manually allocate (`malloc`) and free (`free`) memory, leading to dangling pointers (accessing freed memory) or severe memory leaks. However, even with automatic GC, holding unintentional references (e.g., static collections, unclosed listeners) causes out-of-memory crashes in long-running services.

## How It Works

### Stack vs. Heap Reference Model

```text
Stack (Thread-specific, fast)        Heap (Shared across threads)
┌───────────────────────────┐        ┌───────────────────────────────┐
│ main() frame              │        │                               │
│  userRef ─────────────────┼───────►│  User Object                  │
│                           │        │    name: "Anil"               │
│  count = 10 (primitive)   │        │    account ──────────────────┐│
│                           │        │                              ││
│                           │        │  Account Object  ◄───────────┘│
│                           │        │    balance: 50000.0           │
└───────────────────────────┘        └───────────────────────────────┘
```

### Memory Cleanup Comparison

#### Python: Reference Counting + Cyclic GC
```python
import sys
import gc

class Node:
    def __init__(self, value):
        self.value = value
        self.next = None

# Reference count example
a = Node(1)
print(sys.getrefcount(a))  # 2 (variable 'a' + getrefcount arg)
b = a
print(sys.getrefcount(a))  # 3 (variable 'a', 'b' + getrefcount)
del b
print(sys.getrefcount(a))  # 2

# Cyclic reference (requires cyclic GC)
node1 = Node(1)
node2 = Node(2)
node1.next = node2
node2.next = node1  # Circular reference!

del node1
del node2
gc.collect()  # Cyclic garbage collector breaks the cycle and reclaims memory
```

#### Java: Generational Garbage Collection
1. **Eden Space (Young Gen)**: New objects are born here. Most die young.
2. **Survivor Spaces (S0 / S1)**: Objects that survive minor GC copy between S0 and S1.
3. **Tenured / Old Generation**: Objects surviving multiple GC rounds (e.g., age threshold 15) promote here.
4. **Metaspace**: Stores class metadata and bytecodes (outside Java Heap, in native OS memory).

## Important Terminology

| Term | Java | Python | Meaning |
|---|---|---|---|
| **Reference Count** | Not used as primary | Primary mechanism | Tracks how many variables reference an object |
| **Generational GC** | G1, ZGC, Parallel | Generational cyclic GC (0, 1, 2) | Optimizes collection based on object age |
| **Stop-the-World (STW)**| Pauses app threads | Minimal pauses | Period when GC freezes threads to safely trace pointers |
| **Weak Reference** | `WeakReference<T>` | `weakref.ref(obj)` | Reference that does NOT protect object from GC |
| **Memory Leak** | Unintentional retained references | Unintentional references / global dicts | Objects that are unreachable by logic but reachable by GC roots |

## Example: Memory Leak via Static / Global Cache

```java
// JAVA LEAK: Unbounded static cache keeps growing forever
public class MemoryLeakDemo {
    private static final Map<String, byte[]> CACHE = new HashMap<>();

    public void processData(String id) {
        CACHE.put(id, new byte[1024 * 1024]); // 1 MB per call! Never cleared!
    }
}
```

```python
# PYTHON LEAK: Unbounded global list
GLOBAL_CACHE = []

def process_data(id_val: str):
    GLOBAL_CACHE.append([id_val] * 1000000) # Memory grows indefinitely!
```
**Solution**: Use Weak References (`WeakHashMap` in Java, `weakref.WeakValueDictionary` in Python) or LRU caches with explicit size bounds and TTL eviction.

## Real-World Example

- **JVM Tuning in Production**: Backend microservices configure GC flags (e.g., `-XX:+UseG1GC -Xms4g -Xmx4g`) to keep Stop-The-World pause times under 10 milliseconds for high-throughput payment gateways.
- **Python PyPy / CPython**: CPython uses reference counting immediately on object deallocation (`del obj` decrements counter to 0, invoking immediate free), but circular references need the `gc` module.

## Important Properties

| Mechanism | Java | Python (CPython) |
|---|---|---|
| **Primary Reclamation** | Tracing Garbage Collection (G1, ZGC, Shenandoah) | Reference Counting |
| **Cyclic Handling** | Natural (tracing handles cycles effortlessly) | Generational Cyclic Garbage Collector (`gc.collect`) |
| **Destruction Hook** | `Cleaner` / `PhantomReference` (avoid `finalize()`) | `__del__` (dunder del) |
| **Deterministic Cleanup**| `try-with-resources` | Context Manager (`with` block) |

### Common Misconceptions

* `System.gc()` in Java or `gc.collect()` in Python does not guarantee immediate total cleanup and should never be called routinely in production code.
* Automatic Garbage Collection does NOT eliminate memory leaks; if you leave an unused object in a global collection or forget to unregister an event listener, it will never be collected.
* Python's `del x` does NOT delete the object from memory; it only unbinds the name `x` and decrements the object's reference counter by 1.

### What Interviewers Usually Test

1. Stack vs. Heap allocation (where do primitives, local variables, and object references live).
2. How Java GC determines if an object is alive (GC Roots: static fields, thread stacks, JNI handles).
3. How Python handles cyclic references when reference counting fails.
4. Identifying code patterns that cause memory leaks in managed languages.
5. Why `finalize()` is deprecated in Java and the caveats of `__del__` in Python.

### Common Follow-up Questions

1. What are GC Roots in Java?
2. Why is `finalize()` deprecated in Java?
3. How does Python's `weakref` module prevent circular reference leaks?
4. What is a "Stop-the-World" pause?
5. What is the difference between shallow heap and retained heap?

### Follow-up Answers

1. GC Roots are starting points from which memory tracing begins: active thread stack frames, static/global class variables, and JNI global references. Any object unreachable from these roots is eligible for GC.
2. `finalize()` is unpredictable, slow, can cause deadlocks, and allows resurrecting dying objects; modern Java uses `AutoCloseable` or `java.lang.ref.Cleaner`.
3. A `weakref` allows reading an object without increasing its reference count. When the strong references drop to 0, the object is reclaimed, preventing cycles from keeping cache entries alive.
4. A phase in tracing garbage collectors where all application execution threads are temporarily suspended so the GC can safely inspect and move memory without race conditions.
5. **Shallow Heap** is the memory consumed by the object itself (its own fields and header). **Retained Heap** is the total memory freed if this object were garbage collected (including all uniquely referenced child objects).
