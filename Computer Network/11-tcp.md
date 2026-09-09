# TCP (Transmission Control Protocol)

## What is it?

TCP is a connection-oriented, reliable, ordered **byte-stream** transport protocol. It uses sequence numbers, acknowledgements, retransmission, and windows.

## Simple Explanation

TCP is a courier service that numbers pieces, confirms received bytes, resends missing pieces, and prevents the receiver or network from being overwhelmed.

## Hinglish Explanation

TCP ko reliable stream samjho: sender data ke bytes number karta hai, receiver cumulative ACK bhejta hai, aur missing data timeout ya duplicate ACKs se detect hone par resend hota hai. App ko packet boundaries nahi milti—sirf byte stream milta hai.

## Why Do We Need It?

Applications such as web pages, database protocols, and file transfer need correct ordered bytes despite loss, duplication, reordering, and varying receiver speed.

## How It Works

1. Endpoints establish state with a handshake.
2. Sender sends numbered bytes within the allowed window.
3. Receiver acknowledges bytes received and advertises receive capacity.
4. Sender retransmits suspected losses and adapts to congestion.
5. The OS exposes an ordered stream to each application socket.

## Important Components / Terminology

| Term | Meaning |
|---|---|
| Sequence number | Position of bytes in stream |
| ACK | Usually cumulative indication of next expected byte |
| MSS | Maximum TCP payload per segment, derived from MTU |
| Receive window | Receiver-advertised buffer capacity |
| RTO | Retransmission timeout |

## Example

An HTTP/1.1 response uses TCP underneath. `send()` twice does not guarantee the peer receives two separate `recv()` calls; application framing must define message boundaries.

## Real-World Example

Long-distance database replication can be limited by RTT and TCP window sizing. Large bandwidth alone will not fill the link if the bandwidth-delay product exceeds the usable window.

## Important Properties

| Property | TCP |
|---|---|
| Reliability | Yes, through retransmission/ACK logic |
| Ordering | Yes, delivered byte order |
| Message boundaries | No |
| Congestion control | Yes |
| Encryption | No; use TLS where needed |

### Common Misconceptions

* TCP cannot promise zero loss on the wire; it aims to recover or report failure.
* TCP is not “slow” by definition; its safeguards have trade-offs.
* TCP does not encrypt data.

### What Interviewers Usually Test

The difference between packet delivery and byte-stream semantics, plus reliability mechanisms and costs.

### Common Follow-up Questions

1. Why sequence numbers?
2. What triggers retransmission?
3. What is head-of-line blocking?
4. Why does TCP need buffers?
5. TCP versus TLS?

### Follow-up Answers

1. Sequence numbers identify byte positions so TCP can order data, detect gaps, reject duplicates, and acknowledge progress.
2. TCP retransmits after a retransmission timeout or fast-loss signals such as enough duplicate acknowledgements; exact algorithms vary.
3. In TCP, later bytes cannot be delivered to the application while an earlier missing byte leaves a gap in the ordered stream.
4. Buffers hold bytes waiting for the application, network, acknowledgements, or reordering, and they support flow control.
5. TCP provides ordered reliable transport; TLS adds authentication, encryption, and integrity protection on top of TCP in common HTTPS use.
