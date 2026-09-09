# Ethernet and MAC Addresses

## What is it?

Ethernet is a common LAN technology. A MAC address identifies a network interface on a local link; switches use it to forward frames.

## Simple Explanation

MAC is like a label used inside one building; IP is the postal address used across cities. The same parcel gets a new local label at each building entrance.

## Hinglish Explanation

MAC address local LAN ka address hai. Switch MAC table dekhkar frame ko correct port par bhejta hai. Router cross karte hi next link ke liye naya MAC header banega.

## Why Do We Need It?

LAN devices need a next-hop destination before IP routing can continue. Ethernet standardizes framing and local delivery.

## How It Works

1. Sender needs the next-hop MAC (often the gateway) through ARP/Neighbor Discovery.
2. It sends a frame with destination and source MAC.
3. The switch learns the source MAC-to-port mapping.
4. If destination is known, it forwards to that port; otherwise it floods within the VLAN.

```text
Host A -- frame dst: gateway-MAC --> Switch -- one port --> Router
         packet dst: web-server-IP remains inside the frame
```

## Important Components / Terminology

| Term | Meaning |
|---|---|
| MAC address | Usually 48-bit interface identifier in Ethernet |
| CAM/MAC table | Switch mapping from MAC to port/VLAN |
| Broadcast | Frame delivered to all ports in a broadcast domain |
| VLAN | Logical Layer-2 segmentation |

## Example

Two VMs in one VLAN can exchange Ethernet frames directly; VMs in different subnets typically need a router, even if located on the same physical switch.

## Real-World Example

Cloud networks present virtual NICs, switches, security controls, and MAC learning behaviour even though physical switching is hidden.

## Important Properties

| Address | Scope | Used by |
|---|---|---|
| MAC | Local link/VLAN | NICs and switches |
| IP | Routed network | Hosts and routers |

### Common Misconceptions

* MAC addresses do not identify a device globally in an operational sense; they can be changed or virtualized.
* Switches do not use IP routing tables for normal frame forwarding.
* A router does not forward Ethernet broadcast frames by default.

### What Interviewers Usually Test

Why the destination MAC is a gateway while the destination IP is a remote server.

### Common Follow-up Questions

1. What happens for an unknown unicast frame?
2. Switch versus hub?
3. What is a broadcast domain?
4. Why does a MAC change across a router?
5. What does VLAN tagging solve?

### Follow-up Answers

1. The switch floods it to relevant ports in the same VLAN, except the port where it arrived, until it learns the destination location.
2. A hub repeats traffic to all ports; a switch learns MAC-to-port mappings and sends known unicast traffic only where needed.
3. A broadcast domain is the set of devices that receive a Layer-2 broadcast; routers normally separate these domains.
4. The router removes the old local frame and builds a new frame addressed to the next hop, so link-layer addresses change.
5. VLAN tagging lets one trunk link carry traffic from multiple isolated logical LANs while preserving the VLAN identity of each frame.
