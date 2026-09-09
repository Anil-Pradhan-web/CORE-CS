# Provider Abstraction + LLM Integration Mental Model

## 1. What is it?

**Provider abstraction**: designing your app so the LLM provider (OpenAI / Anthropic / Gemini / Groq / OpenRouter) sits behind one interface — you can swap providers by changing config, not rewriting code.

## 2. Why does it exist?

Providers have different APIs, pricing, rate limits, and outage risks. Hard-coding one provider couples your whole product to their uptime and prices. Abstraction gives you: easy swaps, failover (resilience folder), cost optimization, and vendor independence.

## 3. Simple Mental Model

Like a power adapter: your app speaks ONE plug format; an adapter converts it to any wall socket. Or like a repository pattern in backend dev — business logic never talks to the DB driver directly.

## 4. How it Works (conceptually)

```
Your app code  ->  your own LLMService interface
                        (chat(messages, options) -> response)
                        |
        +---------------+----------------+
        |               |                |
   OpenAIAdapter   GeminiAdapter   GroqAdapter
```

Each adapter translates the same logical call into that provider's request format.

## 5. ASCII: Where abstraction sits

```
Agents / RAG / App logic
          |
          v
   LLMService (interface)
          |
    +-----+------+------+
    v     v      v      v
 OpenAI Anthropic Groq  OpenRouter
```

## 6. Important Terms

- **Adapter pattern**: a class wrapping one provider behind a common interface.
- **OpenRouter-style gateway**: a service that already unifies many providers behind one API.
- **Capability matrix**: which provider supports tools, schema output, streaming, context size, cost.

## 7. What happens internally?

Your interface receives a logical request → the chosen adapter maps parameters (model name, temperature, tools) to the provider's format → sends → maps the response back to your common shape.

## 8. How is it different?

| | Direct SDK calls | Abstracted layer |
|---|---|---|
| Speed to build | Faster initially | Slightly more setup |
| Provider swap | Rewrite everywhere | Config change |
| Failover possible | Hard | Natural (resilience!) |
| Testing | Real calls needed | Inject a mock adapter |

## 9. Common Misunderstandings

- ❌ "Abstraction hides provider-specific powers." Good interfaces expose optional capability flags instead.
- ❌ "It's unnecessary early." It's cheap to add at the start and painful to retrofit when the first outage hits.
- ❌ "Abstraction = resilience." Abstraction ENABLES failover; resilience logic (retries, breakers) is a separate layer — next folder topics build on this.

## 10. Connection to Previous

Previous: the mechanics of calls (roles, cost, streaming, async). Abstraction wraps all of that behind one clean interface.

## 11. Connection to Next

The LLM knows nothing about your data. To fix that, text must become searchable-by-meaning — **embeddings** (semantic search folder).

## 12. One-Minute Explanation

"Provider abstraction means my app depends on my own LLMService interface, with one adapter class per provider translating requests. Like the repository pattern for databases, this decouples business logic from vendor APIs — so I can switch providers, route by cost or capability, and later add failover, without touching app logic. It's the foundation that resilience mechanisms like failover are built on."
