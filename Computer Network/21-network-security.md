# Network Security

## What is it?

Network security protects confidentiality, integrity, availability, identity, and access across networked systems. It combines protocol choices, segmentation, identity, observability, and safe operations.

## Simple Explanation

Secure networking is not one lock. It is several layers: verify who is talking, encrypt the conversation, permit only needed paths, watch for abuse, and recover when something fails.

## Hinglish Explanation

Security sirf firewall rule nahi hai. TLS data protect karta hai, auth identity check karta hai, authorization permission decide karta hai, segmentation blast radius reduce karti hai, aur logs/alerts incident detect karte hain.

## Why Do We Need It?

Networks face spoofing, interception, unauthorized access, denial of service, misconfiguration, and lateral movement. Defense in depth avoids dependence on one control.

## How It Works

1. Authenticate users/services (tokens, certificates, mTLS where appropriate).
2. Authorize least-privilege actions and network paths.
3. Encrypt in transit and validate certificates.
4. Segment networks, rate-limit ingress, and patch exposed services.
5. Log, monitor, test, rotate secrets, and prepare incident response.

## Important Components / Terminology

| Term | Meaning |
|---|---|
| Authentication | Proving identity |
| Authorization | Deciding allowed action |
| mTLS | Mutual certificate authentication |
| DDoS | Distributed attempt to exhaust availability |
| Zero trust | Verify explicitly; do not assume network location is trust |

## Example

An internal payment API may require mTLS at the service mesh, JWT authorization at the gateway, security-group rules at network layer, and audited database credentials.

## Real-World Example

SSRF can let an application server make unintended internal requests. Egress allow-lists, metadata-service hardening, URL validation, and identity-bound access reduce impact.

## Important Properties

| Control | Primary protection | Limitation |
|---|---|---|
| TLS | In-transit confidentiality/integrity | Endpoint compromise |
| Firewall | Path restriction | Valid allowed traffic can still be malicious |
| mTLS | Service identity | Needs lifecycle/rotation |
| Rate limit | Abuse reduction | Not full DDoS solution |

### Common Misconceptions

* Encryption does not replace authorization.
* “Internal” is not a sufficient security boundary.
* An allow rule should be as narrow as practical, but security also needs operational validation.

### What Interviewers Usually Test

Threat-model thinking and practical controls around TLS, identity, segmentation, DDoS, and observability.

### Common Follow-up Questions

1. Authentication versus authorization?
2. What does mTLS solve?
3. How defend an API from DDoS?
4. Why segment networks?
5. What is SSRF’s networking risk?

### Follow-up Answers

1. Authentication proves who a user or service is; authorization decides which resources or actions that identity may use.
2. mTLS provides mutual cryptographic endpoint identity and an encrypted integrity-protected channel; application authorization is still required.
3. Combine CDN/anycast or scrubbing, rate limits, WAF rules, caching, quotas, application hardening, capacity planning, and monitoring.
4. Segmentation restricts which workloads can communicate, reduces blast radius, and makes least-privilege policy practical.
5. SSRF can make a trusted server access internal services or cloud metadata endpoints that an outside attacker cannot directly reach.
