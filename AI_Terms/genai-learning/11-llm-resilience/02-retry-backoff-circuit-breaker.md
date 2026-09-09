# Retries, Exponential Backoff & Circuit Breaker

## 1. What is it?

- **Retry**: automatically trying a failed (transient) call again.
- **Exponential backoff**: doubling the wait between retries (1s → 2s → 4s...) so you don't hammer a struggling provider.
- **Circuit breaker**: after repeated failures, STOP calling that provider for a cool-down period — fail fast, let it recover.

## 2. Why does it exist?

Transient failures often resolve in moments — a retry succeeds. But naive instant retries hammer an overloaded provider and make things worse (and can extend your rate-limit punishment). The circuit breaker stops you from repeatedly hitting a dead service and wasting latency on every doomed call.

## 3. Simple Mental Model

```
Retry            = knocking again
Backoff          = waiting longer between knocks
Circuit breaker  = the door's sign: "CLOSED - come back later"
```

## 4. How it Works

**Retry + backoff:**
```
attempt 1 -> fail -> wait 1s
attempt 2 -> fail -> wait 2s
attempt 3 -> fail -> wait 4s
attempt 4 -> fail -> GIVE UP -> failover/fallback
(+ jitter: small random extra delay so many clients
  don't retry at the same instant)
```

**Circuit breaker states:**
```
CLOSED      -> calls pass through normally
   | (failures >= threshold, e.g. 5 in a row)
   v
OPEN        -> calls fail INSTANTLY (no attempt)
   | (after cool-down, e.g. 30s)
   v
HALF-OPEN   -> one test call allowed
   | success            | fail
   v                    v
CLOSED              back to OPEN
```

## 5. ASCII: Request journey with all three

```
Request
  |
  v
[Circuit: OPEN?] --yes--> fail fast -> failover
  |no
  v
Attempt 1 --fail--> backoff 1s --> Attempt 2 --fail--> backoff 2s
                                        |
                                    attempts exhausted
                                        |
                                        v
                          failover / fallback / error
```

## 6. Important Terms

- **Jitter**: random extra delay in backoff (prevents synchronized retry storms).
- **Failure threshold**: how many failures trip the breaker.
- **Cool-down / recovery time**: how long the breaker stays open.
- **Half-open probe**: the single test call when recovering.

### Retry vs Circuit Breaker (confusion!)

| | Retry | Circuit breaker |
|---|---|---|
| Scope | One call | Whole provider/dependency |
| Question | "Did this call fail?" | "Is this service healthy at all?" |
| Prevents | Single missed request | Stampeding a dead service |

Retry and breaker work TOGETHER: retries handle single hiccups; the breaker stops retry storms when a provider is truly down.

## 7. What happens internally?

Your client wraps each call: breaker checks state → attempt → classify the error (transient vs permanent) → on transient, schedule a backoff retry; count failures; trip or reset the breaker.

## 8. How is it different?

| | Retry | Failover | Fallback |
|---|---|---|---|
| Action | Same provider again | Different provider | Non-LLM plan (cache/template) |
| Trigger | Transient error | Persistent failure | Everything failed |

## 9. Common Misunderstandings

- ❌ "Retry everything." Only transient errors (429, 429, timeout, 5xx); never 400/401.
- ❌ "Immediate retries are fine." Without backoff you amplify the outage.
- ❌ "Circuit breaker = giving up." It recovers automatically via half-open probing.

## 10. Connection to Previous

Previous: the failure landscape (timeouts, rate limits). This file: the first two tools to survive it.

## 11. Connection to Next

When retries aren't enough — the provider is DOWN — you need another provider: **failover**.

## 12. One-Minute Explanation

"Resilience starts with retrying transient errors — timeouts, 429s, 5xx — never permanent ones like 400 or 401. Retries use exponential backoff with jitter: wait 1s, 2s, 4s — so you don't stampede a struggling provider. The circuit breaker sits above retries: after a threshold of failures it opens and fails calls instantly, then half-open probes with a single test call to auto-recover. Retries handle single hiccups; the breaker prevents retry storms against a dead provider — they're complementary, not alternatives."

## Hinglish Explanation

Retry matlab dobara try karna, par sirf transient errors (timeout, 429, 5xx) pe — 400/401 permanent hain, unhe retry karna paisa barbaad. Exponential backoff matlab har retry se pehle double wait (1s, 2s, 4s) — taaki down service pe tornado na karo. Circuit breaker matlab "band darwaze ka board": 5 baar fail hone ke baad calls turant fail karo, thodi der baad ek test call se check karo ki provider wapas zinda hai ya nahi.
