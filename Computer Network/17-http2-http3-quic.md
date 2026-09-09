# HTTP/2, HTTP/3, and QUIC

## What is it?

HTTP/2 multiplexes binary streams over one TCP connection. HTTP/3 uses QUIC, a secure transport over UDP, to multiplex streams without TCP’s connection-wide head-of-line blocking from a lost packet.

## Simple Explanation

HTTP/1.1 can make requests wait in a line. HTTP/2 creates lanes on one TCP road. HTTP/3 creates independent lanes so loss in one lane need not stop the others.

## Hinglish Explanation

HTTP/2 mein multiple streams TCP connection par chalte hain, but TCP-level packet loss still all streams ko delay kar sakta hai. HTTP/3 ke QUIC streams loss impact ko affected stream tak limit kar sakte hain.

## Why Do We Need It?

Modern pages have many resources and mobile networks change paths often. Multiplexing, header compression, lower setup latency, and connection migration improve practical performance.

## How It Works

1. HTTP/2 negotiates `h2` via ALPN and uses framed streams over TCP/TLS.
2. HTTP/3 negotiates `h3` and runs HTTP frames over QUIC/UDP.
3. QUIC integrates TLS 1.3 handshake, stream flow control, recovery, and connection IDs.
4. A QUIC connection can survive some network-address changes using connection migration when policy permits.

## Important Components / Terminology

| Term | Meaning |
|---|---|
| Multiplexing | Multiple logical streams share a connection |
| HOL blocking | One lost unit delays unrelated later work |
| HPACK/QPACK | HTTP/2/HTTP/3 header compression schemes |
| 0-RTT | Early data option with replay considerations |
| Connection ID | QUIC identifier decoupled from a 4-tuple |

## Example

A browser may use HTTP/3 to a CDN over UDP/443, while a corporate network that blocks UDP makes it fall back to HTTP/2 or HTTP/1.1 over TCP.

## Real-World Example

QUIC is implemented in user space, so applications can evolve transport behaviour faster than kernel TCP stacks. That flexibility also shifts congestion-control and observability work into the QUIC implementation.

## Important Properties

| Property | HTTP/2 | HTTP/3 |
|---|---|---|
| Underlying transport | TCP + TLS | QUIC over UDP |
| Stream multiplexing | Yes | Yes |
| TCP-level HOL | Yes | Avoided across QUIC streams |
| UDP dependency | No | Yes |

### Common Misconceptions

* HTTP/3 is not “HTTP directly over UDP”; it is HTTP over QUIC.
* QUIC does not ignore congestion control.
* HTTP/2 fixes HTTP/1.1 application-level request queuing but cannot remove TCP packet-loss HOL.

### What Interviewers Usually Test

Precise comparison of TCP, QUIC, streams, handshake, and why UDP is a substrate rather than a shortcut.

### Common Follow-up Questions

1. Why does QUIC use UDP?
2. Explain head-of-line blocking at two layers.
3. What is connection migration?
4. What are 0-RTT replay risks?
5. Why can HTTP/3 fall back?

### Follow-up Answers

1. UDP gives QUIC a commonly available packet transport while QUIC implements streams, recovery, security, and congestion control in user space.
2. HTTP/1.1 can queue requests behind earlier requests; TCP can hold all later bytes until an earlier lost byte arrives, affecting HTTP/2 streams sharing that TCP connection.
3. Connection migration lets a QUIC connection continue after a client network/address change by using connection IDs and validation, when server policy allows.
4. Early 0-RTT data can potentially be replayed by an attacker, so servers should accept only safe replay-tolerant requests as early data.
5. Some networks block or impair UDP, so clients may use HTTP/2 or HTTP/1.1 over TCP instead.
