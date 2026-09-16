# Stack, Heap, and Memory Leaks

## What is it?

The stack stores call frames, local execution state, and return information per thread. The heap stores dynamically allocated objects with flexible lifetime. A memory leak is memory that remains reachable/allocated longer than intended or is never released.

## Simple Explanation

Stack is a pile of plates: last function call is removed first. Heap is a storeroom where objects can live in any order until explicit free or garbage collection decides they are no longer needed.

## Hinglish Explanation

Har thread ka apna stack hota hai jisme function calls, local variables aur return address hote hain. Heap shared process memory mein dynamic objects ke liye hota hai. Leak tab hai jab memory release nahi hoti ya Java mein unwanted references object ko alive rakhti hain.

## Why Do We Need It?

Functions need fast nested-call storage; applications need dynamic objects whose lifetime is not tied to one function. Correct memory lifecycle prevents crashes, paging, OOM, and latency spikes.

## How It Works

1. Function call pushes a frame/updates stack pointer.
2. Return pops frame automatically.
3. Dynamic allocation asks allocator for heap block/object.
4. C/C++ code frees explicitly; garbage-collected runtimes reclaim unreachable objects later.
5. Excessive recursion exhausts stack; uncontrolled allocation exhausts heap or memory limit.

## Important Terminology

| Term | Meaning |
|---|---|
| Stack frame | Per-call state: locals, return address, saved registers |
| Heap | Dynamic allocation region/abstraction |
| RSS | Resident physical memory used by process |
| GC | Garbage collector reclaims unreachable managed objects |
| Memory leak | Unreleased or unintentionally retained memory |
| Stack overflow | Stack limit exceeded, often deep/unbounded recursion |

## Example

Java objects are normally allocated on the managed heap. A static `Map` that grows forever retains references, so GC cannot free those objects: a managed-memory leak.

## Real-World Example

An API service leaks request buffers or retains cache entries. Its heap/RSS grows, GC frequency increases, latency rises, then a container limit or OS OOM condition can terminate it.

## Important Properties

| Property | Stack | Heap |
|---|---|---|
| Lifetime | Function/thread scope | Explicit or reachability-based |
| Allocation | Very fast pointer movement | Allocator/GC work |
| Sharing | Per thread | Process-wide/shared by threads |
| Typical failure | Stack overflow | Leak/OOM/fragmentation |

### Common Misconceptions

* Java can have memory leaks despite garbage collection.
* Heap does not mean physical RAM only; virtual mappings/backing are involved.
* Each thread has a separate stack, but threads usually share heap.

### What Interviewers Usually Test

Lifetime rules, recursion, GC/leaks, memory symptoms, and language-specific nuance without confusing heap with a single physical location.

### Common Follow-up Questions

1. Can Java leak memory?
2. Stack overflow versus heap OOM?
3. Why is stack allocation fast?
4. What is a dangling pointer?
5. How find a leak?

### Follow-up Answers

1. Yes. Objects retained by unintended references remain reachable and cannot be collected.
2. Stack overflow exceeds per-thread call-stack limit; heap OOM occurs when dynamic allocation cannot obtain enough allowed memory.
3. It usually only moves a stack pointer and releases memory automatically on return, with simple LIFO lifetime.
4. In manual-memory languages, it is a pointer that still refers to memory already freed, causing undefined/unsafe behaviour.
5. Compare heap/RSS growth over time, take heap profiles/dumps, inspect allocation paths and retained references, and reproduce under controlled load.
