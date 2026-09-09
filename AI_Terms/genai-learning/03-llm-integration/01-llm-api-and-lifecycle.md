# LLM API Concept & Request–Response Lifecycle

## 1. What is it?

An **LLM API** is how software talks to an LLM: you send a structured HTTP request (messages + settings), and a server running the model sends back generated text. Providers: OpenAI, Anthropic, Google, Groq, OpenRouter, etc.

## 2. Why does it exist?

You cannot run a 70-billion-parameter model inside your laptop's web app. Providers host the huge model on GPUs; the API is the door to it. Your backend calls it like any other external service.

## 3. Simple Mental Model

It's exactly like calling any backend API (you know this from backend dev) — except the "database" is a giant text predictor. Input: an array of messages. Output: text + metadata (tokens used, finish reason).

## 4. How it Works — Lifecycle

```
1. Your app builds the request  (messages, model, temperature, max_tokens)
2. HTTPS POST to provider endpoint  (with API key)
3. Provider authenticates, rate-checks, queues on GPU
4. Model generates token by token
5. Response returned: text + usage + finish_reason
6. Your app parses, validates, uses the text
```

## 5. ASCII Architecture

```
+--------+   HTTPS    +-----------+        +----------------+
| Client | ---------> |  FastAPI  | -----> |  LLM Provider  |
| (UI)   | <--------- |  backend  | <----- |  (GPU servers) |
+--------+            +-----------+        +----------------+
   JSON JSON              API key kept secret here
```

The API key lives in the BACKEND, never in the browser/frontend.

## 6. Important Terms

- **Endpoint**: the URL you POST to.
- **API key**: secret that identifies and bills you.
- **Request body**: messages array + parameters (model, temperature, max_tokens, stream).
- **finish_reason**: why generation stopped (`stop`, `length`, `tool_call`).
- **usage**: how many tokens input/output — the billing basis.

## 7. What happens internally?

Your messages → tokenized → model inference → tokens streamed/returned → response JSON. Provider handles queuing, GPUs, and scaling — you handle calls, prompts, and failures.

## 8. How is it different?

| | Normal REST API | LLM API |
|---|---|---|
| Response | Fixed, deterministic | Generated, variable |
| Failure types | 4xx/5xx | 4xx/5xx + bad content + slow tokens |
| Latency | Milliseconds | Seconds |
| Cost | ~Free per call | Per token |

## 9. Common Misunderstandings

- ❌ "Call the LLM directly from the frontend." Exposes your key; always proxy via backend.
- ❌ "Same prompt = same answer." Temperature > 0 makes outputs vary.
- ❌ "API call is instant." It takes seconds — design for latency.

## 10. Connection to Previous

Prompt engineering defined WHAT to send. This file defines HOW it travels: the HTTP request and its lifecycle.

## 11. Connection to Next

Inside the request lives a list of **messages with roles** (system, user, assistant) — the structure that makes conversations work.

## 12. One-Minute Explanation

"An LLM API works like any REST API: my backend sends a POST with a messages array and parameters like temperature, using an API key stored only server-side. The provider runs the model on GPUs, generates token by token, and returns text plus usage and finish_reason. Key differences from normal APIs: latency is seconds, cost is per token, and the response is non-deterministic — so I design for validation and failure handling from day one."
