# Retrieval, Context Injection & Generation

## 1. What is it?

The query-time half of RAG:
- **Retrieval**: fetch the top-k most relevant chunks for the user's question.
- **Context injection**: paste those chunks into the prompt with clear labels.
- **Generation**: the LLM answers the question using ONLY that context.

## 2. Why does it exist?

Retrieval finds knowledge; injection makes the LLM see it; generation turns it into an answer. Each step is a separate place where quality can rise or fall — so they're understood separately.

## 3. Simple Mental Model

A lawyer's clerk (retriever) pulls the right case files, the lawyer (LLM) reads only those files and answers. If the clerk pulls wrong files, the lawyer confidently cites the wrong cases.

## 4. How it Works

**Retrieval:** question → embed → ANN search → top-k chunks (optionally re-rank, filter).

**Context injection — the prompt template:**

```
You are a support assistant. Answer ONLY from the context below.
If the answer is not in the context, say "I don't know."

Context:
[1] (source: refunds.md, page 3) Refunds are processed within 7 days...
[2] (source: faq.md) Refund requests must be raised within 30 days...

Question: How long do refunds take?
```

**Generation:** normal LLM call — but now grounded. Good systems return citations `[1]`.

## 5. ASCII: Query-time flow

```
User question
     |
     v
Embed question
     |
     v
Vector search (+filters)
     |
     v
Top-k chunks  ---> (optional re-rank -> keep best 3)
     |
     v
Prompt = rules + labeled context + question
     |
     v
LLM
     |
     v
Answer + citations
```

## 6. Important Terms

- **Top-k**: how many chunks to inject (3–8 typical).
- **Re-ranking**: a second, smarter model scores the top-k to reorder — better precision at small extra cost.
- **"Answer only from context"**: the grounding instruction that fights hallucination.
- **Citation**: mapping each answer claim to a source chunk.

## 7. What happens internally?

Injected chunks sit in the context window; attention lets the model read them as evidence. The grounding instruction shifts probability toward faithful, sourced answers.

## 8. How is it different?

| | Plain LLM call | RAG generation |
|---|---|---|
| Knowledge | Frozen training data | Fresh retrieved docs |
| Hallucination risk | High | Low (still not zero) |
| Traceability | None | Citations |
| Latency/cost | Low | +1 retrieval step |

## 9. Common Misunderstandings

- ❌ "The model always uses the provided context." It can still drift — instructions reduce, never eliminate.
- ❌ "More chunks = better answers." Too many = noise + cost + "lost in the middle".
- ❌ "RAG is only retrieval + prompt." Production RAG adds re-ranking, evaluation, and failure handling.

## 10. Connection to Previous

Previous: chunks stored in a vector DB. This file: fetching them and turning them into answers — RAG's payoff moment.

## 11. Connection to Next

RAG often disappoints in practice. Knowing *where it fails* is the next skill.

## 12. One-Minute Explanation

"At query time, RAG embeds the question, retrieves top-k chunks — sometimes re-ranked for precision — injects them into the prompt with labels and a 'answer only from context' instruction, and generates a grounded answer with citations. The retrieval step is the clerk, the LLM is the lawyer: if retrieval is wrong, generation is confidently wrong. That's why I tune chunk size, top-k, and re-ranking as one system."
