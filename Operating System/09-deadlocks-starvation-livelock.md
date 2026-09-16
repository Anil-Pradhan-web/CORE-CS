# Deadlocks, Starvation, and Livelock

## What is it?

Deadlock is permanent waiting because each participant holds a resource another needs. Starvation is indefinite delay because a participant is repeatedly bypassed. Livelock is active repeated change without useful progress.

## Simple Explanation

Deadlock is two people each holding one key and waiting for the other key. Starvation is one person never reaching the counter because VIPs always cut in. Livelock is two people endlessly stepping aside in the same direction.

## Hinglish Explanation

Deadlock mein threads stuck hain aur kisi ka progress nahi ho raha. Starvation mein system progress kar raha hai but ek unlucky thread wait karta rehta hai. Livelock mein threads active hain—retry/step aside karte hain—but useful kaam complete nahi hota.

## Why Do We Need It?

Locks, connection pools, transactions, and resource limits are necessary but create waiting dependencies. Understanding liveness failures prevents production hangs and cascading timeouts.

## How It Works

Deadlock needs all four Coffman conditions:

1. Mutual exclusion: a resource is exclusive.
2. Hold and wait: a holder requests another resource.
3. No preemption: resource is not forcibly taken.
4. Circular wait: a cycle of dependencies exists.

```text
Thread A holds Lock 1 → waits for Lock 2
Thread B holds Lock 2 → waits for Lock 1
                 └──── circular wait ────┘
```

## Important Terminology

| Term | Meaning |
|---|---|
| Liveness | System eventually makes useful progress |
| Resource allocation graph | Nodes/edges representing requests and holdings |
| Starvation | Task is continually denied service |
| Aging | Raise priority as waiting time increases |
| Livelock | Actors react but do not finish |

## Example

Two backend threads acquire `accountLock` and `ledgerLock` in opposite order. Under load, each may hold one and wait for the other forever.

## Real-World Example

Databases detect transaction deadlocks and abort one transaction. Applications should retry the aborted transaction carefully because the database cannot safely choose business-level behaviour.

## Important Properties

| Problem | CPU activity | System progress | Typical response |
|---|---:|---:|---|
| Deadlock | Often low | No cycle participants progress | Break/abort cycle |
| Starvation | Varies | Others progress | Fairness/aging |
| Livelock | Often high | No useful progress | Backoff/randomness |

### Common Misconceptions

* Deadlock does not require CPU to be at 100%.
* Starvation is not the same as deadlock.
* Killing a random process may damage data; recovery needs policy.

### What Interviewers Usually Test

The four conditions, precise comparison of liveness bugs, and a credible prevention/recovery strategy.

### Common Follow-up Questions

1. Can deadlock happen with one lock?
2. How does lock ordering help?
3. What is priority inversion?
4. Can retry cause livelock?
5. How do databases handle deadlock?

### Follow-up Answers

1. Typical lock deadlock needs multiple resources or a self-deadlock/non-reentrant lock; one resource alone cannot form a multi-party cycle.
2. A global order forces all threads to acquire resources consistently, removing circular wait.
3. A high-priority task waits for a low-priority holder while medium-priority tasks keep preempting that holder.
4. Yes. Identical immediate retries can collide repeatedly; jitter/backoff reduces this risk.
5. They detect wait cycles and abort one transaction, returning an error that the application may retry safely.
