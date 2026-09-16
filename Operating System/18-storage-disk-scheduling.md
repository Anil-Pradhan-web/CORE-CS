# Storage, Disk Management, and Disk Scheduling

## What is it?

Storage persists data beyond RAM. HDDs have mechanical seek/rotation costs; SSDs use flash with controller-managed mapping, wear leveling, and parallelism. Disk scheduling orders I/O requests to reduce delay and improve fairness.

## Simple Explanation

An HDD is a record player: moving the arm costs time, so request order matters a lot. An SSD has no moving arm, but it still has queues, flash erase/write behaviour, and controller limits.

## Hinglish Explanation

HDD mein head movement aur rotational delay hota hai, isliye SCAN/SSTF scheduling useful hota hai. SSD mechanically fast hai but infinite fast nahi—queue depth, write amplification, garbage collection aur controller behavior performance affect karte hain.

## Why Do We Need It?

I/O is much slower than CPU/RAM. Scheduling, caching, batching, and asynchronous I/O reduce waiting while preserving fairness and durability.

## How It Works

1. Process issues file/block I/O; kernel/page cache may satisfy or batch it.
2. Block layer queues requests and device/controller schedules work.
3. Device completes request, often using DMA and interrupt completion.
4. Kernel wakes waiting tasks or completes asynchronous callbacks.

| Algorithm | Idea | HDD relevance |
|---|---|---|
| FCFS | Arrival order | Fair/simple, may seek a lot |
| SSTF | Closest request next | Lower seek, starvation risk |
| SCAN | Sweep in one direction then reverse | Elevator-like fairness |
| C-SCAN | Sweep one direction, jump back | More uniform wait |
| LOOK/C-LOOK | Stop at last request, not disk end | Avoid unnecessary travel |

## Important Terminology

| Term | Meaning |
|---|---|
| Seek time | HDD head movement time |
| Rotational latency | Wait for HDD sector rotation |
| IOPS | I/O operations per second |
| Throughput | Bytes transferred per second |
| Queue depth | Outstanding I/O requests |
| Write amplification | SSD writes internally more than host writes |

## Example

`iostat -xz 1` helps inspect device utilization, queueing, latency, and throughput. High await plus near-saturated device can indicate storage bottleneck, but interpret by device/workload type.

## Real-World Example

Databases use write-ahead logs and `fsync` for durability. SSDs improve random I/O, but long synchronous write latency still impacts transaction p99 and must be measured.

## Important Properties

| Property | HDD | SSD |
|---|---|---|
| Mechanical seek | High | None |
| Random I/O | Expensive | Much better, not free |
| Wear concern | Mechanical aging | Flash program/erase cycles |
| Scheduling impact | Often strong | Still queue/controller dependent |

### Common Misconceptions

* SSDs do not make I/O latency zero.
* Disk scheduling is not only an exam topic; queueing/fairness still matter.
* `df` free space does not diagnose I/O latency.

### What Interviewers Usually Test

HDD-vs-SSD nuance, request ordering, IOPS-vs-throughput, and diagnosing storage-induced application slowness.

### Common Follow-up Questions

1. SSTF drawback?
2. SCAN versus C-SCAN?
3. IOPS versus throughput?
4. Why does SSD still have latency?
5. What does `iostat` help inspect?

### Follow-up Answers

1. Requests far from current head position can starve if closer requests keep arriving.
2. SCAN services in both sweep directions; C-SCAN services one direction and returns, making waits more uniform by position.
3. IOPS counts operations; throughput measures bytes per second. Small random I/O may have high IOPS but modest throughput.
4. Flash translation, controller queues, garbage collection, write amplification, and interface latency still cost time.
5. It shows per-device throughput, utilization, queue/latency indicators, and CPU I/O wait context.
