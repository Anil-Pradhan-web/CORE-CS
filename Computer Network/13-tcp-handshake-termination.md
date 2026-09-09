# TCP Handshake and Termination

## What is it?

TCP normally establishes a connection with a three-way handshake and closes each direction independently with FIN/ACK exchanges. A reset (RST) aborts rather than gracefully closes.

## Simple Explanation

Before a call, both people confirm they can hear each other and agree on starting positions. At the end, each says they will send no more data, possibly at different times.

## Hinglish Explanation

Three messages isliye chahiye ki client aur server dono ki reachability aur initial sequence numbers confirm ho sakein. Close mein half-close possible hai: ek side FIN bhejke receive karna continue kar sakti hai.

## Why Do We Need It?

TCP needs synchronized sequence-space state before reliably interpreting bytes and needs a safe way to finish pending data while old delayed segments expire.

## How It Works

```text
Client                              Server
  SYN (seq=x)       ───────────────>  LISTEN
                   <───────────────  SYN-ACK (seq=y, ack=x+1)
  ACK (ack=y+1)     ───────────────>  ESTABLISHED

  FIN               ───────────────>  ACK
                   <───────────────  FIN
  ACK               ───────────────>  (client commonly enters TIME_WAIT)
```

1. SYN asks to create state and advertises initial sequence number.
2. SYN-ACK acknowledges client and supplies server sequence number.
3. Final ACK confirms both directions.
4. FIN consumes sequence space and indicates no more bytes in one direction.
5. Active closer’s TIME_WAIT helps handle delayed duplicates and final-ACK retransmission.

## Important Components / Terminology

| Term | Meaning |
|---|---|
| SYN | Synchronize sequence numbers |
| FIN | Graceful end of one send direction |
| RST | Immediate abort/error indication |
| TIME_WAIT | State after active close, not necessarily a bug |
| Half-close | One side has finished sending but can receive |

## Example

Opening an HTTPS TCP connection starts with the TCP handshake, then TLS handshake. A server restarting can send RST for a flow whose state it no longer recognizes.

## Real-World Example

SYN floods try to consume pending connection state. SYN cookies and rate controls reduce impact; load balancers also need carefully sized connection tracking.

## Important Properties

| Event | Meaning |
|---|---|
| SYN timeout | Setup did not complete; can be loss/filtering/no path |
| Connection refused | Usually RST from reachable host/stack with no listener |
| Timeout | Often dropped traffic or unreachable path; not proof of one cause |

### Common Misconceptions

* Three-way handshake is not “SYN, ACK, FIN.”
* FIN is not the same as RST.
* TIME_WAIT usually belongs to the active closer, though details depend on implementation.

### What Interviewers Usually Test

Why three messages, what each field proves, and how to distinguish refused from timeout.

### Common Follow-up Questions

1. Why not a two-way handshake?
2. What if SYN-ACK is lost?
3. Why TIME_WAIT?
4. FIN versus RST?
5. What is a SYN flood?

### Follow-up Answers

1. Two messages do not confirm that the client received the server’s sequence number; the third ACK confirms both directions are synchronised.
2. The server normally retransmits SYN-ACK after a timeout; the client may retransmit SYN if it did not receive a reply.
3. TIME_WAIT lets the active closer resend the final ACK if needed and prevents old delayed segments being confused with a reused connection tuple.
4. FIN gracefully closes one send direction after queued data; RST immediately aborts the connection and may discard unread data.
5. A SYN flood sends many incomplete connection requests to consume server/backlog resources or capacity.
