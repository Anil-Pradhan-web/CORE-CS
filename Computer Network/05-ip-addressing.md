# IP Addressing: IPv4 and IPv6

## What is it?

IP gives hosts logical addresses and lets routers forward packets across networks. IPv4 uses 32-bit addresses; IPv6 uses 128-bit addresses.

## Simple Explanation

IP is the postal address for the final destination. Routers read the destination address and pass the packet toward the next appropriate post office.

## Hinglish Explanation

IP address Internet-level location jaisa hai. Router destination IP dekhkar next hop choose karta hai. IPv4 addresses limited hain; IPv6 bahut large address space deta hai.

## Why Do We Need It?

Local MAC delivery cannot scale across millions of networks. IP provides hierarchical prefixes so routers store routes to networks, not a route for every device.

## How It Works

1. A host chooses source IP and destination IP.
2. It compares destination against its subnet mask/prefix.
3. Local destinations go directly; remote destinations go to the default gateway.
4. Routers decrement IPv4 TTL or IPv6 Hop Limit and forward using longest-prefix match.

## Important Components / Terminology

| Term | Meaning |
|---|---|
| IPv4 | 32-bit address, e.g. `192.0.2.10` |
| IPv6 | 128-bit address, e.g. `2001:db8::10` |
| Private IPv4 | RFC 1918 ranges used internally, not globally routed |
| Loopback | Local host address (`127.0.0.1`, `::1`) |
| TTL/Hop Limit | Loop-prevention counter decremented at routers |

## Example

An EC2 instance may have a private IPv4 for VPC traffic and reach the public Internet through a NAT gateway. An IPv6-enabled instance can receive globally routable IPv6 addressing if configured.

## Real-World Example

Containers commonly receive private addresses; service meshes and Kubernetes networking add virtual routes and policies around normal IP reachability.

## Important Properties

| Property | IPv4 | IPv6 |
|---|---|---|
| Address length | 32 bits | 128 bits |
| Broadcast | Has broadcast | No broadcast; multicast used |
| NAT reliance | Common | Not inherently required |
| Header | Variable options | Simpler base header + extensions |

### Common Misconceptions

* IPv6 is not simply “IPv4 with a longer address.”
* A private address is not automatically secure.
* IP provides best-effort delivery, not reliability or encryption.

### What Interviewers Usually Test

Address scope, default gateway behaviour, IPv4-vs-IPv6 trade-offs, and TTL.

### Common Follow-up Questions

1. What is private IP versus public IP?
2. What happens when TTL reaches zero?
3. Why no IPv6 broadcast?
4. Does IPv6 eliminate firewalls?
5. What is a default route?

### Follow-up Answers

1. Private IP ranges are for internal networks and are not normally routed on the public Internet; public IPs can be globally routed when policy permits.
2. The router drops the packet and commonly returns ICMP Time Exceeded, preventing routing loops from lasting forever.
3. IPv6 uses multicast and neighbour discovery instead of broadcasting every device on the local network, reducing unnecessary traffic.
4. No. IPv6 hosts still need firewalls, least-privilege rules, and secure service configuration.
5. A default route is the catch-all route used when no more specific route matches a destination.
