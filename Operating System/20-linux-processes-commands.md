# Linux Processes and Useful Commands

## What is it?

Linux exposes process, memory, filesystem, and I/O state through commands and `/proc`. These tools turn OS concepts into evidence for debugging production services.

## Simple Explanation

Commands are an observability toolkit: some show who is alive, some show who uses CPU/memory, some show open files, and some show what a process is asking the kernel to do.

## Hinglish Explanation

Linux commands blind guess ko evidence mein convert karte hain. Pehle process/CPU/memory/disk ka symptom isolate karo, phir `ps`, `top`, `free`, `vmstat`, `lsof`, `strace` jaise tools se root cause narrow down karo.

## Why Do We Need It?

Backend outages often appear as “server slow” or “app stuck.” OS tools identify whether the problem is CPU, memory, I/O, process state, descriptors, or a system call.

## How It Works

```bash
ps -ef | grep java          # process, PID, PPID, command
top -H -p <pid>             # per-thread CPU view
free -h                     # memory and swap summary
vmstat 1                    # runnable, blocked, swap, CPU signals over time
iostat -xz 1                # storage latency/throughput indicators
df -h ; du -sh *            # filesystem space versus directory use
lsof -p <pid>               # files/sockets held by process
strace -f -p <pid>          # system calls; use carefully in production
```

## Important Terminology

| Command | Best use | Interview relevance |
|---|---|---|
| `ps`, `pstree` | Process/parent hierarchy | Zombie/worker diagnosis |
| `top`, `htop` | CPU/memory live view | Find hot process/thread |
| `kill`, `jobs`, `fg`, `bg` | Signals and shell jobs | Lifecycle control |
| `nice`, `renice` | Scheduling niceness | Priority trade-off |
| `free`, `vmstat` | RAM/swap/reclaim activity | Memory pressure |
| `df`, `du` | Filesystem vs directory usage | Disk-full root cause |
| `lsof` | Open descriptors/files/sockets | Deleted-open files/leaks |
| `strace` | Syscall activity | Stuck process diagnosis |

## Example

If API CPU is high, run `top -H -p <pid>` to find hot thread, then use profiler/stack dump. If `df` is full but `du` seems low, look for deleted-but-open files with `lsof +L1`.

## Real-World Example

During an incident, capture a short `vmstat 1` and `iostat -xz 1` sample before restart. It preserves evidence about runnable queue, swap, I/O wait, and device saturation.

## Important Properties

| Symptom | First checks |
|---|---|
| CPU high | `top -H`, `ps`, profiler/stack dump |
| Memory high | `free -h`, RSS/PSS, cgroup limit, heap dump |
| Disk full | `df -h`, `du`, `lsof +L1` |
| Process stuck | `strace`, thread dump, sockets/files, logs |

### Common Misconceptions

* `kill` sends a signal; it does not always forcibly terminate. `kill -9` is SIGKILL and cannot be handled/cleaned up by process.
* High “used” Linux memory can include healthy filesystem cache.
* `strace` has overhead; use targeted short captures.

### What Interviewers Usually Test

Whether you choose commands by hypothesis and interpret output rather than listing tools without a debugging order.

### Common Follow-up Questions

1. `kill` versus `kill -9`?
2. `df` versus `du`?
3. What does `lsof` reveal?
4. Why `top -H`?
5. What does `vmstat` show?

### Follow-up Answers

1. `kill` sends SIGTERM by default, allowing cleanup; `kill -9` sends uncatchable SIGKILL and should be last resort.
2. `df` reports filesystem allocated/free blocks; `du` totals files reachable under a directory, so deleted-open files can create a difference.
3. It lists file descriptors including files, directories, sockets, pipes, and deleted-but-open files held by processes.
4. It displays individual threads so a hot thread can be identified inside a multithreaded process.
5. It samples system runnable/blocked tasks, memory/swap, interrupts/context switches, and CPU states over time.
