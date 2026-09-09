# OSI, TCP/IP, and Encapsulation

## What is it?

OSI is a teaching model with seven layers; TCP/IP is the practical Internet suite commonly shown as four layers. They help assign responsibilities, not force every product into one perfect box.

## Simple Explanation

Like sending a gift: the application makes the gift, transport adds delivery handling, IP adds city address, and Ethernet/Wi-Fi adds the next-hop label.

## Hinglish Explanation

Layers ko team members samjho. App ko bas message bhejna hai; lower layers packaging aur delivery handle karte hain. Har layer apne upar wali layer ko service deti hai.

## Why Do We Need It?

Layering lets HTTP work over Wi-Fi or Ethernet, and lets applications evolve without redesigning routers. It also narrows debugging: DNS is not an Ethernet problem.

## How It Works

1. Application produces bytes.
2. Transport adds ports and TCP/UDP metadata: a segment or datagram.
3. Internet layer adds source/destination IP: a packet.
4. Link layer wraps it for the next hop: a frame.
5. At each routed hop, the old link frame is removed and a new one is created; the IP packet normally continues.

```text
Data
└─ TCP header + Data                = segment
   └─ IP header + segment           = packet
      └─ Ethernet header/trailer + packet = frame
         └─ bits on the medium
```

## Important Components / Terminology

| TCP/IP layer | Typical job | Examples |
|---|---|---|
| Application | App protocols | HTTP, DNS, TLS* |
| Transport | Process-to-process delivery | TCP, UDP, QUIC |
| Internet | Host addressing/routing | IP, ICMP |
| Link | Local-hop delivery | Ethernet, Wi-Fi, ARP |

`TLS` is often grouped with application in TCP/IP even though it provides security between application and transport.

## Example

`curl https://example.com` creates HTTP data; TLS encrypts it; TCP adds ports; IP adds addresses; Ethernet sends it to the gateway.

## Real-World Example

An L7 load balancer understands HTTP paths, while an L4 load balancer generally forwards TCP/UDP flows. The layer determines what it can inspect.

## Important Properties

| Model | Best use |
|---|---|
| OSI | Reasoning and teaching |
| TCP/IP | Mapping real Internet protocols |

### Common Misconceptions

* OSI is not a literal implementation blueprint.
* Routers often use several layers for management, filtering, or tunnelling.
* TCP carries a byte stream, not inherent application messages.

### What Interviewers Usually Test

Accurate encapsulation and the ability to identify where a failure belongs.

### Common Follow-up Questions

1. Which address changes at every routed hop?
2. Why are ports not link-layer addresses?
3. Where does TLS fit?
4. Which layers does a router use?
5. What is decapsulation?

### Follow-up Answers

1. The source and destination **MAC addresses** normally change because the router creates a new frame for the next local link.
2. Ports identify an application process at the transport layer; link-layer addresses identify the next-hop network interface.
3. TLS sits between application data and transport in practice; TCP/IP diagrams commonly group it with the application layer.
4. A router mainly forwards at Layer 3 but also receives/sends Layer-2 frames and may use higher layers for features such as firewalls or VPNs.
5. Decapsulation is removing lower-layer headers at the receiver so upper-layer data can be delivered to the application.
