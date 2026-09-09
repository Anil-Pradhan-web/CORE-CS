# ARP, ICMP, and DHCP

## What is it?

ARP maps an IPv4 address to a local MAC address. ICMP carries IP control and diagnostic messages. DHCP leases network configuration to clients. IPv6 uses Neighbor Discovery rather than ARP.

## Simple Explanation

ARP asks “who has this local IP?” DHCP gives a new device its network settings, and ICMP is the network’s feedback channel.

## Hinglish Explanation

Naya laptop network join kare to DHCP se IP, gateway aur DNS mil sakta hai. Remote server ko packet bhejne se pehle ARP gateway ka MAC poochta hai. `ping` ICMP use karta hai, TCP nahi.

## Why Do We Need It?

Hosts need automatic setup, local next-hop resolution, and a way to report unreachable paths or expired TTLs.

## How It Works

1. DHCP client broadcasts discovery; server offers configuration; client requests; server acknowledges (DORA).
2. For an off-subnet destination, host ARPs for the gateway MAC and caches it.
3. Routers/hosts may send ICMP errors such as destination unreachable or time exceeded.

```text
DHCP: Discover → Offer → Request → ACK
ARP : “Who has 192.168.1.1?” → “192.168.1.1 is aa:bb:...”
```

## Important Components / Terminology

| Term | Meaning |
|---|---|
| Lease | Time-limited DHCP allocation |
| Default gateway | Router used for remote networks |
| ARP cache | Temporary IP-to-MAC mappings |
| Echo request/reply | ICMP messages used by ping |
| PMTUD | Path MTU Discovery, often depends on ICMP feedback |

## Example

Run `ipconfig /all` on Windows or `ip addr` and `ip route` on Linux to inspect DHCP-assigned address, gateway, and DNS. `arp -a` or `ip neigh` shows neighbour cache.

## Real-World Example

Blocking all ICMP can break useful diagnostics and sometimes Path MTU Discovery. Rate-limit and filter thoughtfully rather than blindly denying it.

## Important Properties

| Protocol | Primary role | Typical scope |
|---|---|---|
| ARP | IPv4-to-MAC resolution | Local LAN/VLAN |
| ICMP | IP control/diagnostics | End-to-end network |
| DHCP | Configuration lease | Local administrative network |

### Common Misconceptions

* ARP is not routed across the Internet.
* `ping` success does not prove HTTP is working.
* DHCP is not required for manually configured hosts.

### What Interviewers Usually Test

Local versus remote delivery and practical interpretation of `ping`, ARP cache, and DHCP failure.

### Common Follow-up Questions

1. Why ARP the gateway instead of the remote server?
2. What does a DHCP lease include?
3. Why can ping fail while a website works?
4. How does traceroute use TTL?
5. What replaces ARP in IPv6?

### Follow-up Answers

1. A remote server is not on the local link, so the host needs the local gateway's MAC address to send the first frame.
2. A lease commonly includes an IP address, subnet prefix or mask, default gateway, DNS servers, duration, and sometimes other local options.
3. ICMP echo may be blocked or rate-limited while TCP/QUIC and HTTPS are allowed and working.
4. It raises TTL/Hop Limit step by step so each router that expires a probe can reveal itself through an ICMP response.
5. IPv6 Neighbor Discovery Protocol uses ICMPv6 messages to resolve neighbours and routers.
