# 100 Most Frequently Asked Operating Systems Interview Questions

Each answer follows a speakable pattern: definition, why, how, example, and caveat.

## Beginner — 1–30

### 1. What is an operating system?

**Short Answer:** An OS manages hardware resources and provides protected services to applications. It schedules CPU, manages memory, files, devices, and permissions.

**Detailed Explanation:** It lets many programs safely share one machine instead of each program controlling hardware directly.

**Hinglish:** OS computer ka manager hai—apps ko CPU, RAM, disk aur devices controlled way mein deta hai.

> **Interview Tip:** Name resource management and protection.

**Follow-up & Answer:** **Kernel?** The privileged core of the OS. **Why needed?** To share and protect finite hardware resources.

### 2. Kernel versus operating system?

**Short Answer:** The kernel is the privileged core; the OS also includes user-space libraries, tools, services, and often a desktop environment.

**Detailed Explanation:** Kernel implements low-level scheduling, memory, drivers, and syscalls, while the complete OS distribution supplies usable programs around it.

**Hinglish:** Kernel OS ka core hai; poora OS kernel plus user-space tools/services hota hai.

> **Interview Tip:** Do not use the two words as exact synonyms.

**Follow-up & Answer:** **What runs in kernel?** Privileged resource-management code. **Can a server have no GUI?** Yes, it still runs an OS/kernel.

### 3. What is a program versus a process?

**Short Answer:** A program is passive executable code; a process is a running instance with memory, execution state, PID, open resources, and scheduling information.

**Detailed Explanation:** Multiple processes can run the same program independently with separate address spaces.

**Hinglish:** Program file/blueprint hai; process us blueprint ka live running instance hai.

> **Interview Tip:** Mention address space and runtime state.

**Follow-up & Answer:** **Can one program have many processes?** Yes. **Does a process have a PID?** Yes, the OS assigns an identifier.

### 4. What is a process control block (PCB)?

**Short Answer:** PCB is the kernel’s record for a process. It stores identity, state, registers, scheduling information, memory mappings, and resource references.

**Detailed Explanation:** The OS needs it to pause, resume, manage, and account for a process.

**Hinglish:** PCB process ka kernel-side record card hai jisme state, registers, PID, memory aur open files ki info hoti hai.

> **Interview Tip:** Link PCB to context switching.

**Follow-up & Answer:** **Why save registers?** So execution can resume correctly. **PCB versus TCB?** TCB stores thread-specific execution state.

### 5. Name common process states.

**Short Answer:** New, Ready, Running, Waiting/Blocked, and Terminated are common states. A zombie is an exited process whose parent has not collected status.

**Detailed Explanation:** Ready waits for CPU; blocked waits for an event, so they are not the same state.

**Hinglish:** Ready thread CPU ke liye wait karta hai; blocked thread I/O, lock ya event ke liye wait karta hai.

> **Interview Tip:** Clearly distinguish ready from blocked.

**Follow-up & Answer:** **What wakes blocked process?** Its awaited event. **Who chooses ready process?** Scheduler.

### 6. What is `fork()`?

**Short Answer:** `fork()` creates a child process from a parent. Child begins with a similar process image and usually shares pages using copy-on-write until one side writes.

**Detailed Explanation:** Both return from `fork`, receiving different return values so they can choose different paths.

**Hinglish:** `fork()` parent ka child process banata hai; memory initially copy-on-write se efficient hoti hai.

> **Interview Tip:** `fork` creates a process; `exec` replaces code.

**Follow-up & Answer:** **Why COW?** It avoids copying unused pages. **Does child get new PID?** Yes.

### 7. What is `exec()`?

**Short Answer:** `exec` replaces the current process’s program image with a new executable while keeping the same PID. It does not create a new process by itself.

**Detailed Explanation:** Shells commonly fork a child, then child execs the requested command.

**Hinglish:** `exec` current process ke andar naya program load karta hai; PID same rehta hai.

> **Interview Tip:** Say “replace,” not “create.”

**Follow-up & Answer:** **What happens to old code?** It is unmapped/replaced. **Why fork then exec?** Parent can continue while child runs command.

### 8. What is a zombie process?

**Short Answer:** A zombie has exited but retains a small process-table entry containing exit status until its parent calls `wait()`. It is not actively running.

**Detailed Explanation:** Many zombies can exhaust PID/process-table resources if parent never reaps them.

**Hinglish:** Zombie ka kaam khatam hai but parent ne exit status collect nahi kiya, isliye entry bachi hai.

> **Interview Tip:** Do not call a zombie a memory/CPU-heavy process.

**Follow-up & Answer:** **Fix?** Parent calls `wait`; repair/restart parent if needed. **Orphan?** A child whose parent exited, usually adopted by init/systemd.

### 9. What is a thread?

**Short Answer:** A thread is a schedulable execution path within a process. Threads share process code, heap, and open files but have individual registers, stack, and program counter.

**Detailed Explanation:** Threads support responsive/concurrent work with less isolation than processes.

**Hinglish:** Thread process ke andar execution unit hai; heap share hota hai but stack aur execution state private hota hai.

> **Interview Tip:** Mention own stack.

**Follow-up & Answer:** **Why threads?** Concurrency and shared data. **Can threads race?** Yes, because they share mutable memory.

### 10. Process versus thread?

**Short Answer:** Processes usually have separate address spaces and stronger isolation; threads share one process address space and communicate cheaply but require synchronization.

**Detailed Explanation:** Process creation/switch is generally costlier, while thread failures can corrupt sibling-thread state.

**Hinglish:** Process isolation strong deta hai; threads lightweight sharing dete hain but race risk badhta hai.

> **Interview Tip:** Compare memory, isolation, cost, and communication.

**Follow-up & Answer:** **Separate stacks?** Each thread has one. **IPC needed?** Usually between processes, not sibling threads.

### 11. What is context switching?

**Short Answer:** Context switching saves one running thread’s CPU state and restores another’s so CPU can run a different task. It happens on preemption, blocking, interrupts, or yield.

**Detailed Explanation:** Registers, program counter, stack pointer, and scheduling state are saved; caches/TLB effects add overhead.

**Hinglish:** Context switch mein current thread ki position save hoti hai aur next thread ki restore hoti hai.

> **Interview Tip:** Switching is useful but not free.

**Follow-up & Answer:** **Why expensive?** State saving, scheduler work, cache/TLB disruption. **Process switch extra cost?** It may change address-space mappings.

### 12. What is concurrency versus parallelism?

**Short Answer:** Concurrency means tasks make overlapping progress; parallelism means tasks execute simultaneously on multiple cores. Concurrency can exist on one core through time slicing.

**Detailed Explanation:** Async I/O provides concurrency without necessarily using one thread per request.

**Hinglish:** Concurrency mein tasks alternate/progress karte hain; parallelism mein multiple cores par same time run karte hain.

> **Interview Tip:** One CPU core can be concurrent, not truly parallel.

**Follow-up & Answer:** **Why concurrency?** Responsiveness/I/O overlap. **Why parallelism?** More CPU-bound throughput.

### 13. What is CPU scheduling?

**Short Answer:** CPU scheduling selects which ready thread should run next. It balances response time, fairness, throughput, waiting time, and context-switch overhead.

**Detailed Explanation:** A scheduler runs when a thread becomes ready, blocks, exits, or a time slice ends.

**Hinglish:** Scheduler ready queue se next runnable thread choose karta hai according to OS policy.

> **Interview Tip:** Schedulers normally schedule threads, not abstract programs.

**Follow-up & Answer:** **Ready queue?** Runnable tasks waiting for CPU. **Dispatcher?** Component that performs the switch to selected task.

### 14. Define waiting, turnaround, and response time.

**Short Answer:** Waiting time is ready-queue delay; turnaround is arrival to completion; response time is arrival to first execution/response. They measure different user/system experiences.

**Detailed Explanation:** Interactive systems care strongly about response/tail latency, while batch jobs may emphasize turnaround and throughput.

**Hinglish:** Waiting CPU queue ka wait hai, turnaround total completion time, response first response tak ka delay hai.

> **Interview Tip:** Do not interchange these metrics.

**Follow-up & Answer:** **Formula turnaround?** Finish minus arrival. **Can response be low while turnaround high?** Yes, a job can start fast but finish late.

### 15. What is FCFS scheduling?

**Short Answer:** First-Come, First-Served runs ready work in arrival order. It is simple and fair by arrival but can cause poor response when a long job blocks short jobs.

**Detailed Explanation:** It is usually non-preemptive in textbook form and can suffer convoy effect.

**Hinglish:** FCFS mein jo pehle aaya woh pehle run karega; simple hai but long job ke peeche short jobs wait karte hain.

> **Interview Tip:** Mention convoy effect.

**Follow-up & Answer:** **Convoy effect?** Long job delays many short ones. **Preemptive?** Classic FCFS is not.

### 16. What is SJF and SRTF?

**Short Answer:** SJF chooses shortest predicted CPU burst non-preemptively; SRTF is its preemptive form that runs shortest remaining time. They minimize average wait under ideal burst knowledge.

**Detailed Explanation:** Real systems estimate bursts and must prevent long jobs from starving.

**Hinglish:** SJF shortest burst choose karta hai; SRTF mein shorter new job aaye toh current job preempt ho sakta hai.

> **Interview Tip:** Future burst time is estimated, not known perfectly.

**Follow-up & Answer:** **Drawback?** Starvation. **Why low average wait?** Short jobs leave queue sooner.

### 17. What is Round Robin scheduling?

**Short Answer:** Round Robin gives each ready task a fixed time quantum then preempts and cycles it to queue end. It improves interactive fairness and response.

**Detailed Explanation:** Very small quantum raises switch overhead; very large quantum behaves like FCFS.

**Hinglish:** Round Robin har process ko small time slice deta hai, phir next ready process ko chance milta hai.

> **Interview Tip:** Quantum size is the key trade-off.

**Follow-up & Answer:** **Small quantum?** Better response but high overhead. **Large quantum?** Lower overhead but poorer responsiveness.

### 18. What is priority scheduling and aging?

**Short Answer:** Priority scheduling runs higher-priority work first. Aging gradually boosts waiting tasks so low-priority tasks do not starve indefinitely.

**Detailed Explanation:** Static priorities can cause starvation; priority inheritance can address a different issue, priority inversion.

**Hinglish:** Priority scheduling important task ko pehle run karta hai; aging wait karte task ki priority increase karta hai.

> **Interview Tip:** Aging solves starvation, not every priority problem.

**Follow-up & Answer:** **Starvation?** Indefinite wait. **Priority inversion?** High priority waits on low priority holder while medium work runs.

### 19. What is a race condition?

**Short Answer:** A race condition is a correctness failure where result depends on timing/interleaving of concurrent operations on shared state. Unsynchronized read-modify-write is a common cause.

**Detailed Explanation:** A counter increment can lose updates when two threads read same old value and overwrite each other.

**Hinglish:** Race mein result execution order par depend karta hai; same shared data par unsynchronized update unsafe hota hai.

> **Interview Tip:** Give a concrete interleaving.

**Follow-up & Answer:** **Fix?** Mutex, atomic operation, transaction, or ownership/message passing. **Is volatile enough?** Usually no for `x++`.

### 20. What is a critical section?

**Short Answer:** A critical section accesses a shared resource and must not overlap with conflicting access. Correct solutions aim for mutual exclusion, progress, and bounded waiting.

**Detailed Explanation:** The protected invariant—not merely a random code block—defines what needs synchronization.

**Hinglish:** Critical section woh code hai jahan shared resource modify/read ho raha ho aur conflict avoid karna ho.

> **Interview Tip:** State the shared invariant before choosing a lock.

**Follow-up & Answer:** **Mutual exclusion?** One conflicting actor at a time. **Bounded waiting?** A task cannot be postponed forever.

### 21. What is a mutex?

**Short Answer:** A mutex is an ownership-based lock allowing one thread at a time into a critical section. The owner should unlock it after protecting the shared invariant.

**Detailed Explanation:** Contention causes waiting/spinning; wrong ordering or forgotten unlock can create liveness bugs.

**Hinglish:** Mutex ek owner-based lock hai—ek time par ek thread critical section mein jaata hai.

> **Interview Tip:** Mutex is for exclusion, not counting resources.

**Follow-up & Answer:** **Reentrant mutex?** Owner can lock again with count. **Contention?** Multiple threads compete for same mutex.

### 22. What is a semaphore?

**Short Answer:** A semaphore is a nonnegative counter changed atomically by wait/acquire and signal/release operations. Binary semaphores can model a signal; counting semaphores model available permits.

**Detailed Explanation:** Semaphores coordinate events/resources but do not inherently enforce mutex ownership semantics.

**Hinglish:** Semaphore count maintain karta hai—resource permits ya event signal ke liye useful hai.

> **Interview Tip:** Compare ownership with mutex.

**Follow-up & Answer:** **Counting example?** DB connection pool permits. **Can non-owner signal?** Semaphore semantics can allow it, unlike typical mutex unlock.

### 23. Mutex versus semaphore?

**Short Answer:** A mutex gives exclusive ownership to one thread; a semaphore tracks permits and coordinates resources/events. A binary semaphore is not automatically an owner-enforced mutex.

**Detailed Explanation:** Use a mutex for shared-state exclusion and a counting semaphore for bounded resource availability.

**Hinglish:** Mutex ownership ke saath lock hai; semaphore count/permits ke liye synchronization primitive hai.

> **Interview Tip:** Avoid saying they are identical.

**Follow-up & Answer:** **Use for pool?** Counting semaphore. **Use for map update?** Mutex or appropriate concurrent map/atomic primitive.

### 24. What is deadlock?

**Short Answer:** Deadlock is permanent waiting where each participant holds a resource and waits for another in a cycle. None of the cycle participants can proceed.

**Detailed Explanation:** It needs mutual exclusion, hold-and-wait, no preemption, and circular wait together.

**Hinglish:** Deadlock mein threads ek dusre ke resources ka wait karte hain aur cycle ban jaati hai.

> **Interview Tip:** List all four Coffman conditions.

**Follow-up & Answer:** **Fix?** Lock ordering, timeout/try-lock, detection/abort, or breaking resource conditions. **CPU high?** Not required.

### 25. Deadlock versus starvation versus livelock?

**Short Answer:** Deadlock means cyclic no progress; starvation means one task waits while others progress; livelock means actors keep changing state but no useful work completes.

**Detailed Explanation:** They are liveness problems with different symptoms and fixes.

**Hinglish:** Deadlock stuck hai, starvation mein ek task ignore hota hai, livelock mein sab active hain but result nahi aata.

> **Interview Tip:** Use one short real-world analogy for each.

**Follow-up & Answer:** **Aging helps?** Starvation. **Backoff helps?** Livelock. **Lock order helps?** Deadlock.

### 26. What is IPC?

**Short Answer:** Inter-Process Communication lets separate processes exchange data and coordinate through pipes, shared memory, message queues, signals, or sockets.

**Detailed Explanation:** Choice depends on local/remote scope, data model, performance, synchronization, and failure handling.

**Hinglish:** IPC alag processes ko communicate karne deta hai—pipe, shared memory, queue, signal, socket common options hain.

> **Interview Tip:** Shared memory still needs synchronization.

**Follow-up & Answer:** **Fastest local bulk option?** Shared memory often, with careful synchronization. **Remote option?** Sockets.

### 27. What is a pipe?

**Short Answer:** A pipe is a kernel-buffered byte stream, commonly connecting related processes. A named pipe has a filesystem name so unrelated local processes can open it.

**Detailed Explanation:** Pipes preserve byte order but not application message boundaries by themselves.

**Hinglish:** Pipe ek byte stream channel hai, jaise shell mein `cmd1 | cmd2`.

> **Interview Tip:** Pipe is stream, not guaranteed message queue.

**Follow-up & Answer:** **Named pipe?** FIFO file endpoint. **What on no reader?** Writes may block/fail depending on endpoint state and flags.

### 28. What is shared memory IPC?

**Short Answer:** Shared memory maps the same physical pages into multiple processes, enabling fast data sharing. It requires explicit synchronization and lifetime/permission management.

**Detailed Explanation:** It avoids repeated payload copying but makes races and ownership bugs easier.

**Hinglish:** Shared memory same pages multiple processes ko map karti hai; fast hai but lock/atomic protocol zaroor chahiye.

> **Interview Tip:** Performance does not remove coordination requirements.

**Follow-up & Answer:** **Why fast?** Direct shared pages. **How protect?** Mutex/semaphore/atomic protocol plus permissions.

### 29. What is virtual memory?

**Short Answer:** Virtual memory is a protected per-process address-space abstraction mapped by hardware and OS to physical memory and backing storage. It is not simply extra RAM.

**Detailed Explanation:** It supports isolation, sparse mappings, demand paging, shared libraries, and controlled overcommit.

**Hinglish:** Virtual memory process ko private logical address space deta hai; MMU usko RAM pages se map karta hai.

> **Interview Tip:** Separate abstraction from swap.

**Follow-up & Answer:** **Why useful?** Isolation/flexible allocation. **Does it always use disk?** No, many mappings stay entirely in RAM.

### 30. What is paging?

**Short Answer:** Paging divides virtual memory into fixed-size pages and physical RAM into equal-sized frames. Page tables map virtual pages to physical frames with permissions.

**Detailed Explanation:** Paging removes external physical fragmentation but can have internal waste in a partially used last page.

**Hinglish:** Paging mein page number ko frame number se map kiya jaata hai; offset same rehta hai.

> **Interview Tip:** Page is virtual; frame is physical.

**Follow-up & Answer:** **Why fixed size?** Any free frame can hold any page. **Offset changes?** No, page/frame sizes match.

## Intermediate — 31–70

### 31. What is a page table?

**Short Answer:** A page table maps virtual page numbers to physical frames and stores metadata such as present, permission, dirty, and accessed bits.

**Detailed Explanation:** Each process normally has address-space-specific mappings; multi-level tables reduce memory for sparse virtual ranges.

**Hinglish:** Page table virtual page ko physical frame aur permissions se map karta hai.

> **Interview Tip:** Mention PTE metadata, not only frame number.

**Follow-up & Answer:** **Present bit?** Says page mapping is resident/valid. **Why multi-level?** Avoid giant flat table for unused addresses.

### 32. What is a TLB?

**Short Answer:** A TLB is a small fast cache of recent virtual-to-physical page translations. It accelerates almost every memory access.

**Detailed Explanation:** TLB miss triggers page-table lookup, whereas a page fault occurs only if needed page is absent/invalid.

**Hinglish:** TLB page-table translation ka fast cache hai, normal data cache nahi.

> **Interview Tip:** TLB miss and page fault are different.

**Follow-up & Answer:** **TLB miss?** Translation not cached. **Context switch effect?** Entries may be flushed/tagged depending on hardware and address-space IDs.

### 33. What is a page fault?

**Short Answer:** A page fault is a trap when a memory access finds page absent or disallowed. Kernel checks legality; it loads a valid absent page or signals an invalid access.

**Detailed Explanation:** Valid demand-paging faults are normal but major faults waiting on disk are expensive.

**Hinglish:** Page fault par kernel dekhta hai access legal hai ya nahi; legal missing page ko RAM mein laata hai.

> **Interview Tip:** Not every fault is a crash.

**Follow-up & Answer:** **Major fault?** Needs storage I/O. **Invalid fault?** Can cause segmentation fault/access violation.

### 34. What is demand paging?

**Short Answer:** Demand paging loads a page only when it is first referenced instead of loading all pages at process start. It reduces startup I/O and RAM use.

**Detailed Explanation:** It relies on locality; too many misses under memory pressure create high latency.

**Hinglish:** Demand paging page ko use hone par hi RAM mein load karta hai.

> **Interview Tip:** Link to lazy loading and page faults.

**Follow-up & Answer:** **Benefit?** Faster start/lower RAM. **Risk?** First access may cause slow fault.

### 35. What is thrashing?

**Short Answer:** Thrashing is excessive page replacement and faults because active working sets do not fit in RAM. CPU spends much time waiting on storage rather than executing useful work.

**Detailed Explanation:** Reduce load, add memory, right-size containers, limit concurrency, or fix memory leaks/working-set behaviour.

**Hinglish:** Thrashing mein system mostly pages disk se laane-lejaane mein busy hota hai, useful execution kam hoti hai.

> **Interview Tip:** More processes can worsen it.

**Follow-up & Answer:** **Working set?** Recently/frequently needed pages. **Symptom?** High major faults/swap I/O and poor throughput.

### 36. Compare FIFO, LRU, and Optimal page replacement.

**Short Answer:** FIFO evicts oldest loaded page, LRU evicts least recently used page, and Optimal evicts page used farthest in future. Optimal is theoretical because future is unknown.

**Detailed Explanation:** LRU uses locality better than FIFO but exact tracking costs work; Clock approximates it efficiently.

**Hinglish:** FIFO oldest page hataata hai, LRU least recently used, Optimal future mein sabse late use hone wala page.

> **Interview Tip:** Optimal is benchmark, not deployable predictor.

**Follow-up & Answer:** **Belady anomaly?** FIFO can fault more with more frames. **Clock?** Reference-bit approximate LRU.

### 37. What is internal versus external fragmentation?

**Short Answer:** Internal fragmentation is wasted space inside an allocated unit; external fragmentation is free space split into small holes that cannot satisfy a large contiguous allocation.

**Detailed Explanation:** Paging eliminates external fragmentation in physical frames but not last-page internal waste.

**Hinglish:** Internal waste allocated block ke andar hai; external free holes scattered hone ka problem hai.

> **Interview Tip:** Give paging as nuanced example.

**Follow-up & Answer:** **Compaction?** Moves allocations to combine holes. **Why costly?** Data movement/reference updates.

### 38. What is segmentation versus paging?

**Short Answer:** Segmentation uses variable-size logical regions; paging uses fixed-size blocks. Segmentation naturally represents code/data units but can have external fragmentation.

**Detailed Explanation:** Modern systems often use paging as main physical-memory mechanism, sometimes with segmentation features for protection/compatibility.

**Hinglish:** Segmentation logical variable-size units use karta hai; paging fixed-size pages use karta hai.

> **Interview Tip:** Do not say they are identical because both split memory.

**Follow-up & Answer:** **Page/frame relation?** Equal size. **Segment size?** Variable.

### 39. What is stack versus heap?

**Short Answer:** Stack stores per-thread call frames with LIFO lifetime; heap supports dynamic object allocation with flexible lifetime. Stacks are private per thread; heap is usually shared process memory.

**Detailed Explanation:** Stack overflow and heap OOM/leaks have different causes and debugging paths.

**Hinglish:** Stack function-call memory hai; heap dynamic objects ke liye flexible memory hai.

> **Interview Tip:** Each thread has its own stack.

**Follow-up & Answer:** **Why stack fast?** Pointer movement/LIFO. **Java leak?** Yes, reachable unwanted objects remain uncollected.

### 40. What is a memory leak?

**Short Answer:** A memory leak is memory retained or never freed beyond intended lifetime. It happens in manual-memory code and garbage-collected code through unwanted references.

**Detailed Explanation:** Leak symptoms include growing heap/RSS, increasing GC, paging, latency, and eventual allocation failure/OOM.

**Hinglish:** Leak tab hai jab memory logically useless ho but release/collect nahi ho rahi.

> **Interview Tip:** GC does not solve reachable-object leaks.

**Follow-up & Answer:** **Find it?** Trend/profile/heap dump. **Fix?** Remove ownership or bound cache/lifetime.

### 41. What is a filesystem?

**Short Answer:** A filesystem organizes persistent storage as files/directories, tracks metadata and blocks, enforces permissions, and provides recovery/durability behaviour.

**Detailed Explanation:** It maps human-readable paths to data and device blocks through kernel structures and drivers.

**Hinglish:** Filesystem disk par files, directories, metadata aur permissions ko manage karta hai.

> **Interview Tip:** Filesystem is more than a folder tree.

**Follow-up & Answer:** **Mount?** Attach filesystem at directory. **Journaling?** Log updates for crash recovery.

### 42. What is an inode?

**Short Answer:** An inode is a Unix-like filesystem metadata record containing permissions, owner, size, timestamps, link count, and references to data blocks. Names are stored in directories.

**Detailed Explanation:** Multiple hard links can point to one inode.

**Hinglish:** Inode file ki metadata/data-location entry hai; filename directory mapping mein hota hai.

> **Interview Tip:** State that inode normally does not store filename.

**Follow-up & Answer:** **See inode?** `ls -li`. **Inode exhaustion?** Filesystem can reject new files despite free data blocks.

### 43. Hard link versus symbolic link?

**Short Answer:** A hard link is another directory entry to the same inode; a symlink is a separate file containing target pathname. Symlinks can cross filesystems and become dangling.

**Detailed Explanation:** Removing one hard-link name does not remove data until link count and open references reach zero.

**Hinglish:** Hard link same inode ka another naam hai; soft link path ko point karta hai.

> **Interview Tip:** Explain delete behaviour.

**Follow-up & Answer:** **Cross filesystem hard link?** Usually no. **Dangling link?** Symlink whose target no longer exists.

### 44. What is a file descriptor?

**Short Answer:** A file descriptor is a small per-process integer handle for an open file, socket, pipe, or device. Kernel maps it to an open-file object/state.

**Detailed Explanation:** Descriptors must be closed; leaks can exhaust per-process/system limits and prevent connections/files opening.

**Hinglish:** FD process ka handle number hai, file/socket/pipe access karne ke liye.

> **Interview Tip:** FD is not the file contents or inode itself.

**Follow-up & Answer:** **Inspect?** `lsof -p PID`. **FD leak?** Open handles grow until `EMFILE`/limit errors.

### 45. Why can disk remain full after deleting a file?

**Short Answer:** Deleting pathname removes directory entry, but blocks remain allocated while a process holds the file open. The space releases after final descriptor closes.

**Detailed Explanation:** This often occurs with rotated logs; `lsof +L1` can find deleted open files.

**Hinglish:** File delete dikh rahi hai but process ne old FD open rakha hai, isliye disk space abhi free nahi hota.

> **Interview Tip:** Compare `df` and `du`.

**Follow-up & Answer:** **Fix?** Safely restart/reload owning process or make it close descriptor. **Why du low?** Deleted name is no longer in directory tree.

### 46. What is disk scheduling?

**Short Answer:** Disk scheduling orders block I/O requests to reduce delay and balance fairness. It matters especially for HDD seek/rotation, but SSD queues/controllers still benefit from sensible scheduling.

**Detailed Explanation:** FCFS, SSTF, SCAN, and C-SCAN demonstrate ordering trade-offs.

**Hinglish:** Disk scheduler I/O request order choose karta hai taaki latency aur fairness improve ho.

> **Interview Tip:** SSD means lower mechanical cost, not no queueing cost.

**Follow-up & Answer:** **SSTF risk?** Starvation. **SCAN idea?** Sweep like elevator then reverse.

### 47. HDD versus SSD?

**Short Answer:** HDD uses mechanical heads/platters, so seeks and random I/O are expensive. SSD uses flash/controller mapping, giving much better random latency but still has queues, wear, and write amplification.

**Detailed Explanation:** Workload, queue depth, durability, and controller behaviour matter more than a simplistic “SSD is instant.”

**Hinglish:** HDD mechanical hai; SSD flash-based fast hai but controller/GC/write amplification latency la sakte hain.

> **Interview Tip:** Explain both latency and endurance nuance.

**Follow-up & Answer:** **IOPS?** Operations per second. **Throughput?** Bytes per second.

### 48. What is user mode versus kernel mode?

**Short Answer:** User mode restricts application privileges; kernel mode allows OS core to execute privileged instructions and access protected hardware/memory. CPU hardware enforces the boundary.

**Detailed Explanation:** Apps request privileged work through system calls rather than direct device access.

**Hinglish:** User mode app ka restricted mode hai; kernel mode OS ka privileged mode hai.

> **Interview Tip:** This is a protection mechanism, not merely a speed setting.

**Follow-up & Answer:** **Why transition?** For controlled privileged operation. **Can app execute privileged instruction?** CPU traps/rejects it.

### 49. What is a system call?

**Short Answer:** A system call is controlled entry from user program into kernel to request services such as open, read, write, fork, memory mapping, or networking.

**Detailed Explanation:** Kernel validates inputs/permissions, performs or queues work, then returns a result/error.

**Hinglish:** System call app aur kernel ke beech controlled gate hai.

> **Interview Tip:** It is not an ordinary in-process function call.

**Follow-up & Answer:** **Examples?** `open`, `read`, `write`, `fork`, `execve`. **Can it block?** Yes, for I/O, locks, sockets, etc.

### 50. Interrupt versus exception?

**Short Answer:** An interrupt is usually an asynchronous external hardware event; an exception is synchronous to current instruction, such as divide-by-zero or page fault. Both transfer control to kernel handlers.

**Detailed Explanation:** System calls use a controlled synchronous entry mechanism, often grouped with traps/exceptions in architecture discussions.

**Hinglish:** Interrupt external event ka signal hai; exception current instruction ki wajah se aata hai.

> **Interview Tip:** Page fault is a classic exception example.

**Follow-up & Answer:** **ISR?** Interrupt service routine. **Why short?** To avoid delaying other work/interrupts.

### 51. What is polling versus interrupt-driven I/O?

**Short Answer:** Polling repeatedly checks device status; interrupt-driven I/O lets device notify CPU on events. Polling can waste idle CPU but can be efficient for controlled high-rate batches.

**Detailed Explanation:** Systems often combine both, using interrupts at low rate and polling/batching under heavy load.

**Hinglish:** Polling baar-baar check karta hai; interrupt device complete hone par CPU ko notify karta hai.

> **Interview Tip:** Neither is universally superior.

**Follow-up & Answer:** **Why interrupts costly at high rate?** Handler overhead/interrupt storms. **Example hybrid?** Linux NAPI networking.

### 52. What is DMA?

**Short Answer:** Direct Memory Access lets a device transfer bulk data to/from RAM without CPU copying every byte. CPU sets it up and handles completion.

**Detailed Explanation:** DMA reduces CPU copy overhead but needs buffers, mapping, synchronization, and interrupt/deferred completion handling.

**Hinglish:** DMA device ko RAM mein direct bulk transfer karne deta hai; CPU setup aur completion handle karta hai.

> **Interview Tip:** DMA does not eliminate all CPU work.

**Follow-up & Answer:** **Use case?** Disk/network I/O. **Why helpful?** CPU can execute other tasks during transfer.

### 53. What is a device driver?

**Short Answer:** A device driver is kernel software that translates generic OS operations into hardware-specific commands and handles device events/errors.

**Detailed Explanation:** Drivers provide a stable abstraction so applications do not need to know device registers/protocols.

**Hinglish:** Driver OS aur specific hardware ke beech translator/controller hai.

> **Interview Tip:** Drivers often run privileged, so bugs can be serious.

**Follow-up & Answer:** **User app talks directly?** Usually through OS/driver APIs. **Driver failure effect?** May affect kernel/device/system stability.

### 54. What is a monitor and condition variable?

**Short Answer:** A monitor encapsulates shared state with mutual exclusion; a condition variable lets threads sleep until a predicate becomes true while atomically releasing/reacquiring the associated lock.

**Detailed Explanation:** Waiters must recheck condition in a loop because wakeups can be spurious or another thread can consume the state first.

**Hinglish:** Monitor lock plus shared state ka structured abstraction hai; condition variable event/predicate wait ke liye hai.

> **Interview Tip:** Condition variable does not itself store condition state.

**Follow-up & Answer:** **Why while?** Spurious/racing wakeups. **Signal vs broadcast?** Wake one eligible waiter versus all.

### 55. Explain producer-consumer problem.

**Short Answer:** Producers add items to a bounded buffer; consumers remove them. Correct solution prevents concurrent queue corruption and waits when buffer is full/empty.

**Detailed Explanation:** Typically a mutex protects buffer plus semaphores/conditions track empty slots and full items.

**Hinglish:** Producer item add karta hai, consumer remove; buffer full/empty hone par proper wait aur lock chahiye.

> **Interview Tip:** Mention both capacity and mutual exclusion.

**Follow-up & Answer:** **Backpressure?** Slow producers when buffer full. **Why not one semaphore only?** Need protect queue structure and track two conditions.

### 56. Explain readers-writers problem.

**Short Answer:** Many readers may safely access unchanged data together, but writers need exclusive access. Policy must balance reader throughput against writer starvation.

**Detailed Explanation:** Read-write locks implement this pattern but may not help under write-heavy/short critical sections.

**Hinglish:** Multiple readers parallel read kar sakte hain; writer ke time exclusive access chahiye.

> **Interview Tip:** Discuss fairness policy.

**Follow-up & Answer:** **Writer starvation?** Readers keep arriving and delay writer. **Fix?** Fair/writer-preference admission policy.

### 57. Explain dining philosophers problem.

**Short Answer:** Philosophers need two shared forks; if all acquire one then wait for other, circular wait creates deadlock. It models multi-lock resource acquisition.

**Detailed Explanation:** Fixes include global lock ordering, arbitrator/semaphore, or try-lock with randomized backoff.

**Hinglish:** Multiple locks wrong order mein lene se circular wait aur deadlock ka example hai.

> **Interview Tip:** Lock ordering is the clearest solution.

**Follow-up & Answer:** **Which Coffman condition breaks?** Circular wait. **Can starvation remain?** Yes, unless fairness is designed.

### 58. What is Banker’s Algorithm?

**Short Answer:** Banker’s Algorithm grants a request only if system remains in a safe state, using declared maximum claims, current allocation, need, and available resources.

**Detailed Explanation:** It is a classic avoidance algorithm; production use is limited because future maximum demands are hard to know.

**Hinglish:** Banker request tabhi grant karta hai jab safe sequence bachi rahe.

> **Interview Tip:** Unsafe is not automatically deadlocked.

**Follow-up & Answer:** **Need formula?** Max minus Allocation. **Why rare?** Dynamic workloads rarely know maximum claims.

### 59. What is a safe state?

**Short Answer:** A safe state has at least one order in which every process can obtain remaining resources, finish, and release its allocation. An unsafe state is riskier, not necessarily deadlocked.

**Detailed Explanation:** Safety reasoning is the core check in Banker’s Algorithm.

**Hinglish:** Safe state mein ek completion order exist karta hai jisme sab processes finish kar sakte hain.

> **Interview Tip:** Do not equate unsafe with current deadlock.

**Follow-up & Answer:** **Safe sequence?** Such a completion order. **Detection?** Finds existing cycles rather than predicting safety.

### 60. What is deadlock detection and recovery?

**Short Answer:** Detection finds cycles in resource/wait-for graphs after requests are allowed. Recovery aborts/rolls back/preempts selected victim resources where safe.

**Detailed Explanation:** Databases commonly detect deadlock and abort one transaction; application retries must be safe and bounded.

**Hinglish:** Detection cycle find karta hai; recovery ek victim abort/rollback karke cycle break karta hai.

> **Interview Tip:** Recovery policy has business cost.

**Follow-up & Answer:** **Victim criteria?** Rollback cost, priority, impact. **Why retry?** Aborted work may be safely reattempted.

### 61. What is copy-on-write?

**Short Answer:** Copy-on-write lets processes initially share read-only physical pages after fork; OS copies a page only when one process writes. It reduces unnecessary memory copying.

**Detailed Explanation:** It is also used in filesystem/storage snapshot designs with different implementation details.

**Hinglish:** COW mein copy immediately nahi hoti; write ke time separate page banaya jaata hai.

> **Interview Tip:** It is an optimization, not shared writable memory.

**Follow-up & Answer:** **Fork benefit?** Fast child creation. **Write fault?** Kernel allocates/copies page then permits write.

### 62. What is memory-mapped I/O/file mapping?

**Short Answer:** Memory mapping maps file/device pages into a process virtual address space, so normal loads/stores access data and faults/page cache bring pages in as needed.

**Detailed Explanation:** It can simplify random access and sharing but needs careful flushing, bounds, and concurrency handling.

**Hinglish:** `mmap` file ko virtual memory region ki tarah map karta hai; access page faults se load ho sakta hai.

> **Interview Tip:** Mapped writes are not automatically durable immediately.

**Follow-up & Answer:** **Why useful?** Efficient random/shared access. **Durability?** Use sync/msync/fsync semantics as required.

### 63. What is locality of reference?

**Short Answer:** Locality means programs tend to reuse recently accessed data (temporal locality) and nearby data (spatial locality). Caches and virtual memory rely on it.

**Detailed Explanation:** Poor locality causes more cache/TLB misses and page faults, reducing performance.

**Hinglish:** Program usually recent aur nearby memory ko repeat use karta hai—isi ko locality bolte hain.

> **Interview Tip:** Connect locality to cache and paging.

**Follow-up & Answer:** **Temporal?** Reuse same item. **Spatial?** Use nearby addresses.

### 64. What is page cache?

**Short Answer:** Page cache is kernel memory used to cache file data, reducing storage reads and buffering writes. Linux may show it as used memory but it is often reclaimable.

**Detailed Explanation:** It improves I/O performance; under memory pressure kernel can reclaim clean cache pages.

**Hinglish:** Page cache RAM mein file data cache karta hai, isliye repeated disk reads fast ho sakte hain.

> **Interview Tip:** Low `free` memory alone is not a problem.

**Follow-up & Answer:** **Why reclaimable?** Disk-backed clean data can be reread. **How see summary?** `free -h`.

### 65. What is an out-of-memory condition?

**Short Answer:** It occurs when allocations cannot be satisfied within available/reclaimable memory and limits. Linux may reclaim cache, swap, fail allocation, or invoke OOM killer; containers have cgroup-specific limits.

**Detailed Explanation:** OOM diagnosis needs kernel logs, cgroup memory events, RSS/heap/native memory, and workload context.

**Hinglish:** Memory reclaim ke baad bhi enough memory na mile toh OOM condition aati hai; container limit bhi important hai.

> **Interview Tip:** Host free memory is not same as container headroom.

**Follow-up & Answer:** **OOM killer?** Linux last-resort process selection/kill. **Prevent?** Bound caches/concurrency and set accurate limits.

### 66. What is CPU cache versus TLB?

**Short Answer:** CPU cache stores recently used instruction/data contents; TLB stores recent virtual-to-physical address translations. Both exploit locality but cache different things.

**Detailed Explanation:** A TLB hit can still have a data-cache miss, and vice versa.

**Hinglish:** CPU cache actual data/instructions cache karta hai; TLB address translation cache karta hai.

> **Interview Tip:** Never call TLB a replacement for page table or data cache.

**Follow-up & Answer:** **TLB miss cost?** Page-table walk. **Cache miss?** Fetch data from lower cache/RAM.

### 67. How do you diagnose high CPU on Linux?

**Short Answer:** Identify host/PID/thread with `top`, `htop`, `ps`, and `top -H -p PID`; correlate request rate, GC, run queue, logs, profiles, and stack traces before mitigation.

**Detailed Explanation:** Causes include hot code, busy loops, contention/spinning, GC, compression, or unexpected load.

**Hinglish:** Pehle hot process/thread identify karo, phir profile/stack dump se actual hot code dekho.

> **Interview Tip:** CPU percentage alone is not a root cause.

**Follow-up & Answer:** **Why top -H?** Shows threads. **Safe mitigation?** Rate-limit/scale/rollback after capturing evidence.

### 68. How do you diagnose memory growth?

**Short Answer:** Trend heap/RSS/cgroup memory and swap, distinguish cache/native/managed memory, capture profile/dump, inspect retained objects/allocations, and correlate growth with traffic/deployments.

**Detailed Explanation:** A leak is persistent unexpected retention; a planned cache or temporary workload peak needs different treatment.

**Hinglish:** Memory source identify karo—heap, native, cache, threads ya page cache—sirf “used RAM” par decision mat lo.

> **Interview Tip:** Time series is stronger than one snapshot.

**Follow-up & Answer:** **RSS?** Resident RAM. **PSS?** Proportional share of shared mappings, useful for attribution.

### 69. How do you diagnose a stuck process?

**Short Answer:** Check process/thread state, logs, sockets/files, `strace` sample, thread dump, and dependent services. Determine whether it waits on I/O, lock/futex, network, child, or CPU.

**Detailed Explanation:** Capture evidence before SIGKILL; some `D` state I/O waits cannot immediately respond to signals.

**Hinglish:** Stuck process ke liye thread dump aur `strace` batate hain woh kis syscall/lock par wait kar raha hai.

> **Interview Tip:** “Restart it” is mitigation, not diagnosis.

**Follow-up & Answer:** **Futex wait?** Often user-space lock/condition waiting. **D state?** Uninterruptible kernel sleep, often I/O.

### 70. How do you diagnose disk full or slow disk I/O?

**Short Answer:** Use `df -h`, `df -i`, `du`, and `lsof +L1` for space; use `iostat -xz 1`, `vmstat 1`, app latency, and device metrics for I/O delay.

**Detailed Explanation:** Separate capacity issue, inode exhaustion, deleted-open file, and true device saturation.

**Hinglish:** Disk full aur disk slow alag problems hain—space ke liye `df/du`, latency ke liye `iostat/vmstat` dekho.

> **Interview Tip:** Compare df and du for deleted-open logs.

**Follow-up & Answer:** **df -i?** Inode availability. **iowait high?** CPU waits for I/O, but interpret with device/workload metrics.

## Advanced — 71–90

### 71. What is priority inversion and priority inheritance?

**Short Answer:** Priority inversion occurs when high-priority task waits for low-priority lock holder while medium tasks preempt it. Priority inheritance temporarily boosts low holder so it can release lock.

**Detailed Explanation:** This prevents unbounded delay in priority-based systems but does not replace good lock design.

**Hinglish:** High priority task low priority lock holder ke wait mein stuck hai aur medium tasks usko run nahi karne dete—ye priority inversion hai.

> **Interview Tip:** Aging and inheritance solve different problems.

**Follow-up & Answer:** **Aging solves?** Starvation. **Inheritance solves?** Priority inversion around held lock.

### 72. What are user-level versus kernel-level threads?

**Short Answer:** User-level threads are scheduled by a runtime; kernel threads are visible to OS scheduler. User scheduling can be cheap, but blocking/kernel integration depends on mapping model.

**Detailed Explanation:** Modern runtimes may use many user tasks over a smaller/larger pool of kernel threads.

**Hinglish:** User threads runtime manage karta hai; kernel threads ko OS scheduler directly manage karta hai.

> **Interview Tip:** Explain blocking implications, not only speed.

**Follow-up & Answer:** **Many-to-one risk?** One blocking syscall can block runtime thread. **One-to-one?** Each user thread maps to kernel thread.

### 73. What is a thread pool and why bound it?

**Short Answer:** A thread pool reuses a fixed/bounded set of workers for tasks. Bounds control stack memory, context switches, downstream pressure, and queue growth.

**Detailed Explanation:** Pool size depends on CPU cores, I/O wait, task cost, latency targets, and dependency capacity.

**Hinglish:** Thread pool threads reuse karta hai; unbounded threads memory aur context-switching se server ko hurt kar sakte hain.

> **Interview Tip:** More threads can lower throughput for CPU-bound work.

**Follow-up & Answer:** **CPU-bound size?** Near core count. **I/O-bound?** May be larger but still bounded/measured.

### 74. What is a cgroup and namespace in containers?

**Short Answer:** Namespaces isolate views of resources such as PID, mount, network, and users; cgroups account and limit resources such as CPU, memory, and process count. Containers share host kernel.

**Detailed Explanation:** Isolation is strong but not equivalent to a separate guest kernel as in normal VMs.

**Hinglish:** Namespace view isolate karta hai; cgroup resource limits/accounting apply karta hai.

> **Interview Tip:** Containers are processes with kernel features, not magic lightweight VMs.

**Follow-up & Answer:** **Memory limit effect?** Cgroup OOM can kill workload. **PID namespace?** Process IDs appear isolated inside container.

### 75. What is graceful shutdown?

**Short Answer:** Graceful shutdown stops accepting new work, drains or cancels bounded in-flight work, closes resources, flushes required state, and exits before deadline after receiving termination signal.

**Detailed Explanation:** It avoids dropped requests and corrupted state during deploys, autoscaling, or failures.

**Hinglish:** Graceful shutdown mein new requests stop, existing work controlled way mein finish/cancel, resources close, phir process exit karta hai.

> **Interview Tip:** Signals and orchestrator timeout matter.

**Follow-up & Answer:** **SIGTERM?** Request cooperative shutdown. **SIGKILL?** Forced uncatchable termination after deadline/last resort.

### 76. What is file-descriptor exhaustion?

**Short Answer:** It happens when a process/system reaches open-file/socket descriptor limit, causing open/accept/socket operations to fail, often with `EMFILE`. It can take down high-connection services.

**Detailed Explanation:** Causes include leaks, too many clients, unbounded pools, and low limits; inspect with `lsof`, `/proc`, and limits.

**Hinglish:** FD limit hit ho jaaye toh new files/sockets open nahi honge; service connections fail kar sakti hain.

> **Interview Tip:** FD includes sockets and pipes, not only disk files.

**Follow-up & Answer:** **Fix?** Close leaks, bound connections, raise justified limit. **Inspect?** `lsof -p PID` and `ulimit -n`.

### 77. What is asynchronous I/O and why use it?

**Short Answer:** Async I/O starts an operation and lets task continue; completion is delivered later via callback, event loop, future, or readiness notification. It reduces blocked-thread need for many I/O waits.

**Detailed Explanation:** It increases state-management complexity and does not make underlying device/network instant.

**Hinglish:** Async I/O request start karke thread ko wait mein block nahi karta; completion baad mein event/future se aata hai.

> **Interview Tip:** Async improves concurrency, not automatic speed.

**Follow-up & Answer:** **Blocking alternative?** One thread waits. **Trade-off?** Fewer threads but more complex control flow/backpressure.

### 78. What is an event loop?

**Short Answer:** An event loop waits for readiness/completion events and dispatches short handlers, allowing one/few threads to manage many I/O connections. Blocking handler code stalls its loop.

**Detailed Explanation:** CPU-heavy work should be offloaded to worker threads/processes to preserve responsiveness.

**Hinglish:** Event loop events wait karta hai aur ready work handle karta hai; usmein blocking/CPU-heavy code avoid karo.

> **Interview Tip:** Event loop is not a single-thread performance miracle.

**Follow-up & Answer:** **Why block bad?** Delays all connections on loop. **Use worker pool?** For CPU/blocking tasks.

### 79. What is lock-free programming?

**Short Answer:** Lock-free algorithms guarantee system-wide progress: some thread completes in finite steps even if others pause. They use atomic primitives like compare-and-swap.

**Detailed Explanation:** Lock-free does not mean wait-free, simpler, or automatically faster; memory ordering and ABA issues make design difficult.

**Hinglish:** Lock-free mein mutex nahi hota aur at least system-level progress guarantee hoti hai, but implementation complex hoti hai.

> **Interview Tip:** Do not equate lock-free with no synchronization.

**Follow-up & Answer:** **Wait-free?** Every thread completes in bounded steps. **CAS?** Update only if value equals expected old value.

### 80. What is memory ordering?

**Short Answer:** Memory ordering defines when reads/writes by one thread become visible to another and in what order. CPUs/compilers can reorder operations unless synchronization establishes required happens-before relationships.

**Detailed Explanation:** Locks, atomics, and memory fences provide ordering/visibility semantics in language/runtime models.

**Hinglish:** Multi-core mein writes instantly/ordered visible maan lena wrong hai; locks/atomics ordering guarantee dete hain.

> **Interview Tip:** Race-free code needs visibility as well as exclusion.

**Follow-up & Answer:** **Happens-before?** Defined ordering/visibility guarantee. **Volatile?** Language-specific visibility semantics, not universal atomicity.

### 81. What is a major versus minor page fault?

**Short Answer:** A minor fault is resolved without storage I/O, often by mapping a resident page; a major fault waits for storage to read needed data. Major faults strongly hurt latency.

**Detailed Explanation:** Names/statistics vary slightly by OS, but the key distinction is I/O wait.

**Hinglish:** Minor fault memory mein available data se solve hota hai; major fault disk I/O wait karta hai.

> **Interview Tip:** Page fault is not automatically a crash.

**Follow-up & Answer:** **Why count?** Detect memory pressure/locality. **All major faults swap?** No, file-backed pages can require disk read too.

### 82. What is swap and when is it harmful?

**Short Answer:** Swap is disk-backed backing for evicted anonymous pages. It can avoid immediate OOM but is much slower than RAM and heavy swapping causes latency/thrashing.

**Detailed Explanation:** Controlled swap may be useful; sustained swap-in/out under active workload is a warning.

**Hinglish:** Swap RAM shortage mein help karta hai but disk slow hai, isliye heavy swap performance ko damage karta hai.

> **Interview Tip:** “Disable swap always” is too absolute.

**Follow-up & Answer:** **Why keep swap?** Absorb transient pressure/avoid abrupt OOM. **Thrashing sign?** High swap I/O plus poor CPU useful work.

### 83. What happens on a page fault step by step?

**Short Answer:** Hardware traps to kernel, which validates address/permission; for legal absent page it obtains frame, writes dirty victim if needed, reads/maps target page, updates tables/TLB, then retries instruction.

**Detailed Explanation:** Illegal access instead leads to process signal/error rather than loading arbitrary memory.

**Hinglish:** Fault par kernel legality check karta hai, page load/map karta hai, mapping update karke instruction retry karta hai.

> **Interview Tip:** Mention legal-vs-invalid branch.

**Follow-up & Answer:** **Victim choice?** Replacement policy. **Dirty victim?** Must be written back before reuse.

### 84. How does `fork` interact with threads?

**Short Answer:** In a multithreaded process, after `fork` only the calling thread exists in child, but copied memory may contain locks held by vanished threads. Child should usually quickly `exec` or use async-signal-safe operations.

**Detailed Explanation:** This is a subtle source of deadlock in child processes after fork.

**Hinglish:** Multithreaded parent se fork ke baad child mein sirf calling thread aata hai; held locks ka state dangerous ho sakta hai.

> **Interview Tip:** This is a strong advanced Unix nuance.

**Follow-up & Answer:** **Why exec soon?** Replaces problematic inherited memory/runtime state. **Can child use normal lock?** Risky if lock was held at fork.

### 85. What is memory overcommit?

**Short Answer:** Overcommit lets system promise more virtual memory than immediately available RAM/swap, assuming not all allocations are used at once. It improves flexibility but can defer failure to later OOM.

**Detailed Explanation:** OS policies vary; applications should handle allocation limits and set sane resource bounds.

**Hinglish:** Overcommit mein OS future memory demand ko optimistic way mein allow karta hai; actual use par OOM risk aa sakta hai.

> **Interview Tip:** Allocation success may not guarantee future physical availability.

**Follow-up & Answer:** **Benefit?** Sparse/lazy allocation. **Risk?** Sudden OOM under real usage.

### 86. What is write-ahead logging and `fsync` relevance?

**Short Answer:** Write-ahead logging records intended changes durably before applying dependent data changes, enabling recovery. `fsync` asks OS to push file updates toward stable storage before acknowledging critical durability point.

**Detailed Explanation:** Databases use this to recover committed transactions after crashes; storage hardware/cache guarantees also matter.

**Hinglish:** WAL pehle durable log likhta hai, phir data pages update hote hain; crash recovery possible hoti hai.

> **Interview Tip:** `write()` success is not always durable storage success.

**Follow-up & Answer:** **Why costly?** Storage flush latency. **Why batch?** Group commit amortizes sync cost.

### 87. Why can too many threads hurt a service?

**Short Answer:** Each thread uses stack/native resources and adds scheduler/context-switch/cache/lock contention. Too many runnable threads increase queueing and reduce CPU cache efficiency.

**Detailed Explanation:** Thread count must match workload and downstream capacity; I/O wait does not justify unbounded creation.

**Hinglish:** Too many threads se memory, context switch aur lock contention badhta hai; throughput bhi gir sakta hai.

> **Interview Tip:** Use bounded pools and backpressure.

**Follow-up & Answer:** **CPU-bound pool size?** Near cores. **I/O-bound?** More possible, but measure/bound by memory and downstream. **Python vs. Java?** In Python, CPU-bound tasks must use multi-processing (separate OS processes) due to the GIL; Java achieves true multi-core thread parallelism with worker pools.

### 88. How does OS scheduling affect API tail latency?

**Short Answer:** CPU queueing delays runnable request threads; preemption, context switches, GC, lock contention, and noisy neighbors can raise p95/p99 even if average CPU seems moderate.

**Detailed Explanation:** Measure run queue, per-thread CPU, blocked time, cgroup throttling, and request phase timings.

**Hinglish:** Request thread ko CPU time late mile, lock wait ho, ya GC/scheduler delay ho toh p99 latency spike hoti hai.

> **Interview Tip:** Tail latency is not explained by average CPU alone.

**Follow-up & Answer:** **Noisy neighbor?** Other workload competing for shared CPU/cache/I/O. **Mitigate?** Limits/isolation/scale/queue bounds.

### 89. How do containers change OS troubleshooting?

**Short Answer:** Containers add cgroup limits and namespace views, so inspect both process metrics and container-level CPU/memory/PID/filesystem limits/events. Host-wide free resources may not be available to container.

**Detailed Explanation:** OOM, throttling, and filesystem visibility can differ inside versus outside container.

**Hinglish:** Container mein host metrics alone enough nahi; cgroup limit aur namespace view bhi check karo.

> **Interview Tip:** Always ask for cgroup memory/CPU limit.

**Follow-up & Answer:** **OOMKilled?** Container exceeded/was killed under cgroup pressure. **CPU throttling?** Cgroup quota limits runnable CPU time.

### 90. How would you design a robust worker queue using OS concepts?

**Short Answer:** Use bounded queue, bounded workers, backpressure/rejection, cancellation/deadlines, idempotent jobs, durable queue if required, metrics, and graceful shutdown/draining.

**Detailed Explanation:** Choose thread/process/event-loop model by CPU versus I/O workload and isolate unreliable dependencies.

**Hinglish:** Queue bounded rakho, workers limited, overload par backpressure, jobs idempotent aur shutdown graceful hona chahiye.

> **Interview Tip:** Unbounded queue is a memory-risk answer.

**Follow-up & Answer:** **Why idempotent?** Safe retry. **Why durable?** Jobs survive process crash when business requires it.

## Tricky / Interview Trap — 91–100

### 91. Does every program become a process automatically?

**Short Answer:** No. A program is a file or code; it becomes a process only when OS loads/executes it. One program can have zero, one, or many process instances.

**Detailed Explanation:** A compiled executable on disk is not consuming CPU as a process until launched.

**Hinglish:** Program file apne aap process nahi; execute hone par process banta hai.

> **Common Trap:** Treating program and process as synonyms.

**Follow-up & Answer:** **Multiple instances?** Yes, each process has its own PID/address space.

### 92. Do threads have separate memory?

**Short Answer:** Threads share their process address space, code, heap, and many resources, but each thread has private stack, registers, program counter, and thread-local storage.

**Detailed Explanation:** Shared memory improves communication but enables races and fault spread.

**Hinglish:** Threads heap share karte hain but har thread ka stack aur execution state alag hota hai.

> **Common Trap:** Saying threads share everything.

**Follow-up & Answer:** **Why private stack?** Each has independent call sequence/local frames.

### 93. Is a context switch free because CPU is fast?

**Short Answer:** No. It requires state save/restore, scheduler work, possible address-space change, cache/TLB disruption, and lost locality. Frequent switches can hurt throughput/latency.

**Detailed Explanation:** Cost depends on architecture, workload, and switch type, but it is never conceptually free.

**Hinglish:** CPU fast hai but context switch registers/cache/TLB impact ki wajah se free nahi hota.

> **Common Trap:** Ignoring cache effects.

**Follow-up & Answer:** **Thread vs process switch?** Process often changes address space too, increasing potential overhead.

### 94. Is virtual memory just disk used as RAM?

**Short Answer:** No. Virtual memory is the protected address translation abstraction. Disk/swap can back some evicted pages, but many virtual pages are in RAM, file-backed, shared, or never allocated physically.

**Detailed Explanation:** Its main value is isolation and flexible address spaces, not merely capacity extension.

**Hinglish:** Virtual memory address-space abstraction hai; swap uska optional slow backing part hai.

> **Common Trap:** Defining virtual memory as “hard disk RAM.”

**Follow-up & Answer:** **Page fault always disk?** No, minor faults can resolve without disk I/O.

### 95. Does paging eliminate all fragmentation?

**Short Answer:** Paging eliminates external fragmentation for fixed physical frames, but internal fragmentation remains when an allocation does not fill its final page. Allocators also have their own fragmentation patterns.

**Detailed Explanation:** So the correct statement is nuanced, not absolute.

**Hinglish:** Paging external fragmentation remove karta hai but last page/block ke andar internal waste ho sakta hai.

> **Common Trap:** Saying “paging removes fragmentation completely.”

**Follow-up & Answer:** **External fragmentation?** Scattered holes. **Internal?** Wasted space inside allocated unit.

### 96. Is a TLB miss a page fault?

**Short Answer:** No. TLB miss means translation is not cached; hardware/page-table walk can find a present mapping. Page fault happens when mapping is absent or access is invalid.

**Detailed Explanation:** TLB miss is common and much cheaper than a major fault.

**Hinglish:** TLB miss cache miss hai; page fault page-table/residency problem hai.

> **Common Trap:** Mixing translation cache with memory residency.

**Follow-up & Answer:** **TLB stores?** Page translations. **Page table stores?** Full mapping metadata.

### 97. Does garbage collection prevent memory leaks?

**Short Answer:** No. GC reclaims unreachable objects, but objects referenced accidentally by caches, listeners, statics, or queues remain reachable and leak logically.

**Detailed Explanation:** Native/off-heap resources can also leak independently of managed heap objects.

**Hinglish:** GC unreachable objects free karta hai; unwanted references objects ko alive rakh sakte hain.

> **Common Trap:** “Java cannot leak memory.”

**Follow-up & Answer:** **Find retained objects?** Heap dump/profiler dominator/retainer analysis.

### 98. Is `kill -9` the correct first fix for a stuck process?

**Short Answer:** No. First capture safe evidence and try cooperative SIGTERM if appropriate. SIGKILL prevents cleanup and cannot stop a task immediately if it is in uninterruptible kernel I/O wait.

**Detailed Explanation:** It is a last-resort mitigation, not root-cause analysis.

**Hinglish:** Pehle evidence lo aur graceful SIGTERM try karo; `kill -9` last resort hai.

> **Common Trap:** Confusing restart with diagnosis.

**Follow-up & Answer:** **D state?** Uninterruptible wait, often I/O. **Why no cleanup?** SIGKILL cannot be caught by process.

### 99. Are mutex and semaphore interchangeable?

**Short Answer:** No. A mutex is ownership-based mutual exclusion; a semaphore is a counter/permit synchronization primitive. A binary semaphore may resemble a lock but has different ownership semantics.

**Detailed Explanation:** Choose based on invariant and resource/event model rather than type name.

**Hinglish:** Mutex owner-based lock hai; semaphore count/permit coordination ke liye hai—same nahi hain.

> **Common Trap:** Calling binary semaphore “exactly mutex.”

**Follow-up & Answer:** **Connection pool?** Counting semaphore. **Shared counter?** Mutex/atomic primitive.

### 100. Does adding more threads always improve server throughput?

**Short Answer:** No. More threads help only until CPU, memory, locks, scheduler, connection pools, or downstream services become bottlenecks. Beyond that they raise contention and tail latency.

**Detailed Explanation:** Use workload measurement, bounded pools, queue limits, and backpressure instead of unlimited threads.

**Hinglish:** More threads se throughput guaranteed nahi; contention, memory aur context switching se performance worse ho sakti hai.

> **Common Trap:** Treating concurrency as free scalability.

**Follow-up & Answer:** **CPU-bound sizing?** Near core count. **I/O-bound sizing?** Higher can help but must be bounded and measured.
