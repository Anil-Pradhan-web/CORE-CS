# Network Troubleshooting

## What is it?

Network troubleshooting is a controlled method for locating which dependency or layer is failing. Start with a precise symptom, test assumptions one at a time, and compare known-good paths.

## Simple Explanation

Debug a network like a pipeline: verify name, route, reachability, port, TLS, then application response. Do not jump from “website failed” to one guessed cause.

## Hinglish Explanation

Pehle exact error note karo: DNS failure, timeout, refused, TLS error, ya HTTP 5xx. Phir layer-by-layer evidence collect karo. Ek machine works aur doosri nahi—routes, DNS, proxy, firewall, certificates, ya credentials differ ho sakte hain.

## Why Do We Need It?

Similar user symptoms have different causes. A systematic approach shortens outages and avoids destructive changes.

## How It Works

1. Capture hostname, time, client, destination, exact error, and recent changes.
2. Verify local config: IP, route, gateway, proxy/VPN.
3. Resolve DNS and compare expected addresses.
4. Test TCP/UDP reachability and TLS separately from HTTP semantics.
5. Inspect server listener, firewall/load-balancer logs, packet capture, and metrics.

```text
Name?  dig/nslookup
Route? ip route / traceroute
Port?  curl -v / Test-NetConnection / nc
TLS?   openssl s_client
HTTP?  curl -i
Host?  ss -lntp, logs, tcpdump
```

## Important Components / Terminology

| Symptom | Likely meaning (not proof) |
|---|---|
| DNS `NXDOMAIN` | Name absent at resolver/authority |
| Connection refused | Reachable endpoint rejected TCP, often no listener |
| Timeout | Drops, routing issue, overload, or no response |
| TLS certificate error | Trust/name/date/chain/policy issue |
| HTTP 502/503 | Gateway/upstream availability problem |

## Example

`curl -v --connect-timeout 5 https://api.example.com/health` exposes resolved IP, connection attempt, TLS, headers, and response. On Windows, `Test-NetConnection host -Port 443` is a useful initial port check.

## Real-World Example

One client fails while another succeeds: compare DNS answer, source network, VPN/proxy, IPv4 vs IPv6 preference, TLS clock/trust store, authentication, and feature flags before changing server-wide configuration.

## Important Properties

### Production Troubleshooting Commands Cheatsheet

| Layer | Tool | Production Command | Diagnostic Intent |
|---|---|---|---|
| **DNS** | `dig` | `dig +trace +nodnssec example.com A` | Trace full DNS delegation chain to find stale records |
| **Routing / IP** | `mtr` | `mtr --report --report-cycles 10 1.1.1.1` | Combined ping + traceroute showing packet loss per hop |
| **Port / TCP** | `nc` | `nc -zv -w 3 192.168.1.50 443` | Quick TCP handshake connectivity check |
| **Sockets** | `ss` | `ss -tulpn` | Inspect open listeners, backlog queue drops, and PIDs |
| **TLS** | `openssl` | `openssl s_client -connect host:443 -servername host` | Inspect certificate expiry, SAN, and SNI handshake |
| **Packets** | `tcpdump` | `tcpdump -nn -i any 'tcp port 443 and tcp[tcpflags] & (tcp-syn\|tcp-rst) != 0'` | Capture SYN and RST packets to detect drops or rejections |

### The `curl` Latency Breakdown Recipe

Create `curl-format.txt`:
```text
  time_namelookup:    %{time_namelookup}s\n
  time_connect:       %{time_connect}s\n
  time_appconnect:    %{time_appconnect}s (TLS)\n
  time_pretransfer:   %{time_pretransfer}s\n
  time_starttransfer: %{time_starttransfer}s (TTFB)\n
  time_total:         %{time_total}s\n
```
Run: `curl -w "@curl-format.txt" -o /dev/null -s https://api.example.com`
- If `time_namelookup` is high: DNS issue.
- If `time_connect` - `time_namelookup` is high: Network routing / TCP RTT issue.
- If `time_appconnect` - `time_connect` is high: TLS negotiation / cipher overhead.
- If `time_starttransfer` - `time_appconnect` is high: Slow backend server execution (TTFB).

### Common Misconceptions

* A successful ping does not mean port 443 is reachable.
* A failed traceroute hop may simply filter ICMP.
* HTTP 500/503 is not automatically a network problem.

### What Interviewers Usually Test

The order of checks, interpretation of symptoms, evidence-driven isolation, and distinction between client/network/server/application failure.

### Common Follow-up Questions

1. Timeout versus connection refused?
2. Why does DNS work but HTTPS fail?
3. What does a packet capture prove?
4. How troubleshoot one-client-only failure?
5. How investigate high latency?

### Follow-up Answers

1. Refused usually means a reachable TCP stack actively rejected the port; timeout means no successful response arrived before the deadline and has several possible causes.
2. DNS only returns an address. The connection can still fail because of routing, port policy, no listener, TLS validation, or application failure.
3. A capture shows packets seen at that observation point; it does not by itself prove what happened before or after a different network hop.
4. Compare exact DNS result, IP family, route, proxy/VPN, firewall policy, TLS trust/time, request headers, and credentials with a working client.
5. Break the request into DNS, connect, TLS, server, and download phases; correlate with RTT, loss/retransmissions, queues, traces, and dependency metrics.
