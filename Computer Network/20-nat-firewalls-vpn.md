# NAT, Firewalls, and VPNs

## What is it?

NAT rewrites addresses and sometimes ports between networks. Firewalls enforce traffic policy. A VPN creates an authenticated encrypted overlay/tunnel over another network.

## Simple Explanation

NAT is a building receptionist who substitutes one public return address for many apartments. A firewall decides who may enter. A VPN is a protected private corridor through public roads.

## Hinglish Explanation

NAT private IP ko public IP/port se map kar sakta hai, isliye many hosts ek public IPv4 share karte hain. Firewall rule traffic allow/deny karta hai. VPN encryption aur routing tunnel provide karta hai, but policy still required hai.

## Why Do We Need It?

IPv4 conservation and network separation encouraged NAT; firewalls reduce exposed attack surface; VPNs safely join users/sites over untrusted networks.

## How It Works

1. Outbound host sends `10.0.0.5:51500` to Internet.
2. NAT creates state, rewrites source to public `198.51.100.7:40001`.
3. Return traffic matching that state is translated back.
4. Firewall evaluates rules/state; VPN encapsulates inner traffic in encrypted outer packets.

```text
10.0.0.5:51500 → [NAT] 198.51.100.7:40001 → Internet
                ← stateful reverse mapping ←
```

## Important Components / Terminology

| Term | Meaning |
|---|---|
| SNAT | Source address/port translation, commonly outbound |
| DNAT/port forward | Destination translation to internal service |
| Stateful firewall | Allows related return traffic based on tracked flow |
| Split tunnel | Only selected routes use VPN |
| Full tunnel | Default traffic uses VPN |

## Example

A developer reaches `10.20.0.0/16` only after connecting to a corporate VPN. If DNS resolves but TCP times out, inspect VPN routes, security groups/firewall, and service listener—not just the VPN client.

## Real-World Example

Kubernetes and cloud NAT gateways have connection/port limits. High fan-out services should reuse connections and monitor NAT table/port exhaustion.

## Important Properties

| Technology | Solves | Does not inherently solve |
|---|---|---|
| NAT | Address translation | Authentication, encryption |
| Firewall | Traffic policy | Application correctness |
| VPN | Protected connectivity | Least-privilege access by default |

### Common Misconceptions

* NAT is not a firewall, although it can make unsolicited inbound traffic harder.
* VPN does not magically make an untrusted endpoint safe.
* Private IP space is not automatically encrypted.

### What Interviewers Usually Test

Address/port translation state, inbound reachability, stateful filtering, and cloud-network consequences.

### Common Follow-up Questions

1. SNAT versus DNAT?
2. Why does NAT complicate peer-to-peer?
3. Stateful versus stateless firewall?
4. Split versus full tunnel?
5. What causes NAT port exhaustion?

### Follow-up Answers

1. SNAT rewrites a packet’s source address/port, commonly for outbound traffic; DNAT rewrites its destination, commonly to publish an internal service.
2. Hosts behind NAT often lack a stable publicly reachable address/port, so unsolicited inbound connections need traversal, forwarding, or relay techniques.
3. A stateful firewall remembers permitted flows and allows matching return traffic; a stateless firewall evaluates each packet against independent rules.
4. Split tunnel routes selected destinations through VPN; full tunnel routes the default Internet path through VPN.
5. Too many concurrent mappings, high connection churn, too few public addresses, or limited NAT table capacity can exhaust available mappings.
