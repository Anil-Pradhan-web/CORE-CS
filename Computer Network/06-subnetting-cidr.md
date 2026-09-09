# Subnetting and CIDR

## What is it?

CIDR expresses an IP network as `address/prefix-length`, for example `10.0.4.0/22`. Subnetting divides an address range into smaller networks.

## Simple Explanation

A prefix is the neighbourhood name; remaining bits are house numbers. A longer prefix creates a smaller neighbourhood.

## Hinglish Explanation

`/24` ka matlab first 24 bits network ko identify karte hain. Prefix bada karoge, host bits kam honge aur subnet chhota hoga. Routing mein most specific matching prefix choose hota hai.

## Why Do We Need It?

Subnets limit broadcast domains, express routing boundaries, improve allocation, and support security segmentation. CIDR avoids wasteful classful address assumptions.

## How It Works

1. Convert prefix length into network/host bits.
2. Apply the mask to find the network address.
3. Determine the address range and usable host policy.
4. Routers select the matching route with the longest prefix.

```text
10.0.4.0/22
network bits: 22     host bits: 10
range: 10.0.4.0 – 10.0.7.255   (1024 total IPv4 addresses)
```

## Important Components / Terminology

| Term | Meaning |
|---|---|
| Prefix | Number of leading network bits |
| Mask | IPv4 representation such as `255.255.252.0` |
| Network address | First address of a conventional IPv4 subnet |
| Broadcast address | Last address in conventional IPv4 subnet; not used by IPv6 |
| Longest-prefix match | Most specific matching route wins |

## Example

`192.168.1.0/24` has 256 total addresses. In traditional IPv4 host assignment, `.0` is network and `.255` is broadcast, leaving 254 usable addresses; cloud providers may reserve additional addresses.

## Real-World Example

A VPC might allocate separate subnets for public load balancers, private application nodes, and databases. Route tables and security policy then enforce the intended paths.

## Important Properties

| Prefix | Total IPv4 addresses | Typical use |
|---|---:|---|
| /24 | 256 | Small LAN |
| /16 | 65,536 | Larger internal allocation |
| /32 | 1 | Single host route |

### Common Misconceptions

* `/24` does not always mean exactly 254 usable cloud instances.
* A subnet is not a security policy by itself.
* CIDR prefix does not tell you whether an address is public or private.

### What Interviewers Usually Test

Whether you can reason about address ranges and explain why route specificity matters.

### Common Follow-up Questions

1. How many addresses does `/27` contain?
2. Why is a `/32` useful?
3. What is route aggregation?
4. What wins: `/16` or `/24`?
5. Why subnet a VPC?

### Follow-up Answers

1. A `/27` has `2^(32−27) = 32` total IPv4 addresses; traditional host assignment reserves network and broadcast addresses.
2. A `/32` is a route for one exact IPv4 address, useful for a host route, exception, or precise policy.
3. Route aggregation advertises one broader prefix to represent several smaller networks, reducing routing-table size.
4. A matching `/24` wins because it is more specific than a matching `/16`.
5. VPC subnets separate tiers, control route paths, fit availability-zone design, and make security boundaries easier to apply.
