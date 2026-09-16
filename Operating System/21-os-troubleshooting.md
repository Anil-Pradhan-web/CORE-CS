# Real-World OS Troubleshooting Scenarios

## What is it?

OS troubleshooting is evidence-driven isolation of CPU, memory, process, file, descriptor, and I/O problems. Start with exact symptoms and time window; do not restart before collecting safe evidence.

## Simple Explanation

Treat a slow server like a hospital triage: first identify the vital sign that is abnormal, then test likely causes one layer at a time instead of prescribing random medicine.

## Hinglish Explanation

“Server slow” ek symptom hai, root cause nahi. CPU, memory, disk I/O, locks, network waits ya external dependency koi bhi reason ho sakta hai. Exact time, affected PID, metrics aur command output collect karke hypothesis test karo.

## Why Do We Need It?

Different OS failures look similar to users. A structured approach lowers mean time to repair and prevents a restart from erasing the best debugging evidence.

## How It Works

1. Record symptom, affected host/PID, start time, deployment/load changes, and error.
2. Check CPU/load, RAM/swap/cgroup, disk space/I/O, process/thread state, and open descriptors.
3. Correlate with application metrics, logs, traces, and dependency health.
4. Capture a targeted stack trace, heap profile, or syscall sample when safe.
5. Apply smallest safe mitigation, verify recovery, then document root cause.

## Important Terminology

| Symptom | Inspect | Likely directions |
|---|---|---|
| High CPU | `top -H`, profiles, run queue | Busy loop, hot path, GC, contention |
| High memory | `free`, cgroups, RSS/PSS, heap | Leak, cache, load, wrong limit |
| Unresponsive process | `strace`, thread dump, logs | I/O wait, deadlock, external dependency |
| Disk full | `df`, `du`, `lsof +L1` | Logs, deleted-open file, inode exhaustion |
| Slow I/O | `iostat`, `vmstat`, app timing | Device saturation, sync writes, queueing |

## Example

**High CPU API:** identify PID/thread with `top -H`; capture Java thread dump or profiler; compare request rate and GC; inspect hot stack; rate-limit or roll back only after evidence.

**Disk suddenly full:** run `df -h`, `df -i`, `du -xhd1`, and `lsof +L1`. A deleted open log explains `df` full plus low `du`; restart/rotate the owning process safely to release it.

## Real-World Example

**OOM in container:** verify cgroup memory limit and kernel OOM events, distinguish Java heap from native/direct buffers/page cache, capture heap/metrics, then size limits and fix retention/concurrency.

## Important Properties

| Scenario | Do first | Avoid |
|---|---|---|
| Process hung | Stack/`strace` sample | Immediate SIGKILL without evidence |
| Memory leak | Trend/profile/dump | Assuming GC means no leak |
| Deadlock | Thread/lock dump | Repeated blind retries |
| I/O slow | Per-device metrics | Blaming CPU from latency alone |

### Common Misconceptions

* Load average is not CPU percentage; it includes runnable and often uninterruptible tasks.
* High context switching is a clue, not necessarily a root cause.
* A process in `D` state may not respond even to SIGKILL until the kernel I/O wait returns.

### What Interviewers Usually Test

Your diagnostic order, command choice, interpretation of ambiguous symptoms, and safe mitigation under production constraints.

### Common Follow-up Questions

1. How detect zombie processes?
2. What causes high context switching?
3. Why can `df` and `du` disagree?
4. What is process `D` state?
5. How investigate memory leak?

### Follow-up Answers

1. Use `ps` with state columns or `ps aux | grep Z`; then identify/reap or repair the parent process.
2. Too many threads, small CPU quanta, lock contention, frequent blocking/wakeup, interrupts, or oversubscription can raise switches.
3. `du` ignores deleted pathnames, while `df` counts allocated blocks still held by open file descriptors; inode exhaustion can also matter.
4. It is uninterruptible sleep, commonly waiting in kernel I/O; the task may not handle signals until the wait completes.
5. Measure growth, inspect heap/RSS profiles and retained references/allocation stacks, reproduce under load, then remove unwanted ownership or add bounds.
