# CPU Scheduling Algorithms

## What is it?

CPU scheduling decides which ready thread runs next. The goal is a practical balance of responsiveness, fairness, throughput, predictability, and overhead—not one universally best algorithm.

## Simple Explanation

It is a queue manager at a clinic. Some policies serve arrival order, some prioritize short work, and Round Robin gives everyone a small turn so no interactive task waits forever.

## Hinglish Explanation

CPU scheduler ready queue se next thread choose karta hai. Batch job ko throughput chahiye, interactive app ko fast response. Isliye OS policy workload ke according fairness aur latency balance karti hai.

## Why Do We Need It?

More runnable work exists than CPU cores. Without scheduling, one process can monopolize CPU and interactive applications become unresponsive.

## How It Works

1. Events put threads into the ready queue: creation, I/O completion, or preemption.
2. Scheduler selects a candidate according to policy/priority.
3. Dispatcher context-switches to that thread.
4. Thread runs until it blocks, exits, yields, or its time quantum ends.

```text
Ready queue → scheduler choice → dispatch → Running
                 ↑                    │
      I/O complete / preemption / yield┘
```

## Important Terminology

| Term | Meaning |
|---|---|
| Waiting time | Time spent ready but not executing |
| Turnaround time | Completion time − arrival time |
| Response time | First run/start response − arrival time |
| Quantum | Maximum Round Robin time slice |
| Preemptive | OS can interrupt a running task |
| Starvation | A task waits indefinitely due to policy |

## Example

For processes `P1=5 ms`, `P2=2 ms`, `P3=1 ms` arriving together, non-preemptive SJF runs `P3 → P2 → P1`, minimizing average waiting time if burst lengths are known.

```text
Round Robin, quantum = 2:
| P1 | P2 | P3 | P1 | P1 |
0    2    4    5    7    8
```

## Real-World Example

Linux uses a sophisticated fair scheduler design with priority classes, not a textbook single queue. Web services still feel the effects: CPU saturation raises queue time and tail latency.

## Important Properties

| Algorithm | Strength | Weakness |
|---|---|---|
| FCFS | Simple | Convoy effect, poor interactive response |
| SJF/SRTF | Low average wait if estimates are good | Starvation, burst unknown |
| Round Robin | Responsive/fair sharing | Quantum-switch overhead |
| Priority | Serves important work | Starvation without aging |
| MLFQ | Adapts to interactive/batch mix | Complex tuning |

### Common Misconceptions

* SJF requires an estimate of future burst time; real systems cannot know it perfectly.
* A tiny Round Robin quantum is not always better because switching costs matter.
* High CPU use does not prove the scheduler is wrong.

### What Interviewers Usually Test

Definitions of timing metrics, preemption, Gantt-chart reasoning, and policy trade-offs under different workloads.

### Common Follow-up Questions

1. Waiting versus turnaround versus response time?
2. What is convoy effect?
3. Why use aging?
4. SJF versus SRTF?
5. Why quantum size matters?

### Follow-up Answers

1. Waiting is ready-queue delay; turnaround is total arrival-to-finish time; response is arrival-to-first execution/response time.
2. A long FCFS job makes many short jobs wait behind it, like a convoy, reducing responsiveness.
3. Aging gradually raises a waiting task’s priority so low-priority tasks do not starve forever.
4. SJF is non-preemptive shortest job first; SRTF preempts if a newly ready job has less remaining time.
5. Too large behaves like FCFS; too small increases context-switch overhead. The right value balances response and efficiency.
