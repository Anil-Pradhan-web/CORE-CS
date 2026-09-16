# Race Conditions and Critical Sections

## What is it?

A race condition occurs when correctness depends on unpredictable timing of concurrent operations. A critical section is code that accesses shared mutable state and must not be executed by conflicting threads at the same time.

## Simple Explanation

Two cashiers both read a balance of ₹100, each subtract ₹80, and both write back ₹20. The database should show −₹60 or reject one operation, but the unsynchronised result loses an update.

## Hinglish Explanation

Race tab hoti hai jab result execution order par depend kare. Multiple threads same shared variable read-modify-write karein bina lock ke, toh update lost ho sakta hai. Critical section ko protected banana padta hai.

## Why Do We Need It?

Threads share memory for performance and communication. The OS/runtime needs synchronization rules so interleaving does not corrupt data or violate invariants.

## How It Works

1. Thread A reads shared counter `10`.
2. Thread B reads the same `10` before A writes.
3. Both compute `11`.
4. Both write `11`; one increment disappears.

```text
Unsafe counter++
A: read 10 ── compute 11 ── write 11
B:      read 10 ── compute 11 ── write 11
Final: 11, expected: 12
```

## Important Terminology

| Term | Meaning |
|---|---|
| Atomic operation | Appears indivisible to concurrent observers |
| Critical section | Code accessing a shared resource |
| Mutual exclusion | At most one conflicting actor enters critical section |
| Progress | If no one is inside, selection should not be postponed forever |
| Bounded waiting | Waiting has a finite fairness bound |

## Example

Use a database transaction, atomic increment, or lock when concurrent API requests modify the same inventory count. `count++` is not automatically atomic in most languages.

## Real-World Example

In-memory caches and rate limiters often use atomic primitives or striped locks. A distributed system may need database constraints or compare-and-swap, because one process-local mutex cannot protect multiple servers.

## Important Properties

| Fix | Good for | Caveat |
|---|---|---|
| Mutex | One-process shared data | Blocking/contended |
| Atomic CAS | Small lock-free state | Complex retry logic |
| Database transaction | Shared durable data | Latency/contention |
| Message passing | Ownership transfer | Queue/backpressure design |

### Common Misconceptions

* Race condition is a correctness bug, not merely “two threads running.”
* `volatile` alone usually does not make a read-modify-write operation atomic.
* A lock protects only code that consistently uses that same lock.

### What Interviewers Usually Test

Ability to spot an unsafe interleaving, name the shared invariant, and choose an appropriate single-process or distributed fix.

### Common Follow-up Questions

1. Is `x++` atomic?
2. What is an atomic compare-and-swap?
3. Mutex versus database transaction?
4. What is thread-safe code?
5. Can race conditions happen with processes?

### Follow-up Answers

1. Usually no; it contains separate load, add, and store steps unless a language/runtime supplies an atomic operation.
2. It updates a value only if it still equals an expected old value, allowing safe optimistic retry loops.
3. Mutex protects shared memory in one process; a database transaction protects durable shared data and can coordinate multiple clients.
4. Thread-safe code remains correct when called concurrently according to its documented usage rules.
5. Yes, processes can race through shared files, shared memory, databases, or IPC resources.
