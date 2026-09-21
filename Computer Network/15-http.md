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
| **Safe Method** | Calling method does not mutate server state (`GET`, `HEAD`, `OPTIONS`) |
| **Idempotent Method** | Multiple identical requests have the exact same effect on server state as one request (`PUT`, `DELETE`, `GET`) |
| **Non-Idempotent** | Repeating request produces different state / side-effects (`POST`) |
| **Origin** | Scheme + Host + Port tuple defining browser security sandbox (CORS/SOP) |
| **Idempotency Key** | Unique client-generated token (UUID) in header to prevent duplicate charges/orders |

### HTTP Status Code Quick-Reference

| Code | Status Name | Category | Meaning & Interview Caveat |
|---|---|---|---|
| **200** | OK | Success | Standard success with payload |
| **201** | Created | Success | Resource created (e.g. after POST); returns `Location` header |
| **204** | No Content | Success | Action succeeded; response body is intentionally empty (e.g. DELETE) |
| **301** | Moved Permanently | Redirection | Permanent redirect; browser caches new URL aggressively |
| **304** | Not Modified | Redirection | Conditional GET cache hit (`ETag` / `If-None-Match` match); no body transferred |
| **400** | Bad Request | Client Error | Malformed payload or validation error |
| **401** | Unauthorized | Client Error | Authentication required or invalid credentials |
| **403** | Forbidden | Client Error | Identity is authenticated, but user lacks permission/role |
| **404** | Not Found | Client Error | Requested resource URI does not exist |
| **409** | Conflict | Client Error | State conflict (e.g. duplicate key, concurrent edit version mismatch) |
| **429** | Too Many Requests| Client Error | Rate limit exceeded; client should inspect `Retry-After` header |
| **500** | Internal Server Error | Server Error | Unhandled exception inside the backend application |
| **502** | Bad Gateway | Server Error | Reverse proxy/NGINX received invalid/dropped response from upstream app |
| **503** | Service Unavailable | Server Error | Server temporarily overloaded or down for maintenance |
| **504** | Gateway Timeout | Server Error | Upstream application took too long to answer reverse proxy/LB |

## Example

`curl -i https://example.com` shows response headers and status. `curl -v` helps expose DNS, TLS, proxy, and request details.

## Real-World Example

Payment APIs (Stripe, Razorpay) enforce idempotency: `POST /v1/charges` accepts `Idempotency-Key: <uuid>`. If a network timeout occurs and the client retries, the server recognizes the key, skips duplicate credit card debit, and replays the cached 200 response.

## Important Properties

| Method | Safe? | Idempotent? | Typical Use Case |
|---|---|---|---|
| **GET** | Yes | Yes | Retrieve representation without mutation |
| **POST** | No | No | Create subordinate resource / trigger complex action |
| **PUT** | No | Yes | Replace resource completely or create at known URI |
| **PATCH** | No | Usually No | Apply partial modifications to a resource |
| **DELETE** | No | Yes | Remove resource (repeating yields same result) |

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
