# Flow Control and Congestion Control

## What is it?

Flow control prevents a fast sender overrunning its receiver. Congestion control prevents senders collectively overrunning the network. TCP limits sending by the smaller of receive window and congestion window.

## Simple Explanation

Flow control asks “can *you* process more parcels?” Congestion control asks “can the *roads* handle more traffic?” They are different limits with the same practical effect: slow down.

## Hinglish Explanation

Receiver ke buffer ko protect karne ke liye advertised window use hoti hai. Network mein queues bharne aur loss badhne se bachne ke liye congestion window adjust hoti hai. Effective limit roughly `min(rwnd, cwnd)` hota hai.

## Why Do We Need It?

Without flow control, memory buffers overflow. Without congestion control, queues grow, losses cascade, latency spikes, and competing flows can collapse useful throughput.

## How It Works

1. Receiver advertises available receive window (`rwnd`).
2. Sender estimates network capacity with congestion window (`cwnd`).
3. Slow start grows cautiously from a small amount; congestion avoidance grows more slowly.
4. Loss, ECN, or delay signals can reduce `cwnd`; algorithms vary by OS (for example CUBIC or BBR).

```text
sendable bytes ≈ min(receive window, congestion window) - bytes in flight
```

## Important Components / Terminology

| Term | Meaning |
|---|---|
| Sliding window | Allow multiple unacknowledged bytes in flight |
| rwnd | Receiver capacity advertised to sender |
| cwnd | Sender’s network-capacity estimate |
| BDP | Bandwidth × RTT; useful in sizing in-flight data |
| ECN | Explicit congestion signal without dropping packet |

## Example

At 100 Mbps and 100 ms RTT, a path’s BDP is about 1.25 MB. A much smaller usable window cannot fully utilize that capacity even with no loss.

## Real-World Example

Bufferbloat—oversized queues—can create very high latency even while throughput looks good. Queue management and congestion control make responsiveness better.

## Important Properties

| Control | Protects | Signal |
|---|---|---|
| Flow control | Receiver | Advertised window |
| Congestion control | Network | Loss, ECN, delay estimates |

### Common Misconceptions

* Flow control is not congestion control.
* Loss is a useful but imperfect congestion signal; wireless loss may have other causes.
* More buffers can increase latency.

### What Interviewers Usually Test

Ability to reason about high RTT, loss, windows, throughput, and protecting shared infrastructure.

### Common Follow-up Questions

1. What is sliding window?
2. Why is BDP useful?
3. What is slow start?
4. What is bufferbloat?
5. How do rwnd and cwnd interact?

### Follow-up Answers

1. A sliding window lets a sender keep several bytes in flight before receiving acknowledgements, then advances as ACKs arrive.
2. Bandwidth-delay product estimates how much data must be in flight to keep a high-latency, high-bandwidth path busy.
3. Slow start is TCP’s initial congestion-control phase where the congestion window grows quickly until it finds a limit or reaches a threshold.
4. Bufferbloat is excessive queueing in large buffers, which creates high latency and jitter during load.
5. TCP can send only up to the smaller of receiver capacity (`rwnd`) and network estimate (`cwnd`), after accounting for bytes already in flight.
