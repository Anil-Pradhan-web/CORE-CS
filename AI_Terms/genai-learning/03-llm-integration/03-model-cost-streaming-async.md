# Model Selection, Token Usage & Cost, Streaming, Sync vs Async

## 1. What is it?

The practical decisions of every integration:
- **Model selection**: which model to use (big vs small, cheap vs smart).
- **Token usage & cost**: you pay per token, in and out.
- **Streaming**: receiving tokens as they're generated instead of all at once.
- **Sync vs async**: blocking a thread while waiting vs continuing other work.

## 2. Why does it exist?

Real apps must balance quality, speed, and money. A giant model is great but slow and expensive; a small model is fast and cheap but weaker. Streaming exists because generation takes seconds — users should see text appear immediately, not stare at a spinner.

## 3. Simple Mental Model

Cost = (input tokens + output tokens) × price per token. Streaming = water flowing from a tap instead of waiting for the full bucket.

## 4. How it Works

**Model selection factors:**

| Factor | Big model | Small model |
|---|---|---|
| Quality | High | Task-dependent |
| Latency | Slow (seconds) | Fast |
| Cost | High | Low |
| Use | Complex reasoning, agents | Classification, extraction, simple chat |

**Cost:** output tokens usually cost ~3–5× input tokens. Long chat histories = recurring input cost on EVERY call.

**Streaming:** provider sends server-sent events (chunks of tokens); your app forwards them to the UI progressively.

**Sync vs async:**

```
SYNC:   request -> [thread waits 3s] -> response      (simple, blocks)
ASYNC:  request -> thread free for other users  -> callback/completes later
```

## 5. ASCII: Streaming vs normal

```
Normal:   [...........wait 4s..........][full answer appears]
Streaming:[word][word][word][word]... appears as generated
```

## 6. Important Terms

- **Token pricing tier**: input/output per-1M-token rates.
- **Latency**: time to first token (TTFT) vs total time.
- **Throughput**: tokens per second.
- **Concurrency**: handling many user requests simultaneously (why async matters for backends).

## 7. What happens internally?

Streaming uses an open HTTP connection; the provider pushes token chunks as the model emits them. Async works because the HTTP call is I/O-bound — your thread isn't computing, just waiting, so it can serve other requests meanwhile.

## 8. How is it different?

| | Streaming | Non-streaming |
|---|---|---|---|
| UX | Feels instant | Feels slow |
| Parsing | Incremental chunks | One complete JSON |
| Good for | Chat UIs | Background jobs, APIs returning JSON |

## 9. Common Misunderstandings

- ❌ "Streaming makes the model faster." Total generation time is the same — it only *feels* faster (first token arrives sooner).
- ❌ "Always use the biggest model." Many production steps (classification, extraction) work great on small/cheap models.
- ❌ "Sync is wrong." For scripts and simple flows sync is perfectly fine; async matters under load.

## 10. Connection to Previous

Previous: message roles and statelessness. This file: the economic and performance dimensions of those same requests.

## 11. Connection to Next

You'll likely use more than one provider (cost, capability). Next: **provider abstraction** — designing so you can swap providers without rewriting your app.

## 12. One-Minute Explanation

"When integrating LLMs I choose models per task: big models for complex reasoning, small ones for cheap high-volume steps. Cost is per token, with output tokens costing more, and chat history re-billed as input tokens every call. Streaming sends tokens as they're generated — it doesn't speed up generation, it just improves perceived latency. On the backend, LLM calls are slow I/O, so I use async to keep the server responsive under concurrency."
