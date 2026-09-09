# WebSockets and Sockets

## What is it?

A socket is an OS programming endpoint for network communication. WebSocket is an application protocol that begins with an HTTP Upgrade request and provides full-duplex messages over a long-lived connection.

## Simple Explanation

A socket is the phone endpoint an app uses. WebSocket is a persistent call where browser and server can both speak whenever they need, unlike ordinary request-response HTTP.

## Hinglish Explanation

Socket low-level OS interface hai. WebSocket browser-friendly protocol hai: HTTP handshake se upgrade hota hai, phir client aur server dono asynchronously message bhej sakte hain. Long-lived connections ko scale karna hota hai.

## Why Do We Need It?

Real-time dashboards, chat, collaboration, gaming, and notifications need server push with lower repeated-request overhead.

## How It Works

1. Client sends HTTP request with `Upgrade: websocket`.
2. Server accepts with `101 Switching Protocols` if policy allows.
3. Both exchange framed WebSocket messages over the persistent connection.
4. Heartbeats, auth expiry, backpressure, reconnects, and fan-out are application responsibilities.

```text
Browser -- HTTP Upgrade --> Gateway -- TCP/TLS --> Service
Browser <== WebSocket messages ==> Gateway <==> Service
```

## Important Components / Terminology

| Term | Meaning |
|---|---|
| Bind | Associate socket with local address/port |
| Listen/accept | TCP server waits for then creates connections |
| Backpressure | Slow consumer limiting producer safely |
| Heartbeat | Detect dead/stale persistent connection |
| Sticky session | Same client routed consistently when state is local |

## Example

A chat service sends each room message to a broker/topic, then WebSocket gateway instances fan it out to locally connected clients. A reconnect must resynchronize missed events.

## Real-World Example

Reverse proxies need upgrade support and idle timeouts sized for WebSockets. Load balancers track many open connections, so connection count—not just request rate—matters.

## Important Properties

| Pattern | Direction | Best fit |
|---|---|---|
| HTTP polling | Client initiates | Simple, low-frequency updates |
| SSE | Server to client | Streaming events |
| WebSocket | Full duplex | Interactive real time |

### Common Misconceptions

* WebSocket is not raw TCP exposed to JavaScript.
* A WebSocket connection is not automatically scalable or reliable across reconnects.
* WebSocket security still needs origin checks, authentication, and authorization.

### What Interviewers Usually Test

Protocol lifecycle and designing state, fan-out, reconnection, and backpressure for a real-time system.

### Common Follow-up Questions

1. WebSocket versus SSE?
2. How do you scale chat gateways?
3. Why heartbeats?
4. How handle reconnects?
5. What is socket backpressure?

### Follow-up Answers

1. WebSocket supports two-way messages; Server-Sent Events mainly stream server-to-client events over HTTP and use normal HTTP for client writes.
2. Run many stateless connection gateways, use a broker/pub-sub layer for cross-node messages, track capacity, and design reconnect/resume behaviour.
3. Heartbeats detect dead connections hidden by NATs, proxies, sleeping clients, or broken network paths before sending important data.
4. Clients reconnect with backoff, re-authenticate, and request missed events using a cursor or sequence number when the product requires continuity.
5. Backpressure is a mechanism that slows or limits a producer when a socket receiver or send buffer cannot keep up.
