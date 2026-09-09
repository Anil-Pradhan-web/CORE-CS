# Transport Layer, Ports, and Sockets

## What is it?

The transport layer delivers application data to the right process on a host. It uses ports and protocols such as TCP and UDP; a socket identifies one endpoint of a communication flow.

## Simple Explanation

IP gets a parcel to an apartment building; a port gets it to the right flat. TCP and UDP are two different delivery services available at that flat.

## Hinglish Explanation

IP ke baad port decide karta hai ki host par kaunsi process data receive karegi—jaise `443` par HTTPS service. Server `listen` karta hai; client usually temporary ephemeral port use karta hai.

## Why Do We Need It?

One server IP hosts many applications and many simultaneous connections. Transport multiplexes and demultiplexes that traffic.

## How It Works

1. Server binds to a local IP/port and listens (TCP) or receives datagrams (UDP).
2. Client chooses source IP and ephemeral source port.
3. Protocol headers carry source/destination ports.
4. OS demultiplexes to a socket, often identified by protocol plus source/destination IP and port.

```text
client 10.0.0.8:51842 ──TCP──> api 10.0.1.20:443
                          ^
                  5-tuple includes protocol too
```

## Important Components / Terminology

| Term | Meaning |
|---|---|
| Port | 16-bit transport endpoint number |
| Well-known port | Conventionally 0–1023, e.g. 53, 80, 443 |
| Ephemeral port | Temporary client-side source port |
| Socket | OS endpoint/API; often IP + port + protocol context |
| 5-tuple | Protocol, source IP/port, destination IP/port |

## Example

`ss -lntp` on Linux lists listening TCP sockets; `ss -nt` shows established TCP connections. A “connection refused” commonly means the host responded but nothing accepted that TCP port.

## Real-World Example

A NAT or firewall tracks flows using tuples. Reusing source ports too quickly or exhausting ephemeral ports can limit an outbound-heavy service.

## Important Properties

| Concept | Scope |
|---|---|
| Port number | One transport protocol on one host |
| IP address | Network host/interface routing |
| Socket connection | Specific endpoint pair/flow |

### Common Misconceptions

* Port 443 is convention, not a protocol guarantee.
* A port is not a physical hole or a network device.
* TCP and UDP can use the same numerical port independently.

### What Interviewers Usually Test

Whether you can explain a client-server flow, port collision, listener, and socket tuple.

### Common Follow-up Questions

1. What is an ephemeral port?
2. Can two processes bind the same port?
3. What is `TIME_WAIT` related to?
4. Why can a server have many connections on 443?
5. What does `bind` versus `connect` do?

### Follow-up Answers

1. An ephemeral port is a temporary client-side source port selected by the OS for an outgoing connection or datagram flow.
2. Usually not with the same IP/protocol, but OS options and different addresses/protocols can permit controlled sharing or reuse.
3. `TIME_WAIT` is TCP close state that lets old duplicate segments expire and supports retransmission of the final acknowledgement.
4. Each connection has a different client IP/port tuple, so one listening port can accept many distinct TCP connections.
5. `bind` attaches a socket to a local address/port; `connect` chooses or uses a remote peer and initiates communication.
