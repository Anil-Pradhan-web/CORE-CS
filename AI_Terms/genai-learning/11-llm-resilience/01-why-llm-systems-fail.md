# Why LLM Systems Fail: Timeouts & Rate Limits

## 1. What is it?

The real-world failure sources of any LLM-powered system:
- **Timeout**: you stop waiting after N seconds even if no response came.
- **Rate limit**: the provider rejects requests beyond your quota (e.g., "too many requests" — HTTP 429).
- Plus: network errors, provider outages, invalid outputs, cost overruns.

## 2. Why does it exist?

Every LLM call is a network request to a busy, GPU-constrained external service. Unlike your local functions, it can be slow (seconds to minutes), rejected (limits), or fully down. A system that assumes "the call will succeed" will fall over the first busy minute.

## 3. Simple Mental Model

Ordering food delivery: sometimes it's late (timeout), sometimes the restaurant is at capacity (rate limit), sometimes the restaurant is closed (outage). A good customer has a plan for each — that plan is resilience.

## 4. How it Works

**Timeout:**
```
request sent -> [timer 30s] -> no response? -> ABORT and handle
too short  = aborting requests that would have succeeded
too long   = users hang, threads block
typical:   10-60s depending on task (streaming changes this)
```

**Rate limit:**
```
quota exceeded (429 Too Many Requests)
   |
   v
handle by: waiting + retrying later (respecting
Retry-After header), queuing, or switching provider
```

## 5. ASCII: Failure sources in one picture

```
Your app -> [network drop] -> [provider overload] -> [429 rate limit]
                                      |
                                 [slow model, long generation]
                                      |
                                 [timeout!]
                                      |
                                 [bad/invalid output]
                                      |
                                 [validation fails]
```

## 6. Important Terms

- **429 / 5xx**: rate-limit / server errors (transient — retryable).
- **400 / 401 / 403**: bad request / auth errors (permanent — do NOT retry).
- **Retry-After header**: provider's hint on how long to wait.
- **Concurrency limits**: max parallel requests allowed.
- **Model error vs system error** (interview classic):

| | Model error | System error |
|---|---|---|
| What | Wrong/bad CONTENT in response | Call itself fails (timeout, 429, 500) |
| Layer | Semantic (model quality) | Infrastructure |
| Fix | Prompts, RAG, validation, fallback answer | Retry, backoff, failover |

## 7. What happens internally?

Providers enforce quotas per key/minute; when exceeded, requests are rejected before reaching the GPU. Timeouts live in YOUR HTTP client — the provider may still be working when you give up.

## 8. How is it different?

| | Timeout | Rate limit |
|---|---|---|
| Cause | Response too slow | Too many requests |
| Fix | Adjust timeout, stream | Back off, queue, spread load |
| Retryable? | Often (may succeed) | Yes — after waiting |

## 9. Common Misunderstandings

- ❌ "Retrying a 429 immediately helps." It makes rate limiting WORSE — back off properly.
- ❌ "Default timeouts are fine." Defaults are often 60s+ — too long for a user-facing chat.
- ❌ "Retrying a 400 might work." Bad request/auth errors are permanent; retrying wastes money.

## 10. Connection to Previous

Multi-agent systems multiply LLM calls — multiply failure chances. Resilience is the discipline of surviving them.

## 11. Connection to Next

First defensive mechanism: **retries** — and how to space them (exponential backoff).

## 12. One-Minute Explanation

"LLM calls fail for infrastructure reasons — timeouts, network drops, rate limits (HTTP 429), provider outages — and for quality reasons — bad content that fails validation. I distinguish model errors from system errors: content problems are fixed with prompts, RAG, and validation; infrastructure problems are fixed with timeouts, retries, backoff, and failover. And I never retry permanent errors like 400/401 — only transient ones like 429 and 5xx, always with backoff."

## Hinglish Explanation

LLM call ek network request hai busy external service par — isliye timeout (response na aana), 429 (rate limit — bahut zyada requests), aur provider down ho sakta hai. Do alag tarah ke errors yaad rakho: system error (call hi fail — retry/failover se fix) aur model error (jawab ka content galat — prompt/RAG/validation se fix). 400/401 permanent hote hain — unhe retry mat karo.
