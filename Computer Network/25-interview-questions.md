# 100 Most Frequently Asked Computer Networks Interview Questions

Each answer is designed to be spoken: definition first, then why/how, then a practical caveat.

## Beginner — 1–30

### 1. What is a computer network?

**Short Answer:** A computer network connects independent devices so they can exchange data using common protocols. The Internet is a network of networks, not one single machine.

**Detailed Explanation:** Data is packaged and moved across local links and routed networks. Protocol layers let applications communicate without knowing the hardware path.

> **Interview Tip:** Mention data, protocols, and routing—not only cables.

**Follow-up:** What is a protocol? What is the difference between LAN and WAN?

**Follow-up Answers:** A **protocol** is an agreed data format and behaviour for communication. A **LAN** covers a limited local area; a **WAN** links networks across larger distances.

### 2. What is the difference between bandwidth, throughput, and latency?

**Short Answer:** Bandwidth is potential link capacity; throughput is the useful rate actually achieved; latency is delay. A high-bandwidth path can still have high latency.

**Detailed Explanation:** Throughput is constrained by bottlenecks, protocol windows, loss, and application processing. Latency includes propagation, transmission, queueing, and processing delay.

> **Interview Tip:** Use a wide highway versus travel-time analogy.

**Follow-up:** What is RTT? What is jitter?

**Follow-up Answers:** **RTT** is the time for a message to go to a peer and for a response to return. **Jitter** is variation in delay between packets.

### 3. What is the OSI model?

**Short Answer:** OSI is a seven-layer conceptual model: Physical, Data Link, Network, Transport, Session, Presentation, and Application. It helps reason about responsibilities and failures.

**Detailed Explanation:** Real Internet protocols do not always fit one layer exactly, so OSI is best used as a diagnostic and communication model rather than a literal architecture.

> **Interview Tip:** State that it is conceptual; that nuance prevents a textbook-only answer.

**Follow-up:** How does it map to TCP/IP? Which layer is a router?

**Follow-up Answers:** OSI Application/Presentation/Session map roughly to TCP/IP Application; Transport maps to Transport; Network maps to Internet; Data Link/Physical map to Link. A router mainly forwards at OSI Layer 3.

### 4. What is the TCP/IP model?

**Short Answer:** TCP/IP is the practical Internet model, commonly grouped as Application, Transport, Internet, and Link layers. It describes how Internet protocols cooperate.

**Detailed Explanation:** HTTP/DNS are application protocols, TCP/UDP provide transport, IP routes between networks, and Ethernet/Wi-Fi deliver to a local next hop.

> **Interview Tip:** Do not say TCP/IP means only TCP and IP.

**Follow-up:** Where does TLS fit? What does each layer add?

**Follow-up Answers:** TLS is commonly placed between application and transport, or grouped with TCP/IP Application. Layers add app semantics, ports, IP addressing, then local-link framing/signals.

### 5. What is encapsulation?

**Short Answer:** Encapsulation adds each layer’s header around upper-layer data before transmission. Data becomes a TCP segment, IP packet, then link-layer frame.

**Detailed Explanation:** At a routed hop, the router removes the incoming link frame and creates a new frame for the next link; the end-to-end IP and transport information normally remains.

> **Interview Tip:** Explain which address changes at a router: usually MAC, not destination IP.

**Follow-up:** Frame versus packet versus segment? What is decapsulation?

**Follow-up Answers:** A frame is local-link data, a packet is an IP unit, and a segment is TCP data. Decapsulation removes received headers to expose upper-layer data.

### 6. What is an IP address?

**Short Answer:** An IP address is a logical network-layer address used to identify and route toward a host/interface across networks. It is different from a local MAC address.

**Detailed Explanation:** Routers use destination IP prefixes to select next hops. IP itself is best effort; reliability, ordering, and encryption are supplied elsewhere when needed.

> **Interview Tip:** Say “interface/addressable endpoint” rather than treating IP as a permanent physical machine identity.

**Follow-up:** Private versus public IP? IPv4 versus IPv6?

**Follow-up Answers:** Private IPs are for internal use and are not normally publicly routed; public IPs may be. IPv4 has 32-bit addresses; IPv6 has 128-bit addresses and different protocol design.

### 7. What is a MAC address?

**Short Answer:** A MAC address is a link-layer identifier used for local Ethernet/Wi-Fi delivery. Switches use MAC addresses to forward frames inside a LAN or VLAN.

**Detailed Explanation:** A host sending to a remote IP normally puts the gateway’s MAC in the Ethernet destination field, then the gateway builds a new frame on the next link.

> **Interview Tip:** MAC is local-hop addressing; it is not the remote Internet destination’s address.

**Follow-up:** How does a switch learn MACs? What is ARP?

**Follow-up Answers:** A switch learns by recording source MAC, VLAN, and ingress port. ARP asks for the local MAC address that owns an IPv4 address.

### 8. What is the difference between a hub, switch, and router?

**Short Answer:** A hub repeats signals to all ports; a switch forwards frames based on MAC tables; a router forwards packets between IP networks using routing tables.

**Detailed Explanation:** Modern devices may combine features, but this distinction captures their primary forwarding decision and broadcast-domain effect.

> **Interview Tip:** Avoid claiming real devices operate at only one layer.

**Follow-up:** What does a Layer-3 switch do? What is a VLAN?

**Follow-up Answers:** A Layer-3 switch can route between IP networks as well as switch frames. A VLAN is a logical Layer-2 broadcast domain on shared switch hardware.

### 9. What is a port number?

**Short Answer:** A port is a 16-bit transport-layer number that directs TCP or UDP traffic to the right process on a host. IP identifies host reachability; port identifies a service endpoint.

**Detailed Explanation:** A connection is commonly distinguished by protocol plus source/destination IP addresses and ports. Clients generally use ephemeral source ports.

> **Interview Tip:** TCP 443 and UDP 443 are separate namespaces.

**Follow-up:** What is an ephemeral port? Can two programs use one port?

**Follow-up Answers:** An ephemeral port is a temporary client source port chosen by the OS. Two programs usually cannot bind the same IP/protocol/port, except with controlled OS reuse features or different addresses.

### 10. What is a socket?

**Short Answer:** A socket is an operating-system communication endpoint exposed through APIs such as bind, listen, accept, connect, send, and receive. It is often described using IP, port, and protocol context.

**Detailed Explanation:** A listening TCP socket accepts many established connections; each accepted connection has its own peer tuple and state.

> **Interview Tip:** Distinguish a listener from an established connection.

**Follow-up:** What do `bind()` and `connect()` do? What is the 5-tuple?

**Follow-up Answers:** `bind()` chooses a local address/port; `connect()` selects a remote peer and initiates communication. The 5-tuple is protocol plus source/destination IP addresses and ports.

### 11. What is DNS?

**Short Answer:** DNS is a distributed naming system that maps domain names to records such as IP addresses. Clients usually query a recursive resolver, which caches answers.

**Detailed Explanation:** On a cache miss, a resolver follows delegation from root to TLD to authoritative nameserver. DNS supports more than web IP lookup: aliases, mail records, and service records exist too.

> **Interview Tip:** DNS resolution does not prove the target application is healthy.

**Follow-up:** Recursive versus authoritative resolver? What is TTL?

**Follow-up Answers:** A recursive resolver finds and caches answers for clients; an authoritative server is the source for a zone’s records. TTL is the maximum intended cache lifetime.

### 12. Does DNS use TCP or UDP?

**Short Answer:** Both. Ordinary queries commonly use UDP, while TCP is used for cases such as large/truncated responses, zone transfers, and fallback; encrypted DNS variants add other transports.

**Detailed Explanation:** The exact transport depends on resolver, response size, EDNS support, policy, and protocol variant such as DNS-over-TLS or DNS-over-HTTPS.

> **Interview Tip:** “DNS uses UDP” without qualification is an interview trap.

**Follow-up:** Why is UDP common? What is DNS over HTTPS?

**Follow-up Answers:** UDP avoids connection setup for small ordinary queries. DNS over HTTPS carries DNS queries inside HTTPS, protecting them from many local observers.

### 13. What is DHCP?

**Short Answer:** DHCP automatically leases configuration such as IP address, subnet prefix, default gateway, and DNS servers to a client. It prevents manual configuration for most managed networks.

**Detailed Explanation:** IPv4’s common DORA sequence is Discover, Offer, Request, Acknowledge. Leases expire or renew, so configuration is not necessarily permanent.

> **Interview Tip:** Name at least gateway and DNS besides IP address.

**Follow-up:** What is a DHCP lease? Can a host use static IP?

**Follow-up Answers:** A DHCP lease is time-limited assigned configuration. Yes, a host can use static IP when the network plan and conflict controls allow it.

### 14. What is ARP?

**Short Answer:** ARP resolves an IPv4 address to a MAC address on the local link. For remote destinations, a host typically resolves the default gateway’s MAC, not the remote server’s MAC.

**Detailed Explanation:** A host broadcasts an ARP request within its broadcast domain and caches the reply. IPv6 uses Neighbor Discovery instead of ARP.

> **Interview Tip:** ARP is not routed across the Internet.

**Follow-up:** Why can ARP spoofing be dangerous? What replaces ARP in IPv6?

**Follow-up Answers:** ARP spoofing can redirect local traffic through an attacker. IPv6 uses Neighbor Discovery Protocol, based on ICMPv6, for neighbour resolution.

### 15. What is ICMP?

**Short Answer:** ICMP is an IP control and diagnostic protocol. It carries messages such as echo request/reply, destination unreachable, and time exceeded.

**Detailed Explanation:** `ping` uses ICMP echo messages, and traceroute relies on TTL/Hop Limit expiry and ICMP responses. ICMP is useful but may be filtered or rate limited.

> **Interview Tip:** Ping success is not proof that TCP port 443 works.

**Follow-up:** How does traceroute work? Why can ICMP be blocked?

**Follow-up Answers:** Traceroute raises TTL/Hop Limit so routers send time-exceeded replies. ICMP can be filtered or rate-limited to reduce abuse or operational exposure.

### 16. What is a subnet?

**Short Answer:** A subnet is a range of IP addresses sharing a network prefix and routing boundary. It groups hosts for address allocation and often local link or policy design.

**Detailed Explanation:** A host compares destination IP with its configured prefix; off-subnet traffic normally goes to the default gateway.

> **Interview Tip:** A subnet alone is not a firewall or security boundary.

**Follow-up:** Why split a VPC into subnets? What is a default gateway?

**Follow-up Answers:** Separate subnets organize tiers, availability zones, and route/policy boundaries. A default gateway is the router used for destinations without a more specific local route.

### 17. What does CIDR notation mean?

**Short Answer:** CIDR writes an address and prefix length, such as `10.0.0.0/16`. The prefix says how many leading bits identify the network; remaining bits identify addresses within it.

**Detailed Explanation:** Longer prefixes make smaller networks. Routers choose the most specific matching route, called longest-prefix matching.

> **Interview Tip:** `/24` means 256 total IPv4 addresses, before provider reservations or host-usage conventions.

**Follow-up:** How many addresses in `/27`? Why use `/32`?

**Follow-up Answers:** A `/27` has 32 total IPv4 addresses. A `/32` specifies one exact IPv4 host route or policy target.

### 18. What is a default gateway?

**Short Answer:** A default gateway is the router a host sends traffic to when no more specific local route matches the destination. It is the host’s next hop, not necessarily the final Internet route.

**Detailed Explanation:** The host resolves the gateway’s link-layer address and sends a frame to it; the IP destination remains the remote host.

> **Interview Tip:** A gateway is selected by the routing table, not DNS.

**Follow-up:** What happens if the gateway is wrong? What is `0.0.0.0/0`?

**Follow-up Answers:** A wrong gateway prevents or misroutes off-subnet traffic. `0.0.0.0/0` is the IPv4 default route that matches every address when no specific route wins.

### 19. What is routing?

**Short Answer:** Routing selects the next hop for an IP packet based on destination prefix and policy. Each router makes a local forwarding decision.

**Detailed Explanation:** Forwarding tables can be static or learned through routing protocols. The most specific route normally wins before metrics and other policy considerations.

> **Interview Tip:** Explain routing as next-hop selection, not “finding every router at once.”

**Follow-up:** What is longest-prefix match? What is a routing loop?

**Follow-up Answers:** Longest-prefix match chooses the most specific matching route. A routing loop is repeated forwarding among routers until TTL/Hop Limit stops the packet.

### 20. What is NAT?

**Short Answer:** Network Address Translation rewrites addresses and often ports between network realms. Source NAT lets many private IPv4 hosts share a public address through per-flow mapping state.

**Detailed Explanation:** Return packets are translated using the mapping. NAT conserves IPv4 space but complicates unsolicited inbound access and some peer-to-peer protocols.

> **Interview Tip:** NAT is not encryption and is not equivalent to a firewall.

**Follow-up:** SNAT versus DNAT? Why can NAT exhaust ports?

**Follow-up Answers:** SNAT rewrites source address/port, while DNAT rewrites destination address/port. NAT can run out of mapping ports or table capacity under high concurrent outbound flows.

### 21. What is a firewall?

**Short Answer:** A firewall enforces network traffic policy, usually by allowing or denying flows based on addresses, ports, protocol, identity, state, or application data.

**Detailed Explanation:** Stateful firewalls track established flows and permit related returns; stateless rules examine each packet without that connection context.

> **Interview Tip:** A firewall rule allowing a port does not create a server listener.

**Follow-up:** Stateful versus stateless? Security group versus network ACL?

**Follow-up Answers:** Stateful firewalls remember allowed flows and permit matching replies; stateless rules check each packet. A security group is usually stateful instance/interface policy; a network ACL is commonly stateless subnet policy.

### 22. What is TCP?

**Short Answer:** TCP is a connection-oriented transport protocol providing reliable, ordered delivery of a byte stream using acknowledgements, sequence numbers, retransmissions, and flow/congestion control.

**Detailed Explanation:** TCP hides loss and reordering from the application when recovery succeeds, but it does not preserve application message boundaries or provide encryption.

> **Interview Tip:** Say “byte stream,” not “reliable packets.”

**Follow-up:** How does TCP recover loss? Does TCP encrypt?

**Follow-up Answers:** TCP uses acknowledgements, timeouts, and retransmission to recover missing bytes. TCP does not encrypt; TLS is commonly added for protection.

### 23. What is UDP?

**Short Answer:** UDP is a connectionless datagram transport protocol. It supplies ports and a checksum but no built-in reliability, ordering, or retransmission comparable to TCP.

**Detailed Explanation:** Each delivered UDP datagram retains a message boundary. Applications may add their own acknowledgements, ordering, and congestion handling when necessary.

> **Interview Tip:** UDP is not automatically faster; it moves reliability choices to the application.

**Follow-up:** Why use UDP for DNS? Can UDP be reliable?

**Follow-up Answers:** UDP keeps typical DNS queries lightweight and fast to start. Applications can add sequence numbers, ACKs, retransmissions, and rate control to build reliability over UDP.

### 24. Compare TCP and UDP.

**Short Answer:** TCP is reliable, ordered, connection-oriented byte-stream transport; UDP is connectionless message/datagram transport with minimal built-in delivery guarantees. Choose based on semantics, not a blanket speed rule.

**Detailed Explanation:** TCP is common for web and databases; UDP suits datagram-oriented/freshness-sensitive traffic and supports QUIC. Both still depend on IP and can be filtered or delayed.

> **Interview Tip:** Mention head-of-line blocking and application complexity.

**Follow-up:** TCP versus UDP for live video? Why does QUIC use UDP?

**Follow-up Answers:** Live video often prefers UDP-based delivery because late media is less useful than fresh media. QUIC uses UDP as a deployable substrate while implementing its own secure reliable streams.

### 25. What is HTTP?

**Short Answer:** HTTP is an application protocol for request-response communication. It defines methods, paths, headers, bodies, status codes, caching, and semantic expectations.

**Detailed Explanation:** HTTP can run over different transports by version. It does not itself encrypt traffic; HTTPS means HTTP over TLS.

> **Interview Tip:** Separate HTTP semantics from TCP/TLS transport security.

**Follow-up:** What makes a request idempotent? GET versus POST?

**Follow-up Answers:** An operation is idempotent if repeating it has the intended same effect. GET retrieves data and should be safe; POST submits work and is not automatically idempotent.

### 26. What is HTTPS?

**Short Answer:** HTTPS is HTTP carried over TLS. TLS provides encryption, integrity checks, and usually server authentication after certificate validation.

**Detailed Explanation:** The browser validates hostname, certificate chain, time, and trust policy before relying on the negotiated secure channel.

> **Interview Tip:** HTTPS is not a separate transport protocol.

**Follow-up:** What is a certificate? What is TLS termination?

**Follow-up Answers:** A certificate binds a public key to an identity assertion. TLS termination decrypts TLS at a proxy/load balancer before it forwards traffic.

### 27. What is TLS?

**Short Answer:** TLS is a cryptographic protocol that negotiates protected session keys and authenticates peers, usually the server. It is commonly used beneath HTTPS.

**Detailed Explanation:** Public-key cryptography and certificates establish identity/key agreement; efficient symmetric cryptography protects subsequent application data.

> **Interview Tip:** Authentication is as important as encryption; otherwise an attacker could terminate a tunnel.

**Follow-up:** What is forward secrecy? What is mTLS?

**Follow-up Answers:** Forward secrecy means later compromise of a long-term key does not normally expose recorded old sessions. mTLS has both client and server authenticate with certificates.

### 28. What is a proxy versus a reverse proxy?

**Short Answer:** A forward proxy represents clients when they access external services; a reverse proxy represents servers when clients access a service. Both can enforce policy and relay traffic.

**Detailed Explanation:** Reverse proxies commonly handle TLS termination, routing, compression, caching, rate limits, and upstream health; forward proxies commonly control client egress.

> **Interview Tip:** Define them from whose perspective they serve.

**Follow-up:** Is a load balancer a reverse proxy? What is CONNECT?

**Follow-up Answers:** An L7 load balancer often acts as a reverse proxy, but the terms are not identical. HTTP CONNECT asks a proxy to create a tunnel to a target, commonly for HTTPS.

### 29. What is a load balancer?

**Short Answer:** A load balancer distributes traffic among healthy targets to improve availability and capacity. It can operate at L4 for TCP/UDP or L7 for HTTP-aware routing.

**Detailed Explanation:** Algorithms include round robin, least connections, weighted rules, and consistent hashing. Health checks must reflect meaningful service readiness.

> **Interview Tip:** Balancing traffic does not repair a shared failing dependency.

**Follow-up:** L4 versus L7? Why use sticky sessions?

**Follow-up Answers:** L4 uses flow metadata; L7 can route using application data such as HTTP paths. Sticky sessions are used when state remains local to one backend, though shared state is usually more scalable.

### 30. What is a CDN?

**Short Answer:** A Content Delivery Network serves cacheable content from edge locations near users and can provide security and traffic absorption. On a cache miss it fetches from an origin.

**Detailed Explanation:** CDN behaviour is driven by DNS/anycast, cache keys, TTLs, headers, invalidation, and routing policy. Dynamic traffic can also be proxied through an edge.

> **Interview Tip:** State that caching personalized responses requires careful cache keys and controls.

**Follow-up:** What is cache invalidation? CDN versus load balancer?

**Follow-up Answers:** Cache invalidation removes or marks stale cached objects. A CDN serves/caches traffic near users; a load balancer distributes traffic among service targets.

## Intermediate — 31–70

### 31. Explain the TCP three-way handshake.

**Short Answer:** Client sends SYN, server replies SYN-ACK, and client sends ACK. The exchange establishes both directions and synchronizes initial sequence-number state.

**Detailed Explanation:** The final ACK confirms that the client received the server’s sequence number. This avoids ambiguity from stale or delayed connection requests better than a two-message exchange.

> **Interview Tip:** Name the directions and what each acknowledgement confirms.

**Follow-up:** Why not two-way? What if SYN-ACK is lost?

**Follow-up Answers:** Two-way does not confirm that the client received the server’s sequence information. If SYN-ACK is lost, the server normally retransmits it and the client may retransmit SYN.

### 32. Why does TCP need a three-way handshake instead of two-way?

**Short Answer:** Both endpoints need confirmation that the other can receive and that each initial sequence number is known. Two messages leave the server uncertain whether the client received its sequence information.

**Detailed Explanation:** The final ACK also reduces problems from old duplicate SYNs creating unintended state. It is about synchronizing bidirectional connection state, not merely saying hello.

> **Interview Tip:** Avoid claiming a handshake proves the application is healthy.

**Follow-up:** What role do initial sequence numbers play? What are SYN cookies?

**Follow-up Answers:** Initial sequence numbers establish each side’s byte-space and reduce confusion from old traffic. SYN cookies encode limited state in the SYN-ACK to resist SYN floods.

### 33. How does TCP terminate a connection?

**Short Answer:** Each direction closes independently: one endpoint sends FIN, peer ACKs it, then peer sends its own FIN when done, and the first endpoint ACKs. RST is an abrupt alternative.

**Detailed Explanation:** TCP supports half-close because an endpoint may finish sending but still receive data. The active closer commonly enters TIME_WAIT after the final ACK.

> **Interview Tip:** FIN and RST have very different semantics.

**Follow-up:** Why TIME_WAIT? What is half-close?

**Follow-up Answers:** TIME_WAIT lets the active closer handle delayed duplicates and retransmit the last ACK. Half-close means one endpoint has finished sending but may still receive data.

### 34. What is TIME_WAIT and why does it exist?

**Short Answer:** TIME_WAIT is a TCP state usually entered by the active closer after final ACK. It helps retransmit that ACK if needed and lets old duplicate segments expire before tuple reuse.

**Detailed Explanation:** It is normal during high connection churn. Excessive accumulation may reveal short-lived-connection patterns or resource constraints, but blindly removing it is unsafe.

> **Interview Tip:** Do not call TIME_WAIT a leak by default.

**Follow-up:** Which side enters it? How can connection reuse reduce churn?

**Follow-up Answers:** The endpoint that actively initiates close usually enters TIME_WAIT. Reusing persistent connections avoids frequent handshakes and closes.

### 35. What are TCP sequence numbers and acknowledgements?

**Short Answer:** Sequence numbers identify byte positions in TCP’s stream; acknowledgements usually state the next byte expected. They enable ordering, duplicate detection, and loss recovery.

**Detailed Explanation:** TCP ACKs are generally cumulative, though extensions such as SACK can report additional noncontiguous received blocks to improve recovery.

> **Interview Tip:** ACK number means next expected byte, not simply “I got one packet.”

**Follow-up:** What is SACK? Why are numbers randomized?

**Follow-up Answers:** SACK tells the sender which non-contiguous byte blocks arrived, improving loss recovery. Randomized initial sequence numbers make old/injected traffic harder to confuse with a new connection.

### 36. How does TCP detect and recover packet loss?

**Short Answer:** TCP infers loss from retransmission timeout or patterns such as duplicate acknowledgements, then retransmits missing bytes and reduces sending pressure according to congestion control.

**Detailed Explanation:** It cannot directly know every dropped packet. SACK lets receiver identify received blocks so sender can avoid retransmitting unnecessarily.

> **Interview Tip:** Loss can result from congestion, wireless errors, filtering, or device failure.

**Follow-up:** What is fast retransmit? What is RTO?

**Follow-up Answers:** Fast retransmit resends suspected missing data after duplicate ACK evidence without waiting for timeout. RTO is the retransmission timeout based on observed round-trip behaviour.

### 37. What is the sliding window protocol?

**Short Answer:** Sliding windows allow multiple bytes or packets to be sent before waiting for acknowledgements. As ACKs arrive, the send window advances and new data can enter.

**Detailed Explanation:** TCP’s effective sending window is constrained by receiver-advertised flow-control window and sender congestion window, minus bytes already in flight.

> **Interview Tip:** Windowing improves utilization on nonzero-RTT paths.

**Follow-up:** What is bandwidth-delay product? rwnd versus cwnd?

**Follow-up Answers:** Bandwidth-delay product is bandwidth times RTT and estimates needed in-flight data. `rwnd` protects receiver buffers; `cwnd` protects the network from congestion.

### 38. What is flow control in TCP?

**Short Answer:** Flow control protects a receiver from being overrun by a faster sender. The receiver advertises available buffer space as its receive window (`rwnd`).

**Detailed Explanation:** If `rwnd` is small or zero, a sender must limit data even if the network has spare bandwidth. Window updates resume sending as the application consumes data.

> **Interview Tip:** Flow control protects the endpoint, not the shared network.

**Follow-up:** What is a zero window? How is it different from congestion control?

**Follow-up Answers:** A zero window means receiver has no buffer space and asks sender to pause. Congestion control slows traffic because the network, not the receiver, appears overloaded.

### 39. What is congestion control in TCP?

**Short Answer:** Congestion control regulates sending to avoid overwhelming network paths and queues. TCP uses a congestion window (`cwnd`) and reacts to loss, ECN, and/or delay estimates depending on algorithm.

**Detailed Explanation:** Slow start grows the sending rate quickly but cautiously; congestion avoidance grows more gradually. Implementations such as CUBIC and BBR differ in their signals and behaviour.

> **Interview Tip:** Congestion control is about fairness and stability, not receiver buffer size.

**Follow-up:** What is slow start? What is ECN?

**Follow-up Answers:** Slow start rapidly grows TCP’s congestion window from a small value while probing capacity. ECN marks packets to signal congestion without requiring a drop.

### 40. What is head-of-line blocking?

**Short Answer:** Head-of-line blocking occurs when later useful work waits behind earlier missing or blocked work. TCP’s in-order byte delivery can delay later bytes after a lost segment.

**Detailed Explanation:** HTTP/2 multiplexes streams but still rides one TCP sequence space, so a packet loss can stall streams at transport level. QUIC isolates ordering per stream.

> **Interview Tip:** Explain whether you mean HTTP/1 request queueing or TCP transport HOL.

**Follow-up:** How does HTTP/3 help? Is TCP HOL always bad?

**Follow-up Answers:** HTTP/3 uses QUIC streams so loss in one stream need not block other streams. TCP HOL is acceptable when strict total ordering is needed, but harms independent resource delivery on loss.

### 41. What is MTU?

**Short Answer:** Maximum Transmission Unit is the largest payload a particular link can carry in one frame. An IP packet larger than the path allows can require fragmentation or fail based on configuration.

**Detailed Explanation:** TCP commonly chooses MSS below MTU to avoid IP fragmentation. Path MTU can be lower than a host’s first-hop MTU because every link on the path matters.

> **Interview Tip:** MTU is link-focused; MSS is TCP payload-focused.

**Follow-up:** What is Path MTU Discovery? What is MSS?

**Follow-up Answers:** Path MTU Discovery finds the smallest usable MTU along a route. MSS is the maximum TCP payload size in one segment, normally selected below the path/link MTU.

### 42. What is IP fragmentation and why is it undesirable?

**Short Answer:** Fragmentation splits an IP datagram so it can traverse a smaller-MTU link. It is undesirable because loss of one fragment can lose the whole datagram and it adds processing/state complexity.

**Detailed Explanation:** IPv4 routers may fragment unless DF is set; IPv6 routers do not fragment transit packets, so endpoints use Path MTU Discovery and source fragmentation when needed.

> **Interview Tip:** Do not say fragmentation occurs only at endpoints for IPv4.

**Follow-up:** What happens when DF is set? Why can blocked ICMP cause issues?

**Follow-up Answers:** With IPv4 Don’t Fragment set, a router drops an oversized packet and should send an ICMP fragmentation-needed message. Blocking that feedback can create PMTU black holes.

### 43. What is TTL/Hop Limit?

**Short Answer:** IPv4 TTL and IPv6 Hop Limit are counters decremented by routers. A router discards a packet reaching zero and typically returns ICMP Time Exceeded.

**Detailed Explanation:** They prevent packets circulating forever due to routing loops. Traceroute varies initial TTL/Hop Limit to reveal routers that send those ICMP replies.

> **Interview Tip:** TTL here is hop count, not elapsed seconds in normal modern use.

**Follow-up:** How does traceroute work? Why might a hop not reply?

**Follow-up Answers:** It sends probes with increasing TTL/Hop Limit and observes ICMP responses. A router may filter, rate-limit, or deprioritize those responses while still forwarding traffic.

### 44. What is longest-prefix matching?

**Short Answer:** When multiple routes match a destination IP, the route with the most matching leading prefix bits is chosen. Thus `10.1.2.0/24` is preferred over `10.1.0.0/16`.

**Detailed Explanation:** It enables summary routes and specific exceptions to coexist. Metrics and policy can decide among equally specific alternatives.

> **Interview Tip:** Longer prefix means more specific, not a longer physical path.

**Follow-up:** What is a default route? What is route aggregation?

**Follow-up Answers:** A default route is used when no more-specific route matches. Route aggregation advertises one broad prefix to represent multiple smaller routes.

### 45. What is a VLAN?

**Short Answer:** A VLAN logically separates Layer-2 broadcast domains on shared switch infrastructure. Hosts in different VLANs normally need Layer-3 routing to communicate.

**Detailed Explanation:** VLAN tagging allows trunks to carry multiple VLANs; access ports generally carry one untagged VLAN for an endpoint. Security still needs policy and correct configuration.

> **Interview Tip:** VLAN is segmentation, but it is not automatically end-to-end security.

**Follow-up:** Access versus trunk port? Why route between VLANs?

**Follow-up Answers:** An access port normally connects one endpoint to one VLAN; a trunk carries tagged traffic for multiple VLANs. Different VLANs need routing because they are separate Layer-2 networks.

### 46. How does an Ethernet switch learn where to forward frames?

**Short Answer:** A switch records the source MAC address, ingress port, and VLAN when it receives a frame. For known destination MAC it forwards to the learned port; unknown unicast is flooded within VLAN.

**Detailed Explanation:** Entries age out. Broadcast and some multicast traffic are also flooded or specially managed inside their broadcast domain.

> **Interview Tip:** Learning uses source MAC, not destination MAC.

**Follow-up:** What is MAC flooding? What happens after a MAC moves ports?

**Follow-up Answers:** MAC flooding overwhelms switch learning so more traffic may be flooded. When a MAC appears on another port, the switch updates its table, which can also indicate a loop or spoofing.

### 47. What is a routing table?

**Short Answer:** A routing table maps destination prefixes to next hops or outgoing interfaces, often with metrics and policy attributes. The forwarding plane uses it to send packets.

**Detailed Explanation:** Entries can come from direct links, static routes, dynamic routing protocols, or host routes. A route only provides reachability; policy may still block traffic.

> **Interview Tip:** Separate control-plane route learning from data-plane forwarding.

**Follow-up:** Static versus dynamic routes? What is ECMP?

**Follow-up Answers:** Static routes are manually set; dynamic routes are learned through routing protocols. ECMP spreads flows across multiple equal-cost paths.

### 48. What is BGP and where is it used?

**Short Answer:** BGP is the inter-domain routing protocol used to exchange reachability and policy between autonomous systems on the Internet and often inside large organisations.

**Detailed Explanation:** BGP is policy-driven rather than a simple shortest-path protocol; attributes such as AS path influence route choice and loop prevention across autonomous systems.

> **Interview Tip:** Say “inter-domain” and “policy,” not merely “Internet shortest path.”

**Follow-up:** What is an ASN? What is route hijacking?

**Follow-up Answers:** An ASN identifies an autonomous system for routing policy. Route hijacking is incorrectly advertising a prefix so traffic is diverted, accidentally or maliciously.

### 49. What is a DNS record and name common record types?

**Short Answer:** A DNS record is a typed data entry in a zone. A and AAAA return IPv4/IPv6 addresses; CNAME aliases a name; MX routes mail; TXT carries text/policy data.

**Detailed Explanation:** NS delegates a zone to authoritative servers, and SOA carries zone administrative information. Clients resolve records through caching resolvers.

> **Interview Tip:** CNAME points to a hostname, not directly to an IP address.

**Follow-up:** What is NXDOMAIN? What does an NS record do?

**Follow-up Answers:** NXDOMAIN means the queried DNS name does not exist. An NS record identifies authoritative nameservers for a zone or delegated child zone.

### 50. What is DNS caching and what does TTL mean?

**Short Answer:** DNS caching stores answers for a TTL to reduce latency and authoritative-server load. TTL is a cacheability directive, not a guarantee that every resolver instantly changes at expiry.

**Detailed Explanation:** Browsers, OSes, recursive resolvers, and CDNs may each cache. Negative answers can be cached too; deployments must plan for old answers and long-lived connections.

> **Interview Tip:** Lowering TTL immediately before a change may be too late if resolvers already cached the old TTL.

**Follow-up:** Why do DNS changes appear delayed? What is negative caching?

**Follow-up Answers:** Existing resolvers and clients may retain the old record until TTL expiry. Negative caching stores a missing/error DNS result for a limited period.

### 51. What happens when you enter a URL in a browser?

**Short Answer:** Browser parses URL and checks caches, resolves DNS, routes through local gateway and routers, establishes TCP+TLS or QUIC, sends HTTP, reaches CDN/proxy/app, receives response, then renders resources.

**Detailed Explanation:** Exact steps vary with DNS cache, connection reuse, HTTP/3, CDN hit, IPv4/IPv6, service workers, and browser policy. Local ARP/Neighbor Discovery resolves the next hop, not the remote server.

> **Interview Tip:** State the flow, then explicitly acknowledge variations.

**Follow-up:** Where does ARP occur? What changes under HTTP/3?

**Follow-up Answers:** ARP happens on the local IPv4 link to resolve a neighbour, often the gateway. HTTP/3 uses QUIC over UDP with integrated TLS instead of TCP followed by TLS.

### 52. What is the difference between HTTP and HTTPS?

**Short Answer:** HTTP defines web request-response semantics; HTTPS is HTTP over TLS. HTTPS adds authenticated encryption and integrity protection for the HTTP exchange.

**Detailed Explanation:** HTTP versions can run over different transport stacks, but “HTTPS” does not mean a new application protocol. Certificate validation is essential to HTTPS security.

> **Interview Tip:** Do not say HTTPS is simply “HTTP on port 443”; port is convention.

**Follow-up:** What does TLS authenticate? What is HSTS?

**Follow-up Answers:** TLS usually authenticates the server and can authenticate clients with mTLS. HSTS tells browsers to use HTTPS for a domain after policy is learned or preloaded.

### 53. Explain a TLS handshake at a high level.

**Short Answer:** Client and server negotiate parameters, server presents certificate and key-exchange data, client validates identity, and both derive shared symmetric keys before exchanging protected application data.

**Detailed Explanation:** TLS 1.3 reduces round trips and removes older insecure patterns. SNI identifies desired virtual host and ALPN can choose HTTP/2 or HTTP/3 application protocol.

> **Interview Tip:** Separate certificate authentication from data encryption.

**Follow-up:** Why use asymmetric then symmetric crypto? What is ALPN?

**Follow-up Answers:** Asymmetric cryptography supports identity and key agreement; symmetric cryptography is faster for bulk traffic. ALPN negotiates the application protocol, such as HTTP/2.

### 54. What is a digital certificate?

**Short Answer:** A digital certificate binds a public key to identity assertions, signed by a certificate authority or another trust mechanism. Clients validate chain, hostname, time, and trust anchors.

**Detailed Explanation:** A valid signature alone is insufficient if the certificate is expired, revoked under applicable policy, for another hostname, or chains to an untrusted issuer.

> **Interview Tip:** Mention hostname validation; it is a common omission.

**Follow-up:** What is a CA? What is a self-signed certificate?

**Follow-up Answers:** A CA is a trusted issuer that signs certificates. A self-signed certificate is signed by its own key and is not browser-trusted unless the client explicitly trusts it.

### 55. What is SNI?

**Short Answer:** Server Name Indication is a TLS ClientHello extension carrying the hostname the client wants. It lets one IP address serve different certificates/sites.

**Detailed Explanation:** Without SNI, a server may not know which certificate to present before decrypting HTTP. SNI is normally visible to network observers unless newer encrypted-client-hello mechanisms are used.

> **Interview Tip:** SNI solves certificate selection for virtual hosting, not DNS resolution.

**Follow-up:** What is virtual hosting? What is ECH?

**Follow-up Answers:** Virtual hosting serves multiple domain names from one IP/server. ECH is Encrypted ClientHello, a newer TLS feature that hides more of the client hello, including protected hostname information.

### 56. What is a reverse proxy and why use one?

**Short Answer:** A reverse proxy sits in front of backend servers and receives client traffic on their behalf. It centralises TLS, routing, authentication, caching, compression, rate limiting, and observability.

**Detailed Explanation:** It can select upstreams based on host/path/header and shield applications from direct public exposure, but it adds a dependency and must correctly convey trusted client context.

> **Interview Tip:** Mention only trust proxy-provided forwarding headers.

**Follow-up:** TLS termination versus passthrough? How handle WebSocket upgrade?

**Follow-up Answers:** Termination decrypts at the proxy; passthrough keeps TLS encrypted until backend. A proxy must forward HTTP Upgrade-related headers and use suitable long idle timeouts for WebSockets.

### 57. Compare Layer 4 and Layer 7 load balancing.

**Short Answer:** L4 balancing routes TCP/UDP flows using transport/network metadata; L7 balancing understands application protocol details such as HTTP host, path, headers, or cookies.

**Detailed Explanation:** L7 enables richer routing and policies but commonly needs protocol parsing and often TLS termination. L4 is more protocol-agnostic and can preserve end-to-end encryption differently.

> **Interview Tip:** Do not equate Layer 4 with “faster” in every real system.

**Follow-up:** When choose passthrough? What is consistent hashing?

**Follow-up Answers:** Choose passthrough when backend must terminate TLS or edge should not inspect HTTP. Consistent hashing maps similar keys to targets while minimizing reassignment when targets change.

### 58. How does a load balancer determine health?

**Short Answer:** It periodically checks targets using TCP, HTTP, gRPC, or custom probes and removes failing targets from new traffic. Health should represent readiness, not merely process existence.

**Detailed Explanation:** Poor probes can send traffic to overloaded/degraded dependencies or remove too many healthy nodes. Use thresholds, timeouts, independent zones, and graceful draining.

> **Interview Tip:** A health check is a signal with false positives/negatives, not absolute truth.

**Follow-up:** Liveness versus readiness? What is connection draining?

**Follow-up Answers:** Liveness asks whether a process should be restarted; readiness asks whether it should receive traffic. Connection draining stops new work while giving existing work time to finish.

### 59. What is a CDN cache hit and cache miss?

**Short Answer:** A cache hit occurs when edge has a valid response for the request’s cache key; a miss requires fetching or revalidating from origin. Hits lower origin load and often user latency.

**Detailed Explanation:** Correctness depends on cache-control directives, TTL, vary dimensions, cookies, authorization, invalidation, and stale-serving policy.

> **Interview Tip:** Do not cache authenticated personalised data under a shared key.

**Follow-up:** What is `Cache-Control`? What is stale-while-revalidate?

**Follow-up Answers:** `Cache-Control` defines caching rules such as max age and visibility. Stale-while-revalidate permits serving a stale response briefly while refreshing it in the background.

### 60. What is WebSocket and how does it differ from HTTP?

**Short Answer:** WebSocket is a full-duplex message protocol typically initiated through an HTTP Upgrade handshake. Ordinary HTTP is primarily request-response, although streaming and server push patterns exist.

**Detailed Explanation:** After upgrade, either side can send frames. Production systems need authentication, origin checks, heartbeats, backpressure, reconnection, and cross-instance fan-out.

> **Interview Tip:** WebSocket is not raw TCP in the browser.

**Follow-up:** WebSocket versus SSE? How scale chat?

**Follow-up Answers:** WebSocket is bidirectional; SSE is server-to-client event streaming. Scale chat with many gateways and a shared broker/pub-sub system for delivery across nodes.

### 61. What is HTTP/2 and what problem does it solve?

**Short Answer:** HTTP/2 uses binary framing and multiplexes many streams over one TCP connection, reducing application-level request queueing and improving header efficiency.

**Detailed Explanation:** It retains TCP’s in-order stream, so a lost TCP packet can delay all HTTP/2 streams on that connection. Its priorities and server push require careful real-world handling.

> **Interview Tip:** HTTP/2 removes one form of HOL, not all HOL.

**Follow-up:** What is HPACK? Why can packet loss hurt HTTP/2?

**Follow-up Answers:** HPACK compresses HTTP/2 headers. Loss hurts HTTP/2 because all its streams share TCP’s in-order byte stream and wait for missing bytes.

### 62. What is HTTP/3 and how does it differ from HTTP/2?

**Short Answer:** HTTP/3 maps HTTP semantics onto QUIC rather than TCP. QUIC provides independent streams over UDP, integrated TLS 1.3, and can avoid TCP’s cross-stream packet-loss head-of-line blocking.

**Detailed Explanation:** HTTP/3 is not raw HTTP over UDP. QUIC implements reliable streams, loss recovery, congestion control, and encryption in the transport layer above UDP.

> **Interview Tip:** “Uses UDP” does not mean it has no reliability.

**Follow-up:** Why QUIC? What happens if UDP is blocked?

**Follow-up Answers:** QUIC enables encrypted multiplexed streams and fast transport evolution. If UDP is blocked or unusable, clients normally fall back to HTTP/2 or HTTP/1.1 over TCP.

### 63. What is QUIC?

**Short Answer:** QUIC is a secure multiplexed transport protocol usually carried in UDP datagrams. It integrates TLS 1.3, streams, flow control, loss recovery, congestion control, and connection migration.

**Detailed Explanation:** Implementing QUIC in user space makes transport evolution easier than kernel TCP in many environments, but requires correct congestion and resource management by the QUIC stack.

> **Interview Tip:** QUIC is a transport, HTTP/3 is an application mapping over it.

**Follow-up:** What is a QUIC connection ID? What is 0-RTT?

**Follow-up Answers:** A QUIC connection ID identifies a connection independently of a changing IP/port tuple. 0-RTT allows early data on resumed sessions but has replay risk.

### 64. Why does QUIC run over UDP?

**Short Answer:** UDP provides widely deployable IP transport and NAT/firewall traversal while letting QUIC implement new transport semantics in user space without being constrained by TCP’s fixed kernel behaviour or middleboxes.

**Detailed Explanation:** QUIC adds its own reliability and congestion control; UDP is a substrate. It also permits per-stream ordering rather than one TCP byte sequence for all streams.

> **Interview Tip:** Never answer “because UDP is reliable/faster.”

**Follow-up:** What middlebox problem exists? Does QUIC use ports?

**Follow-up Answers:** Middleboxes often assume or modify familiar TCP behaviour, making new TCP extensions hard to deploy. QUIC runs in UDP and therefore still uses UDP port numbers, commonly 443.

### 65. What is connection pooling?

**Short Answer:** Connection pooling reuses established outbound connections rather than creating one for every request. It reduces handshakes, latency, CPU, and ephemeral-port/NAT pressure.

**Detailed Explanation:** Pools need limits, idle eviction, health handling, and capacity planning. Too many connections can overload a downstream service; too few cause queueing and head-of-line effects.

> **Interview Tip:** State that pool size is bounded by downstream capacity, not arbitrary.

**Follow-up:** Why can pool exhaustion cause timeouts? HTTP keep-alive versus TCP keepalive?

**Follow-up Answers:** Pool exhaustion makes requests wait for an available connection until their deadline. HTTP keep-alive reuses application connections; TCP keepalive probes an idle TCP connection for liveness.

### 66. What is a connection timeout versus connection refused?

**Short Answer:** Connection refused usually means a reachable TCP stack actively rejected the attempt, often because no listener exists. A timeout means no successful response arrived before deadline, commonly due to drops, path issues, or overload.

**Detailed Explanation:** Neither symptom proves one unique root cause. Compare firewall/NAT policy, routing, DNS address, listener state, and packet captures.

> **Interview Tip:** Mention the RST commonly associated with refusal.

**Follow-up:** Why can a firewall cause timeout? How inspect listening ports?

**Follow-up Answers:** A firewall can silently drop packets, so no rejection reaches client. Use `ss -lntp` on Linux or `netstat`/PowerShell tools to inspect local listeners.

### 67. How would you debug DNS resolution failure?

**Short Answer:** Check exact name and error, local resolver configuration, cache/hosts file, `dig` or `nslookup` output, authoritative records, network access to resolver, and IPv4/IPv6 differences.

**Detailed Explanation:** Compare a known-good resolver and client. Distinguish NXDOMAIN, SERVFAIL, timeout, wrong answer, and stale cache because each leads to different investigation.

> **Interview Tip:** Record resolver IP and response code, not only “DNS is down.”

**Follow-up:** What is `dig +trace` useful for? What does NXDOMAIN mean?

**Follow-up Answers:** `dig +trace` follows DNS delegation from root to authoritative servers for diagnosis. NXDOMAIN means the queried name does not exist in DNS.

### 68. How would you debug a slow API?

**Short Answer:** Measure phases: DNS, connect, TLS, request upload, server processing, and response download. Correlate client timing with traces, load-balancer logs, queue depth, dependency latency, loss, and saturation.

**Detailed Explanation:** High RTT, retransmissions, pool queueing, cold cache, database waits, and large response bodies can all look like “slow network.” Use a baseline and compare affected paths.

> **Interview Tip:** Start with a latency breakdown, not a random command.

**Follow-up:** What does p99 reveal? How detect retransmissions?

**Follow-up Answers:** p99 shows slow tail requests hidden by averages. TCP retransmissions can be seen in host/network metrics or inferred/analyzed with packet captures.

### 69. How would you debug one client that cannot reach a server while others can?

**Short Answer:** Compare DNS result, chosen IP family, source subnet/VPN/proxy, route, firewall policy, TLS clock/trust, credentials, and client version against a working client.

**Detailed Explanation:** Test name, port, TLS, and HTTP separately. Packet capture at client/server/load balancer can determine how far the flow gets without guessing.

> **Interview Tip:** The difference between working and failing clients is your strongest clue.

**Follow-up:** Why could IPv6 be the cause? How can proxy settings differ?

**Follow-up Answers:** A client may choose a broken/unreachable AAAA route while IPv4 works. Proxy settings can differ by OS, browser, VPN, PAC file, environment variables, or user policy.

### 70. What commands are useful for network troubleshooting?

**Short Answer:** Use `ip addr`/`ip route` for local config, `dig`/`nslookup` for DNS, `ping` for permitted ICMP, `traceroute`/`tracert` for path hints, `curl -v` for HTTP/TLS, `ss` for sockets, and `tcpdump` for packet evidence.

**Detailed Explanation:** Commands answer different questions; no one command proves end-to-end application health. Avoid capturing or exposing sensitive traffic without authorisation.

> **Interview Tip:** Name the question each command answers.

**Follow-up:** `netstat` versus `ss`? When is packet capture necessary?

**Follow-up Answers:** `ss` is the modern Linux socket inspection tool and is often faster; `netstat` is older and widely known. Capture packets when logs/metrics cannot show whether traffic reaches a point or what replies occur.

## Advanced — 71–90

### 71. What is the bandwidth-delay product and why does it matter?

**Short Answer:** Bandwidth-delay product is bandwidth multiplied by RTT; it approximates how much data must be in flight to fully use a path. High-BDP links need sufficiently large windows/pipelines.

**Detailed Explanation:** For 1 Gbps and 100 ms RTT, about 12.5 MB must be in flight. TCP window scaling and application concurrency can be relevant bottlenecks.

> **Interview Tip:** Convert bits to bytes carefully.

**Follow-up:** How do rwnd/cwnd limit BDP? What is window scaling?

**Follow-up Answers:** The usable TCP send limit is constrained by the smaller of `rwnd` and `cwnd`. Window scaling is a TCP option that permits windows larger than the original 16-bit field.

### 72. What is bufferbloat?

**Short Answer:** Bufferbloat is excessive queueing caused by oversized buffers, producing high latency and jitter under load even when bandwidth is available. It harms interactive traffic.

**Detailed Explanation:** Active queue management, fair queueing, sensible buffering, and congestion control can keep queues shorter while retaining throughput.

> **Interview Tip:** More buffering is not always better.

**Follow-up:** What is AQM? Why does queueing raise RTT?

**Follow-up Answers:** Active Queue Management proactively marks/drops traffic to control queues. Queueing raises RTT because packets wait behind earlier packets before transmission.

### 73. What is ECN?

**Short Answer:** Explicit Congestion Notification lets network devices mark congestion in capable packets instead of dropping them, and endpoints can reduce sending on receiving that signal.

**Detailed Explanation:** ECN requires compatible endpoints and network treatment. It can signal congestion earlier than loss but is not universally available or configured correctly.

> **Interview Tip:** ECN is congestion feedback, not an error-correction mechanism.

**Follow-up:** How does ECN relate to TCP? Why might it be disabled?

**Follow-up Answers:** TCP endpoints can negotiate ECN and reduce sending after congestion marks. It may be disabled because of compatibility, middlebox, or operational-policy concerns.

### 74. What is Path MTU Discovery and how can it fail?

**Short Answer:** Path MTU Discovery discovers the smallest usable MTU along a route, allowing senders to avoid fragmentation. It can fail if essential ICMP “too big/fragmentation needed” messages are blocked.

**Detailed Explanation:** The result can be a PMTU black hole: small traffic works but larger packets hang. MSS clamping and correct ICMP policy are common mitigations.

> **Interview Tip:** This is a classic explanation for “some HTTPS requests hang.”

**Follow-up:** IPv4 versus IPv6 fragmentation? What is MSS clamping?

**Follow-up Answers:** IPv4 routers may fragment packets, while IPv6 routers do not fragment transit packets. MSS clamping lowers advertised TCP MSS at a boundary to avoid oversized packets.

### 75. Explain NAT port exhaustion.

**Short Answer:** A NAT needs unique external mapping tuples for concurrent outbound flows. High connection churn or fan-out can consume available source ports or table capacity, causing new connections to fail or time out.

**Detailed Explanation:** Reusing keep-alive connections, scaling NAT egress addresses, reducing unnecessary retries, and monitoring allocation failures help. Capacity depends on destination tuples and NAT implementation.

> **Interview Tip:** Do not assume one public IP has only 65,535 total simultaneous flows in every scenario; mapping rules matter.

**Follow-up:** Why does connection pooling help? How detect NAT exhaustion?

**Follow-up Answers:** Pooling reduces new outbound mappings by reusing existing connections. Detect exhaustion through NAT allocation errors, high connection failures/timeouts, port/table metrics, and correlation with traffic spikes.

### 76. What is a SYN flood and how is it mitigated?

**Short Answer:** A SYN flood sends many TCP SYNs without completing handshakes, consuming backlog/state or network capacity. Mitigations include SYN cookies, rate limits, filtering, increased backlog, and upstream DDoS protection.

**Detailed Explanation:** It targets connection-establishment resources; protecting it requires both host-stack and network-edge measures. Legitimate bursts must be distinguished from attack traffic.

> **Interview Tip:** SYN cookies preserve service under pressure but have trade-offs and are not the sole defence.

**Follow-up:** What is a backlog? What is half-open state?

**Follow-up Answers:** A backlog is a queue/limit for pending or accepted connection work. Half-open TCP state exists after one side has begun setup but the handshake is not complete.

### 77. What is DDoS and what are the main defence layers?

**Short Answer:** Distributed denial of service uses many sources to exhaust bandwidth, connection state, CPU, or application resources. Defence combines CDN/anycast scrubbing, rate limits, WAF, autoscaling, caching, quotas, and resilient origin design.

**Detailed Explanation:** Volumetric, protocol, and application-layer attacks differ, so a single firewall rule is insufficient. Monitoring and upstream coordination are essential at large scale.

> **Interview Tip:** Separate availability defence from authentication/authorization.

**Follow-up:** Volumetric versus L7 DDoS? Why can autoscaling be insufficient?

**Follow-up Answers:** Volumetric attacks exhaust network bandwidth; L7 attacks exhaust HTTP/application resources. Autoscaling may be too slow, costly, or unable to scale shared bottlenecks such as databases and upstream bandwidth.

### 78. What is TLS termination versus TLS passthrough?

**Short Answer:** TLS termination decrypts at a proxy/load balancer, enabling HTTP-aware policy before forwarding. Passthrough forwards encrypted TLS to backend, preserving end-to-end termination there.

**Detailed Explanation:** Termination simplifies central certificates and L7 routing but creates a trust boundary and may require re-encryption. Passthrough limits L7 visibility at the front door.

> **Interview Tip:** Explain where plaintext exists and how it is protected after termination.

**Follow-up:** What is TLS re-encryption? When use mTLS behind a gateway?

**Follow-up Answers:** Re-encryption starts a new TLS connection from gateway to backend after edge termination. Use backend mTLS when services need authenticated encrypted hops inside the private network.

### 79. What is mutual TLS (mTLS)?

**Short Answer:** mTLS is TLS where both client and server present and validate certificates. It provides strong cryptographic service/client identity in addition to channel protection.

**Detailed Explanation:** It is common for internal service-to-service communication but requires certificate issuance, rotation, trust distribution, authorization mapping, and observability.

> **Interview Tip:** mTLS authenticates identity; it does not decide every action that identity may perform.

**Follow-up:** How rotate certificates? mTLS versus API token?

**Follow-up Answers:** Automate issuance, distribute overlapping valid certificates, reload safely, monitor expiry, and revoke/replace compromised credentials. mTLS proves transport peer identity; API tokens often express user/application authorization claims.

### 80. How do you design retries safely in a distributed system?

**Short Answer:** Use explicit deadlines, bounded attempts, exponential backoff with jitter, retry budgets, and idempotency where operations may have completed despite timeout. Avoid retrying every error blindly.

**Detailed Explanation:** Retries multiply load during partial outages. For non-idempotent actions, idempotency keys or durable operation identifiers let server deduplicate requests.

> **Interview Tip:** A timeout means “unknown outcome,” not necessarily “operation failed.”

**Follow-up:** What is a retry storm? What errors should not be retried?

**Follow-up Answers:** A retry storm is feedback where failing calls create extra load through retries and worsen the outage. Do not blindly retry validation errors, authorization failures, or unsafe non-idempotent actions.

### 81. What is service discovery and how does it relate to DNS?

**Short Answer:** Service discovery finds healthy instances of a logical service. DNS can be one discovery mechanism, while registries, sidecars, and control planes add health, metadata, and dynamic updates.

**Detailed Explanation:** DNS TTL and client caching make fast membership changes tricky. Discovery only finds candidates; load balancing, retries, and health policies still decide actual traffic.

> **Interview Tip:** Do not equate service discovery with a database of IPs only.

**Follow-up:** Client-side versus server-side discovery? What are stale endpoints?

**Follow-up Answers:** Client-side discovery lets clients choose instances; server-side discovery sends clients to a proxy/load balancer that chooses. Stale endpoints are removed/unhealthy instances still present in discovery data or caches.

### 82. How do you scale WebSocket connections?

**Short Answer:** Use horizontally scalable gateways with event-loop efficiency, connection limits, heartbeats, shared pub/sub for cross-node fan-out, routing affinity only when needed, and reconnect/resume design.

**Detailed Explanation:** Capacity is driven by open connections, memory, file descriptors, TLS CPU, message rate, and fan-out—not merely requests per second. Backpressure protects slow clients.

> **Interview Tip:** Explain how a message reaches users connected to different gateway instances.

**Follow-up:** Why use a broker? How manage slow consumers?

**Follow-up Answers:** A broker distributes events from a publisher to clients connected to many gateway nodes. Manage slow consumers with bounded buffers, rate limits, dropped/coalesced updates, and disconnect/resync policies.

### 83. What is anycast and where is it useful?

**Short Answer:** Anycast advertises the same IP prefix from multiple locations; routing sends a client toward a topologically suitable announcement. It is used by DNS providers, CDNs, and DDoS absorption networks.

**Detailed Explanation:** Anycast improves locality and resilience but routing changes can move traffic. Stateful long-lived flows need careful design because a route change may reach a different site.

> **Interview Tip:** Anycast selects by routing topology, not necessarily geographic nearest point.

**Follow-up:** Anycast versus unicast? How does it affect TCP sessions?

**Follow-up Answers:** Unicast targets one specific routeable endpoint; anycast advertises one address from multiple sites. Route changes can move new or sometimes existing traffic, so session continuity needs careful site/state design.

### 84. What is ECMP and what is its trade-off?

**Short Answer:** Equal-Cost Multi-Path uses multiple equal-cost routes to spread traffic, usually by hashing flow attributes. It increases aggregate capacity and resilience.

**Detailed Explanation:** Hashing preserves per-flow ordering but can create imbalance when a few “elephant” flows hash to one path. Per-packet balancing risks reordering.

> **Interview Tip:** Explain flow hashing and elephant flows.

**Follow-up:** Why is reordering bad for TCP? How mitigate imbalance?

**Follow-up Answers:** Reordering can make TCP infer loss and delay in-order delivery. Mitigate ECMP imbalance with better hash fields, flow-aware balancing, more paths, or splitting large flows where appropriate.

### 85. What is a network partition and how should services handle it?

**Short Answer:** A network partition is a communication failure where components cannot reliably reach each other despite possibly remaining operational. Treat remote calls as failure-prone and design timeouts, fallback, consistency, and recovery.

**Detailed Explanation:** A timeout cannot distinguish slow from partitioned or overloaded service. Distributed systems choose behaviour based on business correctness: reject, queue, serve stale data, or degrade.

> **Interview Tip:** “Retry until it works” is not a partition strategy.

**Follow-up:** What is split brain? How does CAP relate?

**Follow-up Answers:** Split brain is when partitioned components independently act as if they are the sole authority. CAP says that during a partition a distributed system must choose trade-offs between strong consistency and availability.

### 86. How would you observe network performance in production?

**Short Answer:** Measure client/server latency percentiles, DNS/connect/TLS timings, retransmissions, error rates, connection counts, queue depth, saturation, packet loss, CDN hit ratio, and path/region dimensions.

**Detailed Explanation:** Combine metrics with distributed traces and structured logs containing request IDs. Observe both golden signals and lower-level host/NIC/load-balancer counters.

> **Interview Tip:** p99 and error correlation often reveal problems hidden by averages.

**Follow-up:** What metrics reveal pool exhaustion? Why trace context propagation?

**Follow-up Answers:** Look for pool wait time, active/idle connections, queue length, timeout rate, and downstream saturation. Trace context propagation joins spans across services into one request path.

### 87. What is a zero-trust networking approach?

**Short Answer:** Zero trust assumes network location alone does not confer trust. Every access is explicitly authenticated, authorised, encrypted as appropriate, and logged with least privilege.

**Detailed Explanation:** It uses identity-aware policy, segmentation, short-lived credentials, device posture where relevant, and continuous verification; it is an architectural approach, not one product.

> **Interview Tip:** Zero trust does not mean trust nobody forever; it means verify explicitly.

**Follow-up:** How does mTLS support it? What is least privilege?

**Follow-up Answers:** mTLS gives strong service identity and channel protection for policy decisions. Least privilege gives each identity only the minimum access it needs.

### 88. What is SSRF and why is it a networking concern?

**Short Answer:** Server-Side Request Forgery lets an attacker induce a server to make unintended network requests, often reaching internal services or cloud metadata endpoints inaccessible from outside.

**Detailed Explanation:** Mitigate with strict URL validation, egress allow-lists, network segmentation, metadata-service protections, identity controls, and avoiding user-controlled generic fetchers.

> **Interview Tip:** Application input validation and egress network policy both matter.

**Follow-up:** Why are metadata endpoints sensitive? How can DNS rebinding matter?

**Follow-up Answers:** Metadata endpoints may return cloud credentials or instance configuration. DNS rebinding can make a previously allowed hostname resolve to a private/internal address after validation.

### 89. Why can an HTTPS request work on one network but not another?

**Short Answer:** Possible differences include DNS answer, IPv4/IPv6 routing, proxy/VPN policy, firewall/UDP filtering, MTU/ICMP behaviour, TLS interception/trust stores, time, or captive portals.

**Detailed Explanation:** Compare exact resolved IP, route, port connection, TLS certificate/ALPN, and HTTP response between networks. Isolate each layer rather than assuming a server defect.

> **Interview Tip:** `curl -v` from both paths creates a useful evidence comparison.

**Follow-up:** How detect TLS interception? Why can HTTP/3 fallback matter?

**Follow-up Answers:** Compare presented certificate issuer/chain and connection behaviour against trusted expectations; corporate proxies may install their own CA. HTTP/3 fallback matters because UDP filtering can change protocol, latency, and debugging evidence.

### 90. How do you handle multi-region traffic routing?

**Short Answer:** Combine global DNS/anycast or traffic management with regional ingress, health-aware failover, replicated data strategy, clear latency/compliance goals, and safe client retry/failover behaviour.

**Detailed Explanation:** Routing traffic is easier than making state consistent. Plan for stale DNS, connection pinning, regional partial failure, data residency, and failback to avoid oscillation.

> **Interview Tip:** Discuss data consistency before claiming active-active is automatically better.

**Follow-up:** Active-active versus active-passive? How test regional failover?

**Follow-up Answers:** Active-active serves traffic from multiple regions; active-passive keeps a standby region for failover. Test failover with controlled drills that include traffic, data, DNS, health checks, and recovery/failback.

## Tricky / Interview Trap — 91–100

### 91. Does TCP guarantee that packets never get lost?

**Short Answer:** No. Packets can be lost in the network. TCP detects many losses and retransmits bytes so the application usually receives an ordered reliable stream, or eventually gets an error/timeout.

**Detailed Explanation:** TCP reliability is an end-to-end recovery mechanism, not a promise that every physical packet arrives. Connection failure or application deadline can still prevent successful delivery.

> **Common Trap:** Saying “TCP prevents packet loss.”

**Follow-up:** How does TCP infer loss? Can a process crash after receiving data?

**Follow-up Answers:** TCP infers loss from timeout or duplicate-ACK/SACK signals. Yes, a process can receive data then crash before persisting or completing business work, so transport reliability is not transaction reliability.

### 92. Is HTTPS a separate protocol from HTTP?

**Short Answer:** HTTPS is HTTP over TLS, not a separate application-layer protocol replacing HTTP semantics. TLS provides the secure transport channel around HTTP.

**Detailed Explanation:** HTTP/1.1 and HTTP/2 commonly use TLS over TCP; HTTP/3 uses TLS integrated with QUIC. The API methods and status codes remain HTTP.

> **Common Trap:** Equating HTTPS with “TCP port 443” only.

**Follow-up:** What does TLS add? Can HTTP run on a nonstandard port?

**Follow-up Answers:** TLS adds authenticated encryption and integrity protection around HTTP. Yes, HTTP can run on any configured port; 80 and 443 are conventions.

### 93. Does DNS always use UDP port 53?

**Short Answer:** No. UDP/53 is common for ordinary queries, but DNS can use TCP/53 and encrypted transports such as DoT or DoH. Transport depends on response, client, resolver, and policy.

**Detailed Explanation:** Zone transfers classically use TCP; response truncation can trigger TCP retry. Modern DNS has multiple deployment modes beyond the old simple statement.

> **Common Trap:** Giving an absolute answer to a conditional protocol choice.

**Follow-up:** Why is UDP common? What is DNS-over-HTTPS?

**Follow-up Answers:** UDP is lightweight for common small DNS queries. DNS-over-HTTPS sends DNS inside HTTPS, which can protect queries from some local network observers.

### 94. Is a MAC address used to route traffic across the Internet?

**Short Answer:** No. MAC addresses are normally used for local link delivery. Routers forward based on destination IP and replace the link-layer frame, so MAC addresses change hop by hop.

**Detailed Explanation:** The source/destination IP packet travels across routing domains while each local segment has its own next-hop link addressing.

> **Common Trap:** Saying the sender ARPs for Google’s MAC address.

**Follow-up:** What MAC does a host use for a remote destination? What does ARP resolve?

**Follow-up Answers:** For a remote destination, the host uses the default gateway’s MAC in the local frame. ARP resolves a local IPv4 address to a MAC address.

### 95. Does UDP mean no connection state exists anywhere?

**Short Answer:** UDP itself has no TCP-style connection handshake or reliable connection state, but applications, NATs, firewalls, and protocols built on UDP can maintain significant state.

**Detailed Explanation:** QUIC creates connection semantics above UDP; a stateful firewall tracks UDP flows; a game server tracks sessions. “Connectionless” describes UDP’s transport semantics.

> **Common Trap:** Treating UDP traffic as stateless in every layer.

**Follow-up:** How does QUIC establish state? What UDP state can NAT keep?

**Follow-up Answers:** QUIC creates state through its handshake, connection IDs, cryptographic keys, streams, and flow control. NAT can keep mappings and timers for UDP source/destination tuples.

### 96. Is a private IP address automatically secure?

**Short Answer:** No. Private addressing prevents ordinary public Internet routing of those addresses, but internal attackers, misconfigurations, VPN users, peering, and compromised workloads can still reach services.

**Detailed Explanation:** Security needs least-privilege firewall/security-group rules, identity, encryption, patching, and monitoring. NAT may reduce unsolicited ingress but is not an access-control policy.

> **Common Trap:** “It is private, so no authentication is needed.”

**Follow-up:** NAT versus firewall? What is network segmentation?

**Follow-up Answers:** NAT rewrites addresses/ports; a firewall allows or denies traffic by policy. Network segmentation divides networks/workloads so only intended communication paths are allowed.

### 97. Does a successful ping prove a website is available?

**Short Answer:** No. Ping tests ICMP echo behaviour if allowed. A website additionally needs correct DNS, route, TCP/QUIC reachability, TLS, HTTP, application dependencies, and authorization.

**Detailed Explanation:** ICMP may be blocked while website works, and ICMP may work while port 443 or the application is down. Use targeted tests such as `curl -v`.

> **Common Trap:** Treating ping as a complete health check.

**Follow-up:** What does connection refused show? Why can ping be blocked?

**Follow-up Answers:** Refused usually shows a reachable TCP stack sent a rejection, often a reset because no listener exists. Ping can be blocked or rate-limited by host/firewall policy.

### 98. Does HTTP/2 completely eliminate head-of-line blocking?

**Short Answer:** No. HTTP/2 multiplexes HTTP streams and reduces application-level queueing, but all streams share TCP’s ordered byte stream. A lost TCP segment can delay delivery for all streams.

**Detailed Explanation:** HTTP/3/QUIC offers independent stream ordering so loss in one stream need not block other streams at the transport level.

> **Common Trap:** Forgetting the underlying transport layer.

**Follow-up:** What is TCP HOL? How does QUIC differ?

**Follow-up Answers:** TCP HOL means missing earlier bytes block later bytes in one ordered stream. QUIC has independently ordered streams, so loss in one need not block another stream.

### 99. Is a timeout always a network problem?

**Short Answer:** No. A timeout means the caller did not receive a successful response by its deadline. Causes include packet drops, routing, overload, queueing, DNS, TLS, server stalls, dependency slowness, or an overly short timeout.

**Detailed Explanation:** Use layered telemetry and packet/log evidence to see whether connection began, TLS completed, request reached server, and response was produced.

> **Common Trap:** Blaming “the network” without phase-level evidence.

**Follow-up:** Timeout versus refused? What does tracing show?

**Follow-up Answers:** Timeout means no success before deadline; refused is an active rejection from a reachable endpoint. Distributed tracing shows which service/span phase consumed time and where errors occurred.

### 100. Does a load balancer make an application highly available by itself?

**Short Answer:** No. It can distribute traffic and remove unhealthy targets, but availability also depends on zones, DNS/ingress, application statelessness, data stores, dependencies, health checks, capacity, and failure handling.

**Detailed Explanation:** A load balancer can become a dependency, route to falsely healthy targets, or expose a shared database bottleneck. HA is a complete system property.

> **Common Trap:** Naming a component instead of explaining failure domains and recovery.

**Follow-up:** What is connection draining? How avoid a single database failure?

**Follow-up Answers:** Connection draining removes a target from new traffic while allowing existing work to finish. Avoid a single database failure with replication/failover, backups, tested recovery, caching, and careful consistency design.
