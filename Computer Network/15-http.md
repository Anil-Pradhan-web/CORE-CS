# HTTP

## What is it?

HTTP is an application-layer protocol for exchanging requests and responses. It defines methods, URLs, headers, status codes, and message semantics—not encryption or transport reliability by itself.

## Simple Explanation

HTTP is the structured conversation between a client and web service: “GET this resource,” then “here is a 200 response with data.”

## Hinglish Explanation

HTTP browser aur server ke beech request-response language hai. Method intent batata hai, URL resource, headers metadata, aur status code result. HTTPS mein ye same HTTP TLS ke andar chalta hai.

## Why Do We Need It?

Web clients and services need a shared, extensible interface independent of programming language and host operating system.

## How It Works

1. Client chooses an origin, method, target, headers, and optional body.
2. Server authenticates/authorizes, processes request, and returns status, headers, and optional body.
3. Client uses caching, redirects, cookies, or retries according to semantics.

```http
GET /users/42 HTTP/1.1
Host: api.example.com
Accept: application/json

HTTP/1.1 200 OK
Content-Type: application/json
```

## Important Components / Terminology

| Term | Meaning |
|---|---|
| Method | Semantic intent: GET, POST, PUT, DELETE, etc. |
| Status code | Response category/result, e.g. 200, 404, 503 |
| Header | Metadata such as content type or cache policy |
| Origin | Scheme + host + port, relevant to browser policy |
| Idempotent | Repeating has intended same effect (e.g. PUT ideally) |

## Example

`curl -i https://example.com` shows response headers and status. `curl -v` helps expose DNS, TLS, proxy, and request details.

## Real-World Example

APIs use `Cache-Control`, `ETag`, timeouts, idempotency keys, and explicit retry policies. A safe retry depends on operation semantics, not merely HTTP method names.

## Important Properties

| Version | Transport framing |
|---|---|
| HTTP/1.1 | Textual framing, persistent connections |
| HTTP/2 | Binary frames over TCP |
| HTTP/3 | Frames over QUIC |

### Common Misconceptions

* HTTP is not necessarily stateless at the application level; cookies/tokens provide state patterns.
* `POST` does not inherently mean “create”; semantics are API design.
* HTTP 200 does not guarantee business success unless the API defines it so.

### What Interviewers Usually Test

You should choose correct methods/statuses, explain idempotency/caching, and distinguish HTTP from HTTPS.

### Common Follow-up Questions

1. GET versus POST versus PUT?
2. 401 versus 403?
3. What is an idempotency key?
4. How does HTTP caching work?
5. What is a reverse proxy’s HTTP role?

### Follow-up Answers

1. GET retrieves a representation, POST submits work or creates a subordinate resource by API design, and PUT replaces/creates a resource at a known target and is intended to be idempotent.
2. `401 Unauthorized` usually means authentication is missing or invalid; `403 Forbidden` means identity is known but not allowed, though APIs may choose wording carefully.
3. An idempotency key lets a server recognise repeated client attempts of the same operation and avoid doing it twice.
4. Clients and intermediaries reuse responses when cache directives and keys allow; expiry, validation with ETags, and invalidation control freshness.
5. A reverse proxy receives HTTP for backend services and can terminate TLS, route requests, apply policy, cache, and forward to upstreams.
