# Boot Process, Interrupts, I/O, and Device Drivers

## What is it?

Boot loads and starts the kernel. Interrupts notify CPU about asynchronous hardware events; exceptions/traps are synchronous events caused by current instruction. Drivers let kernel control devices. DMA moves bulk data between devices and RAM with little CPU copying.

## Simple Explanation

Boot is opening the office and hiring the manager. Polling means repeatedly asking “are you done?” Interrupts are a bell when work completes. DMA is a forklift moving boxes while the manager does not carry every box by hand.

## Hinglish Explanation

Boot firmware se kernel load karta hai. Hardware interrupt asynchronous bell hai—for example disk I/O complete. Exception current instruction ki wajah se hota hai, jaise divide-by-zero/page fault. DMA device ko RAM transfer karne deta hai without CPU copying every byte.

## Why Do We Need It?

CPU should not waste cycles polling slow devices or copying every byte. Hardware diversity needs stable driver interfaces and controlled kernel access.

## How It Works

1. Firmware (UEFI/BIOS) initializes hardware and loads bootloader/kernel.
2. Kernel initializes memory, scheduler, drivers, mounts root filesystem, starts init/systemd.
3. Application requests I/O via syscall; driver configures device/DMA buffers.
4. CPU continues other work; device raises interrupt when ready/completed.
5. Interrupt Service Routine acknowledges event and schedules deferred/bottom-half work as needed.

```text
App → write()/read() → kernel driver → DMA/device
CPU continues work                    │
                    interrupt ← completion
```

## Important Terminology

| Term | Meaning |
|---|---|
| Interrupt | Asynchronous hardware notification |
| Exception | Synchronous event from executing instruction |
| ISR | Interrupt Service Routine, quick handler |
| Polling | CPU repeatedly checks device state |
| DMA | Direct Memory Access transfer device ↔ RAM |
| Driver | Kernel software controlling a device |

## Example

A network card DMA-writes received packets into RAM and raises an interrupt or uses polling mechanisms under load. Kernel networking then queues packets to sockets.

## Real-World Example

High packet rates can cause interrupt overhead. Modern systems may batch work or use polling techniques such as NAPI to avoid interrupt storms while keeping latency acceptable.

## Important Properties

| Mechanism | CPU cost | Latency/use case |
|---|---|---|
| Polling | Can waste CPU when idle | Simple/predictable/high-rate batches |
| Interrupt | Low idle CPU | Efficient sporadic events |
| DMA | Low copy CPU cost | Bulk I/O, needs setup/buffers |

### Common Misconceptions

* A system call is not the same as a hardware interrupt.
* Exceptions include legitimate page faults; they are not always application bugs.
* DMA does not eliminate all CPU work; setup/completion still need handling.

### What Interviewers Usually Test

Boot order, async-vs-sync event distinction, polling-vs-interrupt trade-offs, and DMA’s performance role.

### Common Follow-up Questions

1. Interrupt versus exception?
2. Polling versus interrupt?
3. Why keep ISR short?
4. What does DMA save?
5. What starts after kernel boot?

### Follow-up Answers

1. Interrupt is usually external/asynchronous; exception is caused synchronously by the current CPU instruction.
2. Polling repeatedly checks status and can waste idle CPU; interrupts notify CPU only when an event occurs, though high rates can be costly.
3. Long interrupt handlers delay other interrupts and normal work, increasing system latency.
4. DMA saves CPU from copying every data byte between device and RAM, allowing CPU to run other tasks.
5. The kernel starts an initial user-space process such as systemd/init, which launches services and login/application targets.
