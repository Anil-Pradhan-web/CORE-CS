# Inter-Process Communication (IPC)

## What is it?

IPC lets isolated processes exchange data and coordinate. Common choices are pipes, named pipes, shared memory, message queues, signals, and sockets.

## Simple Explanation

Processes live in separate apartments, so IPC provides options: a tube for bytes, a shared noticeboard, a mailbox, a doorbell, or a network-style conversation.

## Hinglish Explanation

Alag processes apni memory directly share nahi karte. IPC ke through data exchange hota hai: pipe stream deta hai, shared memory fast common region deti hai, queue messages deti hai, signal notification deta hai, socket flexible communication deta hai.

## Why Do We Need It?

Process isolation improves safety but services still need to cooperate: shell pipelines, database clients, local daemons, parent-child supervision, and microservice networking all need communication.

## How It Works

### The IPC Spectrum

```text
┌─────────────────┬──────────────────────────────────┬────────────────────────┐
│  IPC Mechanism  │         Kernel Overhead          │       Best Used For    │
├─────────────────┼──────────────────────────────────┼────────────────────────┤
│ Pipes           │ Medium (double copy to/from buf) │ Parent-child streams   │
│ Shared Memory   │ Lowest (zero copy after mapping) │ High-speed local bulk  │
│ Unix Sockets    │ Low (kernel buffer, no TCP stack)│ Robust local services  │
│ Network Sockets │ Highest (full TCP/IP stack)      │ Distributed machines   │
└─────────────────┴──────────────────────────────────┴────────────────────────┘
```

### 1. Anonymous Pipes (Parent-Child Communication)

A unidirectional byte channel created in the kernel:

```python
import os

# Create kernel pipe: r (read end), w (write end)
r_fd, w_fd = os.pipe()

pid = os.fork()
if pid == 0:
    # Child process: closes unused write end, reads message
    os.close(w_fd)
    msg = os.read(r_fd, 1024)
    print(f"Child received: {msg.decode()}")
    os.close(r_fd)
else:
    # Parent process: closes unused read end, writes message
    os.close(r_fd)
    os.write(w_fd, b"Hello from parent process!")
    os.close(w_fd)
```

---

### 2. Shared Memory (`mmap` / POSIX `shm_open`)

The fastest IPC because both processes map the same physical RAM pages into their virtual address spaces:

```python
import mmap
import os

# Create an anonymous shared memory map backed by RAM
shm = mmap.mmap(-1, 4096, flags=mmap.MAP_SHARED | mmap.MAP_ANONYMOUS)

pid = os.fork()
if pid == 0:
    # Child writes to shared memory
    shm.seek(0)
    shm.write(b"Fast shared memory message")
else:
    os.wait() # Wait for child
    shm.seek(0)
    print(f"Parent read: {shm.read(26).decode()}")
```

---

### 3. Unix Domain Sockets (`AF_UNIX`) vs. Network Sockets (`AF_INET`)

Unix Domain Sockets use the standard Socket API but bypass the entire TCP/IP network stack (no checksumming, packet sequencing, or NIC processing), providing 2x higher throughput than `127.0.0.1`:

```python
import socket

# Unix Domain Socket (Local IPC via filesystem path)
server = socket.socket(socket.AF_UNIX, socket.SOCK_STREAM)
server.bind("/tmp/app_service.sock")
server.listen(10)
```

## Important Terminology

| Mechanism | Data model | Main trade-off |
|---|---|---|
| Pipe | Byte stream, related processes | Usually one direction |
| Named pipe | Filesystem-named byte stream | Local machine only |
| Shared memory | Shared pages | Requires synchronization |
| Message queue | Discrete messages | Kernel/broker limits |
| Signal | Small async notification | Tiny payload/complex handlers |
| Socket | Stream/datagram protocol | Copies/protocol overhead |

## Example

`ps aux | grep java` connects `ps` output to `grep` input using a pipe. A Unix-domain socket lets local processes communicate with network-like APIs but without IP networking.

## Real-World Example

PostgreSQL clients use sockets. A high-throughput local service may use shared memory plus ring buffers, but then must carefully manage visibility, lifetimes, and locks.

## Important Properties

| Need | Useful IPC choice |
|---|---|
| Shell stream | Pipe |
| Fast local bulk exchange | Shared memory |
| Event notification | Signal/eventfd/message queue |
| Local or remote request/response | Socket |

### Common Misconceptions

* Shared memory does not provide synchronization automatically.
* A pipe is a byte stream, not inherent messages.
* Signals are not safe places for arbitrary complex work.

### What Interviewers Usually Test

Ability to select IPC by scope, data shape, performance, synchronization, and failure behaviour.

### Common Follow-up Questions

1. Pipe versus named pipe?
2. Why is shared memory fast?
3. Does shared memory need a lock?
4. Unix-domain versus TCP socket?
5. What is backpressure in IPC?

### Follow-up Answers

1. A pipe is commonly created between related processes; a named pipe has a filesystem name so unrelated local processes can open it.
2. Processes map the same physical pages, avoiding copying payload through a separate kernel buffer for each transfer.
3. Usually yes; shared state needs synchronization or an ownership/lock-free protocol to avoid races.
4. Unix-domain sockets are local IPC with filesystem/credential features; TCP sockets work across networks and use IP/TCP.
5. It is the mechanism that slows senders when receiver buffers or consumers cannot keep up.
