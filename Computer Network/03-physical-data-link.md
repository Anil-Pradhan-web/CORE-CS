# Physical and Data-Link Layers

## What is it?

The physical layer sends raw signals as bits. The data-link layer delivers frames over one local link and detects many transmission errors.

## Simple Explanation

The physical layer is the road surface and traffic signal; the link layer is the local delivery label used from your computer to the next intersection.

## Hinglish Explanation

Physical layer actual cable, fiber, ya Wi-Fi radio signal hai. Data-link layer local network mein frame ko sahi next device tak pahunchata hai—Internet ke final server tak nahi.

## Why Do We Need It?

Hardware media have different signal rules. Link protocols turn unreliable signals into identifiable frames and coordinate local delivery.

## How It Works

1. A NIC converts bits to electrical, optical, or radio signals.
2. A sender builds a frame with destination/source MAC and a trailer such as FCS.
3. A switch forwards based on its learned MAC table; Wi-Fi uses its own medium-access rules.
4. Receiver verifies the frame; corrupt frames are usually discarded. Reliability may be added by Wi-Fi or higher layers.

```mermaid
flowchart LR
 A[NIC] --> B[Frame on local link]
 B --> C[Switch / Access Point]
 C --> D[Next-hop NIC]
```

## Important Components / Terminology

| Term | Meaning |
|---|---|
| NIC | Network interface controller/card |
| Frame | Link-layer container for one hop |
| MTU | Largest payload a link accepts without fragmentation at that layer |
| FCS/CRC | Error-detection value, not encryption |
| Duplex | One-way at a time or simultaneous send/receive |

## Example

Your laptop puts an IP packet inside a Wi-Fi frame addressed to the access point or local next hop.

## Real-World Example

Wrong VLAN membership can make two hosts appear connected but unable to reach one another. This is below DNS, HTTP, and TCP.

## Important Properties

| Property | Ethernet | Wi-Fi |
|---|---|---|
| Medium | Usually switched cable | Shared radio spectrum |
| Delivery | Local frame | Local wireless frame |
| Retries | Usually upper layers | Link retries commonly occur |

### Common Misconceptions

* A frame is not routed unchanged across the Internet.
* CRC detects many accidental errors; it is not a security guarantee.
* MAC addresses are not Internet-routable identities.

### What Interviewers Usually Test

Whether you understand local delivery, MTU, and the distinction between link and IP addressing.

### Common Follow-up Questions

1. Frame versus packet?
2. What is MTU?
3. How does a switch learn MAC addresses?
4. Why can Wi-Fi have higher latency?
5. What is a VLAN?

### Follow-up Answers

1. A frame delivers data on one local link; it usually carries an IP packet, which can be routed across networks.
2. MTU is the maximum payload size a link can carry in one frame without link-layer fragmentation.
3. A switch records the source MAC address, VLAN, and incoming port for frames it receives, then uses that table for later forwarding.
4. Wi-Fi shares radio spectrum and may wait for airtime, retransmit radio frames, or face interference, all of which add delay variation.
5. A VLAN is a logical Layer-2 network that separates broadcast domains on shared switching hardware.
