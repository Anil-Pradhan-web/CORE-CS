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

### 1. TCP Socket System Call Lifecycle

```text
Server Socket Lifecycle:
  socket() ──► bind(addr, port) ──► listen(backlog) ──► accept() [blocks]
                                                            │
Client Socket Lifecycle:                                   │ (new client_fd created)
  socket() ──────────────────────► connect(addr, port) ────┘
                                        │
                         send() / recv() [bidirectional]
                                        │
                                     close()
```

#### Python TCP Socket Example
```python
import socket

# TCP Server
server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
server.bind(('0.0.0.0', 8080))
server.listen(128) # Backlog queue size

conn, client_addr = server.accept() # Blocks until client connects
data = conn.recv(1024)
conn.sendall(b"HTTP/1.1 200 OK\r\nContent-Length: 5\r\n\r\nHello")
conn.close()
```

#### Java TCP Socket Example
```java
import java.io.*;
import java.net.*;

public class SimpleTcpServer {
    public static void main(String[] args) throws IOException {
        try (ServerSocket server = new ServerSocket(8080)) {
            Socket client = server.accept(); // Blocks until connection
            BufferedReader in = new BufferedReader(new InputStreamReader(client.getInputStream()));
            PrintWriter out = new PrintWriter(client.getOutputStream(), true);

            String requestLine = in.readLine();
            out.println("HTTP/1.1 200 OK\r\nContent-Length: 13\r\n\r\nHello Socket!");
        }
    }
}
```

---

### 2. The WebSocket Upgrade Handshake

A WebSocket starts its life as a standard HTTP/1.1 request over TCP/TLS:

```http
Client Request:
GET /chat HTTP/1.1
Host: server.example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Sec-WebSocket-Version: 13

Server Response:
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
```

Once the `101 Switching Protocols` response is received, the HTTP parser is turned off. Both parties exchange lightweight WebSocket binary/text frames over the persistent TCP stream with minimal 2-to-10 byte framing overhead (eliminating repeated HTTP headers).

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
