# Operating System Basics and Architecture

## What is it?

An operating system (OS) is the trusted software layer that manages hardware and provides safe, convenient services to programs. Linux, Windows, macOS, Android, and iOS are OS families.

## Simple Explanation

An OS is like the manager of a shared office: it decides who gets CPU time, memory, files, devices, and permissions so one employee cannot casually break everyone else’s work.

## Hinglish Explanation

OS ko computer ka manager samjho. Apps directly CPU, RAM, disk, ya device ko control nahi karti; OS rules ke according resources allocate karta hai aur isolation maintain karta hai.

## Why Do We Need It?

Without an OS, every program would need to control hardware itself, compete unsafely for resources, and implement drivers/filesystems repeatedly. The OS provides abstraction, sharing, protection, and recovery boundaries.

## How It Works

1. Hardware starts a small boot program that loads the kernel.
2. Kernel initializes memory, CPU scheduling, device drivers, and filesystems.
3. Programs run in user mode and request privileged work through system calls.
4. Kernel schedules runnable threads and handles interrupts from hardware.

```text
Applications
    │ system calls
    ▼
Kernel: processes • memory • files • devices • security
    ▼
CPU • RAM • disk • network • peripherals
```

## Important Terminology

| Term | Meaning |
|---|---|
| Kernel | Privileged core of the OS |
| Process | Running program plus its OS-managed state |
| Thread | Execution path inside a process |
| Driver | Kernel component that controls a device |
| Scheduler | Chooses a runnable thread for CPU time |
| Filesystem | Names, stores, and retrieves persistent files |

## Example

When a Python API opens a log file, the OS checks permissions, maps the request to a filesystem and storage driver, then returns a file descriptor to Python.

## Real-World Example

Containers share a host kernel but use namespaces and cgroups to isolate process views and limit resources. A VM instead generally runs its own guest kernel on virtual hardware.

## Important Properties

| OS responsibility | Benefit | Cost/trade-off |
|---|---|---|
| Abstraction | Easier programming | Some overhead |
| Multiplexing | Many programs share hardware | Scheduling/coordination |
| Isolation | Fault/security containment | Extra checks/resources |
| Persistence | Files survive process exit | I/O latency/failure handling |

### Common Misconceptions

* An OS is not only a graphical desktop; servers run OS kernels too.
* The kernel is not the same as every installed program.
* A process is more than an executable file.

### What Interviewers Usually Test

Whether you can explain why an OS exists, its core responsibilities, and how it protects one workload from another.

### Common Follow-up Questions

1. Kernel versus operating system?
2. VM versus container?
3. What resources does an OS manage?
4. Why cannot apps access hardware freely?
5. What is a device driver?

### Follow-up Answers

1. The kernel is the privileged core; the operating system also includes user-space tools, libraries, and services around it.
2. A VM virtualizes hardware and runs a guest OS; containers share the host kernel while isolating processes.
3. CPU time, memory, storage, devices, network access, processes, files, and permissions.
4. Unrestricted access could corrupt memory, bypass security, or let one app monopolize devices.
5. A driver translates generic OS operations into commands for a particular hardware device.
