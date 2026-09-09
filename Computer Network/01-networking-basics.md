# Networking Basics

## What is it?

A computer network lets independent machines exchange data. The Internet is a network of networks using shared rules called protocols.

## Simple Explanation

Think of a delivery system: an address says *where* a parcel goes, a route says *which roads* it takes, and a protocol says *how* sender and receiver package and acknowledge it.

## Hinglish Explanation

Network ko roads aur courier system ki tarah samjho. Data chhote packets mein travel karta hai. IP address destination batata hai, router agla raasta choose karta hai, aur protocol dono sides ko same rules follow karwata hai.

## Why Do We Need It?

Applications need to communicate across a room or across continents without knowing every cable and router in between. Layered protocols provide interoperability, scale, and fault isolation.

## How It Works

1. An app creates data, for example an API request.
2. Protocol layers add addressing, transport, and link information.
3. Network devices forward the resulting packets or frames.
4. The receiver removes headers and delivers data to the right application.

```text
app data → [TCP/UDP] → [IP] → [Ethernet/Wi-Fi] → wire/radio
                                                 ↓
app data ← [TCP/UDP] ← [IP] ← [Ethernet/Wi-Fi] ← receiver
```

## Important Components / Terminology

| Term | Meaning |
|---|---|
| Bandwidth | Link capacity, usually bits/s |
| Throughput | Actual useful rate achieved |
| Latency | Time for data to travel; RTT is round-trip time |
| Packet loss | Packets discarded or never received |
| Protocol | An agreed format and behaviour |

## Example

A mobile app calls an API. DNS finds an IP, TCP or QUIC carries bytes, and HTTP defines the request and response.

## Real-World Example

Microservices use a mix of private IP routing, service discovery, TLS, and HTTP/gRPC. A slow request might be caused by latency, loss, queueing, or the service—not automatically “the network.”

## Important Properties

| Metric | More is better? | Caveat |
|---|---:|---|
| Bandwidth | Usually | Does not guarantee low delay |
| Throughput | Usually | Bounded by bottlenecks |
| Latency | No | Includes queueing and processing |
| Loss | No | Can trigger TCP retransmission |

### Common Misconceptions

* Bandwidth and throughput are not synonyms.
* A packet is not guaranteed to take the same route as later packets.
* “The Internet” is not one central network.

### What Interviewers Usually Test

Whether you can distinguish capacity, delay, loss, and protocol layers while tracing an everyday request.

### Common Follow-up Questions

1. What is the difference between latency and bandwidth?
2. Why can a fast link still feel slow?
3. What is a bottleneck?
4. Packet versus frame versus segment?
5. What causes packet loss?

### Follow-up Answers

1. **Latency** is delay; **bandwidth** is the maximum capacity of a link. A link can have both high capacity and long delay.
2. Data may wait in queues, travel a long distance, be retransmitted after loss, or wait for a slow server even when bandwidth is high.
3. A bottleneck is the slowest or most limited resource in an end-to-end path, such as a link, CPU, database, or connection pool.
4. A **segment** is TCP data, a **packet** is an IP unit, and a **frame** is a local-link unit that carries the packet one hop.
5. Congestion, weak Wi-Fi, damaged links, overflowing device buffers, policy drops, and faulty hardware can cause loss.
