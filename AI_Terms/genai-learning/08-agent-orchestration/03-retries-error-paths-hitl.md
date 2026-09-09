# Retries, Error Paths & Human-in-the-Loop

## 1. What is it?

The reliability layer of orchestration:
- **Error paths**: what the flow does when a step fails (retry, skip, branch to recovery, abort).
- **Human-in-the-loop (HITL)**: pausing the workflow at chosen points for a human to approve, edit, or reject.

## 2. Why does it exist?

LLM calls fail (timeouts, rate limits, bad output). Steps produce wrong results. Some actions are too risky to automate (refunds, emails to customers, payments). A production workflow must decide all of this IN ADVANCE — not crash at 2 AM.

## 3. Simple Mental Model

Error paths = fire exits planned before the fire. HITL = a supervisor sign-off before the company sends an official letter.

## 4. How Error Handling Works

```
For EVERY node, decide:
  - transient error?  (timeout, rate limit) -> RETRY (with backoff)
  - permanent error?  (bad args, auth)      -> ERROR PATH (don't retry)
  - low confidence?   (empty retrieval)     -> FALLBACK path
  - all failed?                             -> ABORT or notify human
```

```
[draft reply] --fail--> [retry x2] --fail--> [notify human] --> END
```

## 5. ASCII: Workflow with error + approval branches

```
        +----------+
        | classify |
        +----+-----+
             v
        +----------+   fail    +---------+
        | retrieve |---------> | retry   |--fail--> alert human
        +----+-----+           +---------+
             v
        +----------+
        |  draft   |
        +----+-----+
             v
      urgent? --yes--> [WAIT for human approval] --approved--> send
             |                                --rejected --> redraft
             no
             v
           send
```

## 6. Important Terms

- **Transient vs permanent error**: retryable (timeout, 429) vs not (invalid input, 401).
- **Retry with backoff**: wait longer between retries (full treatment in the resilience folder).
- **Checkpoint/pause**: saving state so the workflow can stop and RESUME later (days later, after approval).
- **HITL patterns**: approve/reject, edit-and-continue, choose-between-options.
- **Interrupt/resume**: the mechanism of pausing a run and continuing from saved state.

## 7. What happens internally?

The orchestrator wraps each node with error handling and, at HITL points, persists the state to a store and stops. When a human responds (via UI/API), the run resumes from the saved state.

## 8. How is it different?

| | Retry | Fallback | HITL |
|---|---|---|---|
| On failure | Try again | Degrade gracefully | Ask a human |
| Automation | Full | Full | Pauses |
| Example | Timeout → retry 2× | LLM down → template reply | Big refund → manager approves |

## 9. Common Misunderstandings

- ❌ "Retries fix everything." Retrying a permanent error just wastes money — classify errors first.
- ❌ "HITL is a human watching everything." It's targeted pauses at high-risk nodes only.
- ❌ "Waiting for approval is simple sleep." It requires persisted state + resume — non-trivial engineering (frameworks help here).

## 10. Connection to Previous

Previous: happy-path mechanics (state, nodes, routing). This file: the unhappy paths — what separates a demo from a production system.

## 11. Connection to Next

You now understand orchestration as pure concepts. LangGraph is simply a framework that implements exactly these concepts — state, nodes, edges, routing, checkpoints, interrupts.

## 12. One-Minute Explanation

"Every orchestrated node needs a planned failure path: transient errors like timeouts and rate limits get retried with backoff, permanent errors branch to recovery or human alerting, and risky steps pause for human-in-the-loop approval. HITL means persisting the state, stopping the run, and resuming when a human approves, edits, or rejects — it's targeted sign-offs at high-risk nodes, not constant supervision. These unhappy paths are what make a demo into a production system."

## Hinglish Explanation

Har step ka failure plan pehle se hona chahiye: timeout ya rate limit = retry (thoda wait karke), permanent error = alag error path ya human ko batao, risky action (bada refund, email bhejna) = human approval — workflow pause ho jata hai, state save hoti hai, insaan approve kare to resume. Isko hi Human-in-the-Loop kehte hain. Demo aur production ka yahi difference hai.
