# RAG Failure Modes, RAG vs Fine-tuning & Mental Model

## 1. What is it?

A close look at why RAG systems fail in production, when to choose RAG vs fine-tuning, and the folder's one-page mental model.

## 2. Why does it exist?

Most "RAG doesn't work" complaints are actually one of a handful of specific, fixable failure modes.

## 3. Simple Mental Model

RAG is a chain: **right chunk → right prompt → faithful model**. Any broken link = bad answer.

## 4. The Failure Modes

| Failure | What happens | Fix |
|---|---|---|
| Bad chunking | Retrieved text is half-sentences | Better strategy, overlap |
| Retrieval miss | Relevant chunk exists but isn't fetched | Better embeddings, hybrid search, re-ranking |
| Retrieval noise | Many irrelevant chunks drown the good one | Top-k tuning, thresholds, re-rank |
| Embedding mismatch | Docs and query use different models | One model everywhere |
| Stale data | Docs outdated | Re-index pipeline |
| Context overflow | Too many chunks | Limit top-k |
| Ignored context | LLM answers from memory anyway | Strong grounding instruction |
| No "don't know" | Model invents when context lacks the answer | Explicit escape instruction |

## 5. ASCII: Where failures live

```
Docs -> [chunking] -> vectors -> [retrieval] -> [prompt] -> [LLM] -> answer
          FAIL 1                 FAIL 2        FAIL 3     FAIL 4
```

Debug in this order: was the right chunk retrieved? (Check retrieval first — it's the most common culprit.)

## 6. RAG vs Fine-tuning — the decision table

| Need | Choose |
|---|---|
| LLM must know YOUR changing documents | RAG |
| LLM must cite sources | RAG |
| LLM must always output a specific style/format | Fine-tuning |
| LLM must master a niche domain's vocabulary | Fine-tuning (or both) |
| Knowledge changes weekly | RAG, always |
| Best of both | Fine-tune for style + RAG for facts |

## 7. The One-Page RAG Mental Model

```
OFFLINE:  docs -> clean -> chunk -> embed -> vector DB
ONLINE:   question -> embed -> search -> top-k
          -> prompt(rules + labeled context + question)
          -> LLM -> grounded answer + citations
DEBUG:    retrieval first, then prompt, then model
```

## 8. How RAG connects to everything before it

```
LLM calls (integration) + embeddings/vector search (semantic search)
        = RAG
```

RAG is the first "architecture" you've built — two systems composed together.

## 9. Common Misunderstandings

- ❌ "RAG is a product you buy." It's an architecture you design; every knob (chunk, k, ranker) is yours to tune.
- ❌ "If the doc exists, RAG will find it." Only if chunking + embeddings + search are healthy.
- ❌ "RAG removes hallucination." It *reduces* it for retrieved facts; the model can still misread or extrapolate.

## 10. Connection to Previous

Previous: the retrieval-injection-generation flow. This file makes you able to *diagnose* it like an engineer.

## 11. Connection to Next

RAG gives the LLM knowledge — but the LLM still can't *do* anything (check live stock, send email, call your API). Next: **Tool Calling**.

## 12. One-Minute Explanation

"RAG failures cluster in four places: bad chunking, retrieval misses or noise, weak context injection, and the model ignoring or overreaching beyond context. I debug retrieval first. RAG vs fine-tuning is a simple decision: changing facts and citations → RAG; fixed style and domain behavior → fine-tuning; often both. Conceptually RAG is just LLM integration plus semantic search composed into one architecture — and that 'compose simple systems' pattern is what the rest of this curriculum builds on."

## Hinglish Explanation

RAG fail hone ke 4 main kaaran: galat chunking, retrieval miss (correct chunk fetch hi nahi hua), prompt mein sahi context na jana, ya LLM context ignore karke khud se jhooth bolna. Debug hamesha retrieval se shuru karo. Rule yaad rakho: facts/citations chahiye to RAG, style/domain behavior chahiye to fine-tuning — dono saath bhi use ho sakte hain.
