# Page Replacement Algorithms

## What is it?

When RAM has no free frame, page replacement selects a resident page to evict. The ideal choice avoids evicting a page needed soon, but future references are unknown in real systems.

## Simple Explanation

When your desk is full, you remove a folder. The best folder to remove is the one you will not need for longest, but you cannot see the future, so algorithms use past behaviour as a clue.

## Hinglish Explanation

Memory full ho toh OS ko victim page choose karna padta hai. Optimal future dekh kar choose karta but practical nahi. FIFO old page hataata hai, LRU recently unused page hataata hai, Clock approximate LRU karta hai with reference bit.

## Why Do We Need It?

Physical frames are finite. Replacement policy strongly affects fault rate and therefore application latency when working sets approach RAM capacity.

## How It Works

1. Fault needs a frame and free list is empty.
2. OS chooses a victim using policy and page metadata.
3. Dirty victim is written back if necessary.
4. Incoming page occupies frame and mappings update.

| Algorithm | Rule | Strength | Limitation |
|---|---|---|---|
| FIFO | Evict oldest loaded page | Very simple | Can evict hot page; Belady anomaly |
| Optimal | Evict farthest future use | Theoretical minimum faults | Future unknown |
| LRU | Evict least recently used | Uses locality well | Exact tracking expensive |
| LFU | Evict lowest use count | Can preserve frequent pages | Old history can mislead |
| Clock | Circular scan, clear reference bit | Cheap LRU approximation | Approximate/fairness variation |

## Important Terminology

| Term | Meaning |
|---|---|
| Victim page | Resident page selected for eviction |
| Dirty bit | Page modified and needs writeback before reuse |
| Reference/accessed bit | Hardware/OS clue that page was recently used |
| Belady’s anomaly | More frames causing more faults for some FIFO sequences |
| Stack algorithm | More frames cannot increase faults, e.g. LRU/Optimal |

## Example

For reference sequence `1,2,3,1,4`, LRU evicts the least recently used resident page, not simply the first page originally loaded. Trace frames carefully in interview questions.

## Real-World Example

Linux uses sophisticated reclaim lists and active/inactive heuristics rather than a pure textbook FIFO or LRU. Database buffer managers also implement their own cache replacement policies.

## Important Properties

| Workload | Helpful principle |
|---|---|
| Loop over small data | Keep recent working set |
| One-time scan | Avoid polluting hot cache if possible |
| Large random access | More RAM/cache; policy has limits |

### Common Misconceptions

* Optimal is a benchmark, not a deployable predictor.
* LRU is not free; exact recency tracking costs metadata/work.
* FIFO’s name refers to load order, not necessarily access order.

### What Interviewers Usually Test

Frame-tracing accuracy, Belady’s anomaly, locality reasoning, and distinction between ideal and practical policies.

### Common Follow-up Questions

1. Why is Optimal not practical?
2. What is Belady’s anomaly?
3. FIFO versus LRU?
4. What does dirty bit change?
5. Why Clock is used?

### Follow-up Answers

1. It needs knowledge of future memory references, which an operating system does not have.
2. It is the counterintuitive case where adding frames increases faults, possible with FIFO for certain sequences.
3. FIFO removes oldest loaded page; LRU removes least recently accessed page and often better matches locality.
4. A dirty victim must be written to backing storage before its frame can be reused, increasing eviction cost.
5. It gives a low-overhead approximation to LRU using a circular list and reference bits.
