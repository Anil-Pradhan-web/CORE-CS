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
# Process & Thread Inspection
ps -ef | grep -E "python|java" # Find PID, PPID, and command line
top -H -p <pid>                # Per-thread CPU & memory usage
pidstat -u 1 -p <pid>          # CPU usage breakdown (user vs system/kernel time)
pstree -p <pid>                # Visualize parent-child process tree

# Memory & Swap Diagnostics
free -h                        # Total, used, free RAM and swap memory
vmstat 1 5                     # System-wide runnable/blocked tasks, swap, and context switches
cat /proc/<pid>/status         # VmRSS (Resident Set Size), VmSize, and thread count
cat /proc/<pid>/smaps_rollup   # PSS (Proportional Set Size) for accurate memory tracking

# Storage, Files & Sockets
df -h ; du -sh *               # Filesystem partition capacity vs folder usage
lsof -p <pid>                  # Open file descriptors, sockets, and unlinked files
lsof +L1                       # Find deleted-but-open files still consuming disk space
ss -tulpn                      # Listening TCP/UDP sockets with process PIDs

# Kernel & System Call Tracing
strace -c -p <pid>             # Summary of syscall counts, errors, and time spent in kernel
strace -f -e trace=network,file -p <pid> # Trace specific syscall categories
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
