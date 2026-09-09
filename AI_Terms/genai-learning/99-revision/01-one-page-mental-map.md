# One-Page Mental Map — The Entire Curriculum

## The Story

```
LLM (next-token predictor)
  -> controlled by PROMPTS
  -> called from software via API (LLM APPLICATION)
  -> but doesn't know your data -> EMBEDDINGS make text comparable by meaning
  -> SEMANTIC SEARCH finds similar text
  -> RAG = retrieve + inject + generate (open-book answering)
  -> but LLM can't ACT -> TOOL CALLING (LLM decides, your code executes)
  -> TOOL LOOP + memory + goal = AGENT
  -> many steps need control -> ORCHESTRATION (state, nodes, routing)
  -> ORCHESTRATION framework = LANGGRAPH (graph + checkpoints + interrupts)
  -> specialized agents = MULTI-AGENT SYSTEMS (supervisor/handoff)
  -> calls fail -> RESILIENCE (retry, backoff, breaker, failover, fallback)
```

## The Master Diagram

```
                         USER
                           |
                           v
                      APPLICATION
                           |  (prompts, history, provider abstraction)
                           v
                       ORCHESTRATOR
                           |  (state, routing, parallel, HITL)
             +-------------+-------------+
             |             |             |
             v             v             v
          AGENT A       AGENT B       AGENT C
       (research)     (code)        (review)
             |             |             |
             +------+------+------+------+  shared state, reducers merge
                           |
                           v
                     TOOL / RAG / API
              (functions execute, vectors retrieved)
                           |
                           v
                          LLM
              (generates token by token)
                           |
                           v
                      VALIDATION
              (parse, fields, sanity check)
                           |
                           v
                 RETRY / FAILOVER
        (backoff, circuit breaker, next provider)
                           |
                           v
                       RESPONSE
                    (+ logging everywhere)
```

## One-Line Recall for Each Topic

| # | Topic | One line |
|---|---|---|
| 1 | LLM | Transformer predicting next tokens; frozen knowledge; hallucinates |
| 2 | Prompt | The spec: role + instruction + context + constraints + format |
| 3 | Integration | HTTP API, message roles, stateless, tokens cost money, streaming |
| 4 | Embeddings | Text → vector; meaning = direction |
| 5 | Semantic search | Cosine similarity + top-k + ANN index (FAISS/vector DB) |
| 6 | RAG | Chunk → embed → store → retrieve → inject → generate (open book) |
| 7 | Tool calling | LLM outputs {function, args}; YOUR code executes; result back |
| 8 | Agent | Goal + tools + memory + loop (think→act→observe) + stop condition |
| 9 | Orchestration | State + nodes + routing + parallel + error paths + HITL |
| 10 | LangGraph | Same concepts as graph: state/nodes/edges/checkpoints/interrupts |
| 11 | Multi-agent | Specialized agents + supervisor/handoff + shared state |
| 12 | Resilience | Timeout → retry/backoff → breaker → failover → fallback → validate |

## The Five Questions That Unlock Everything

1. **Why RAG?** LLM doesn't know your data and hallucinates → retrieve real facts into the prompt.
2. **Why tools?** LLM can only talk → let it REQUEST actions your code performs.
3. **What makes an agent?** Autonomy: the LLM chooses its own next steps in a loop.
4. **Why orchestration?** Multi-step systems need state, routing, and error paths designed by you.
5. **Why resilience?** Every call is a fallible network request with non-deterministic output.

## Study Tip

Redraw the master diagram from memory. Then explain each box in one sentence, and how each box passes data to the next. If you can do that, you own this curriculum.
