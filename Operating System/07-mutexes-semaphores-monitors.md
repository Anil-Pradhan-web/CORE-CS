# Mutexes, Semaphores, and Monitors

## What is it?

A mutex provides ownership-based mutual exclusion. A semaphore is a counter-based synchronization primitive. A monitor combines shared state, mutual exclusion, and condition variables in a structured abstraction.

## Simple Explanation

A mutex is one restroom key: its owner must return it. A counting semaphore is a box of three parking tokens: up to three users may enter. A monitor is a guarded room with rules for waiting until a condition becomes true.

## Hinglish Explanation

Mutex ek owner-based lock hai—jis thread ne lock liya usually wahi unlock karega. Semaphore count maintain karta hai, resource permits ya events signal karne ke liye useful hai. Monitor shared data aur condition wait ko structured way mein handle karta hai.

## Why Do We Need It?

Concurrent code needs both exclusive access and coordination: “only one updater at a time” and “wait until work is available” are different problems.

## How It Works

1. `lock()` on free mutex grants ownership; contenders sleep/spin until released.
2. `wait(P)` on semaphore decrements if count is positive; otherwise waits.
3. `signal(V)` increments semaphore and wakes an eligible waiter.
4. In a monitor, a thread holds monitor lock, checks predicate in a loop, and waits on a condition variable; producer signals after changing state.

```text
lock(m)
while queue is empty:
    condition.wait(m)   # atomically releases lock, later reacquires
item = queue.pop()
unlock(m)
```

## Important Terminology

| Term | Meaning |
|---|---|
| Mutex | Exclusive, owner-based lock |
| Binary semaphore | Counter restricted to 0/1; not necessarily ownership-based |
| Counting semaphore | Counter representing permits/resources |
| Condition variable | Waits for a predicate while paired with a lock |
| Monitor | Encapsulated synchronized shared object |
| Spurious wakeup | Wait may return without desired condition; recheck in loop |

## Example

A Java `synchronized` block supplies monitor-style mutual exclusion. `wait()`/`notifyAll()` coordinate condition waiting; `ReentrantLock` plus `Condition` provides explicit control.

## Real-World Example

A bounded database connection pool uses a semaphore to cap concurrent leases. A mutex protects its internal queue; a condition variable wakes waiting workers when a connection returns.

## Important Properties

| Primitive | Ownership | Main use |
|---|---|---|
| Mutex | Yes | Protect critical section |
| Binary semaphore | Not inherently | Signal/event or exclusion |
| Counting semaphore | Permits count | Resource pool/concurrency cap |
| Condition variable | No state itself | Wait for predicate |

### Common Misconceptions

* Mutex and binary semaphore are not interchangeable by semantics.
* A condition variable does not store the condition; shared state does.
* `wait()` should use a loop because wakeups can be spurious or condition can change again.

### What Interviewers Usually Test

Correct primitive selection, ownership, lost-wakeup avoidance, and practical producer-consumer coordination.

### Common Follow-up Questions

1. Mutex versus semaphore?
2. Why use `while`, not `if`, around condition wait?
3. Can a semaphore be greater than one?
4. What is a reentrant lock?
5. What is lock contention?

### Follow-up Answers

1. Mutex gives one owner exclusive access; semaphore maintains permits and can coordinate events or multiple resources.
2. The thread must recheck because wakeups may be spurious or another thread may consume the condition before it reacquires the lock.
3. Yes. A counting semaphore can represent any nonnegative number of available permits.
4. It lets the thread already holding the lock acquire it again, tracking a recursion count before full release.
5. It is competition among threads for the same lock, causing waiting and reduced scalability.
