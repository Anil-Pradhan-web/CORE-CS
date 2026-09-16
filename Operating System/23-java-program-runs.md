# What Happens When a Java Program Runs?

## What is it?

A Java source file compiles to bytecode. The JVM is an OS process that loads classes, manages a managed heap and Java threads, interprets/JIT-compiles code, and uses native system calls for files, network, timers, and memory.

## Simple Explanation

Java code runs inside a powerful workshop—the JVM. The JVM is still just a process from the OS point of view, so OS scheduling, memory limits, files, sockets, and signals still control its real-world behaviour.

## Hinglish Explanation

`javac` bytecode banata hai; `java` JVM process start karta hai. JVM class load karta hai, Java heap/threads manage karta hai aur JIT native machine code bana sakta hai. But OS scheduler hi actual native threads ko CPU deta hai aur cgroup/container limits apply hoti hain.

## Why Do We Need It?

Backend interviewers expect you to relate JVM metrics to OS reality: heap is not all process memory, Java threads consume native stacks, GC affects CPU/latency, and sockets/files still use OS descriptors.

## How It Works

1. `javac` compiles `.java` to `.class` bytecode; packaging may create JAR.
2. `java` starts JVM process; OS loader maps JVM/native libraries.
3. JVM class loaders load classes; verifier checks bytecode; runtime allocates heap/metaspace/thread stacks.
4. JVM creates native OS threads for Java threads in common JVM implementations.
5. Interpreter runs bytecode; JIT compiles hot methods to native code.
6. GC reclaims unreachable heap objects; OS manages pages, scheduling, and memory limit enforcement.
7. Java I/O/NIO ultimately uses syscalls and file descriptors/sockets.

## Important Terminology

| Term | Meaning |
|---|---|
| JVM | Java Virtual Machine process/runtime |
| Bytecode | Platform-independent instruction format |
| JIT | Just-in-time compiler to native code |
| Java heap | Managed object memory, not entire RSS |
| Metaspace | Native-memory area for class metadata in modern HotSpot |
| Native stack | Per-thread stack allocated/managed at OS level |

## Example

A Java service with `-Xmx1g` can still exceed 1 GiB RSS because of thread stacks, metaspace, direct buffers, code cache, native libraries, and allocator overhead. Container memory limit must account for all of them.

## Real-World Example

When Java latency spikes, compare GC logs/metrics, CPU, runnable threads, blocked threads, allocation rate, RSS/cgroup limit, and external I/O. A heap dump alone may miss native-memory leak or too many threads.

## Important Properties

| Layer | JVM responsibility | OS responsibility |
|---|---|---|
| Threads | Runtime mapping/coordination | Schedule native threads |
| Heap | Object allocation/GC | Virtual pages/RAM limit |
| I/O | Java API/buffers | Syscalls, sockets, descriptors |
| CPU | JIT/interpreter work | Core time/preemption |

### Common Misconceptions

* `-Xmx` is not a cap on all JVM process memory.
* Garbage collection does not prevent leaks caused by reachable objects.
* Java threads are not “free”; each uses native/OS resources and stack memory.

### What Interviewers Usually Test

The JVM–OS boundary, memory accounting, thread scheduling, GC effects, and a practical approach to Java production incidents.

### Common Follow-up Questions

1. Heap versus RSS?
2. Why can Java OOM inside container below host RAM?
3. Does JVM schedule CPU?
4. What is JIT?
5. Where do Java sockets go?

### Follow-up Answers

1. Heap is managed object-memory allocation; RSS is physical RAM currently resident for all process mappings, including native memory and stacks.
2. The container has its own cgroup memory limit, and JVM plus native memory can exceed that limit even if host has free RAM.
3. JVM manages runtime work but the OS scheduler ultimately assigns native JVM threads to CPU cores.
4. JIT compiles frequently executed bytecode into optimized native machine code during program execution.
5. Java socket APIs call into native runtime/kernel networking and use OS socket file descriptors underneath.
