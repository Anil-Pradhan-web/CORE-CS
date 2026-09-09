# UDP and TCP vs UDP

## What is it?

UDP is a minimal, connectionless datagram transport protocol. It provides ports and checksum support but no built-in delivery, ordering, retransmission, or congestion-control guarantee comparable to TCP.

## Simple Explanation

UDP is like sending postcards: each message stands alone and may arrive late, out of order, duplicated, or not at all. That can be useful when freshness matters more than waiting.

## Hinglish Explanation

UDP fast-looking isliye lagta hai kyunki handshake aur reliable delivery ka built-in overhead nahi hai. Lekin reliability free mein nahi milti—application ya protocol like QUIC ko khud handle karna padta hai.

## Why Do We Need It?

Real-time media, games, DNS, telemetry, and protocols such as QUIC may prefer application-controlled recovery and message boundaries over TCP’s ordered byte stream.

## How It Works

1. Sender puts one datagram on a socket with destination IP/port.
2. IP routes it independently.
3. Receiver gets a complete datagram or gets nothing; UDP does not reassemble an application conversation.
4. The application chooses retries, ordering, rate control, and loss handling if needed.

## Important Components / Terminology

| Term | Meaning |
|---|---|
| Datagram | One UDP message with preserved boundaries |
| Checksum | Integrity check; required for IPv6 UDP |
| Jitter | Variation in arrival delay |
| Idempotency | Safe repetition of an operation |
| QUIC | Modern transport built over UDP with richer features |

## Example

DNS commonly uses UDP for normal queries because it avoids setup overhead; it can use TCP in other cases. A voice app can skip an old audio datagram rather than delay new audio waiting for it.

## Real-World Example

Game servers may send frequent state snapshots over UDP. A later snapshot can supersede a lost earlier snapshot; critical events can have explicit acknowledgements.

## Important Properties

| Property | TCP | UDP |
|---|---|---|
| Connection setup | Yes | No built-in connection |
| Ordering/reliability | Built in | Application choice |
| Data model | Byte stream | Datagrams |
| Typical use | HTTP/1.1, DB protocols | DNS, media, QUIC |

### Common Misconceptions

* UDP is not automatically faster end-to-end.
* UDP is not inherently insecure; neither is it encrypted by itself.
* Applications can implement reliability above UDP.

### What Interviewers Usually Test

Trade-offs: freshness, head-of-line blocking, implementation complexity, and congestion responsibility.

### Common Follow-up Questions

1. Why does QUIC use UDP?
2. Is UDP message ordering guaranteed?
3. When should you not use UDP?
4. Does UDP avoid congestion?
5. TCP versus UDP for video calls?

### Follow-up Answers

1. UDP lets QUIC implement modern transport features in user space while using widely deployable IP/UDP connectivity.
2. No. UDP datagrams may be lost, duplicated, reordered, delayed, or delivered independently.
3. Avoid raw UDP when your application needs simple built-in reliable ordered byte delivery and cannot safely implement the missing behaviour.
4. No. UDP lacks built-in TCP congestion control, but responsible UDP applications must still control sending to avoid harming the network.
5. Video calls commonly use UDP-based media because late packets are less valuable than new audio/video; reliability is applied selectively where useful.
