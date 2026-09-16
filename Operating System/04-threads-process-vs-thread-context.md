# Threads, Process vs Thread, and Context Switching

## What is it?

A thread is an execution path within a process. Threads of one process share code, heap, and open files, but each has its own registers, program counter, stack, and thread-local storage.

## Simple Explanation

A process is a restaurant; threads are workers inside it. They share the kitchen and supplies but each worker has their own task list and hands. Switching workers requires saving one worker’s place and restoring another’s.

## Hinglish Explanation

Thread process ke andar execution unit hai. Same process ke threads heap aur global variables share karte hain, isliye communication easy hai but race condition ka risk bhi hota hai. Har thread ka own stack aur CPU execution state hota hai.

## Why Do We Need It?

Threads improve responsiveness and allow concurrent work such as accepting requests while waiting for database/network I/O. Multiple threads can run in parallel on different cores, but shared mutable data needs synchronization.

## How It Works

1. Runtime or OS creates a thread and allocates execution state/stack.
2. Scheduler chooses runnable threads, not whole programs, for CPU time.
3. On preemption/blocking, OS saves registers/program counter and scheduling state.
4. It restores another thread’s state; cache/TLB effects and locking make switches non-free.

```mermaid
flowchart LR
 A[Thread A registers + PC + stack] -->|save| K[Scheduler]
 K -->|restore| B[Thread B registers + PC + stack]
 B --> CPU[CPU runs Thread B]
```

## Important Terminology

| Term | Meaning |
|---|---|
| Concurrency | Multiple tasks make progress over time |
| Parallelism | Tasks execute at the same time on multiple cores |
| Context switch | Save one execution context and restore another |
| User thread | Managed mostly by runtime/library |
| Kernel thread | Scheduling entity known to kernel |
| TLS | Per-thread storage despite shared process address space |

## Example

A Java web server may use a worker pool: each request is handled by a thread while database calls block or async work is awaited. A Python service may use threads for I/O, processes for CPU-bound work due to implementation/runtime constraints.

## Real-World Example

Browsers use multiple processes for isolation and many threads inside those processes for rendering, networking, JavaScript coordination, and background work.

## Important Properties

| Property | Process | Thread |
|---|---|---|
| Address space | Usually separate | Shared within process |
| Isolation | Stronger | Lower from siblings |
| Creation/switch cost | Usually higher | Usually lower, not zero |
| Communication | IPC | Shared memory possible |

### Common Misconceptions

* Threads do not share stacks; each has its own stack.
* Threads are not always faster; contention and context switching can reduce performance.
* Concurrency does not require multiple cores; parallelism does.

### What Interviewers Usually Test

Resource-sharing trade-offs, context-switch overhead, thread safety, and how backend servers handle concurrent requests.

### Common Follow-up Questions

1. Why are threads lighter than processes?
2. What gets saved in a context switch?
3. User thread versus kernel thread?
4. Why is thread safety needed?
5. What happens when one thread blocks?

### Follow-up Answers

1. They reuse their process address space and many resources, so less state must be created than for a new isolated process.
2. CPU registers, program counter, stack pointer, scheduling state, and architecture-specific execution state; address-space state may also change across processes.
3. User threads are scheduled by a runtime; kernel threads are visible to and scheduled by the OS kernel.
4. Shared mutable data can be read or changed in harmful interleavings without synchronization.
5. A blocked kernel thread stops running; other runnable threads, including siblings, may continue if scheduler/runtime design permits.
