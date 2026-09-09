# How The Topics Connect

Each topic exists ONLY because the previous topic has a limitation. This is the single most important idea in this curriculum.

## The Limitation Chain

```
[LLM alone]
  Limitation: You cannot control it well.
       |
       v  Fix: PROMPTS
[Prompted LLM]
  Limitation: It is just a chat, not software.
       |
       v  Fix: LLM INTEGRATION (API)
[LLM inside an app]
  Limitation: It does not know YOUR data.
       |
       v  Fix: EMBEDDINGS + SEMANTIC SEARCH
[LLM that can search by meaning]
  Limitation: Search alone does not answer questions.
       |
       v  Fix: RAG
[LLM that answers from your documents]
  Limitation: It can only TALK, not ACT.
       |
       v  Fix: TOOL CALLING
[LLM that can act]
  Limitation: It does one step, then stops.
       |
       v  Fix: AGENTIC AI (agent loop)
[Agent that thinks and acts in steps]
  Limitation: Complex logic is messy in plain code.
       |
       v  Fix: ORCHESTRATION (state, nodes, routing)
[Orchestrated workflow]
  Limitation: You rebuild the same plumbing every time.
       |
       v  Fix: LANGGRAPH
[Framework-based orchestration]
  Limitation: One agent has too many jobs.
       |
       v  Fix: MULTI-AGENT SYSTEMS
[Team of specialized agents]
  Limitation: LLM calls fail in the real world.
       |
       v  Fix: RESILIENCE
[Reliable, production-ready AI system]
```

## Why This Order Matters

- You cannot understand **RAG** without knowing **embeddings**.
- You cannot understand **agents** without knowing **tool calling**.
- You cannot understand **LangGraph** without understanding **orchestration concepts** (state, nodes, edges) first.
- You cannot design **multi-agent systems** without knowing how a single agent works.
- **Resilience** comes last because it wraps around everything.

## One-Line Summary of Every Topic

| Topic | One line |
|---|---|
| LLM | A model that predicts the next token |
| Prompt | The input you design to control the LLM |
| LLM Integration | Calling the LLM from software via API |
| Embeddings | Text converted to number-vectors that carry meaning |
| Semantic Search | Finding text by meaning, not keywords |
| RAG | Search your data first, then let the LLM answer using it |
| Tool Calling | LLM asks your code to run a function |
| Agent | LLM + tools + loop (think, act, observe, repeat) |
| Orchestration | Managing state and steps of an AI workflow |
| LangGraph | A framework that models workflows as graph + state |
| Multi-Agent | Multiple specialized agents coordinated together |
| Resilience | Retrying, switching providers, validating output |

## Final Diagram Preview

By the end you will be able to read and explain this:

```
User -> App -> Orchestrator -> Agents -> Tools/RAG -> LLM
                                                      |
                                    Validation <- Retry/Failover
                                                      |
                                                   Response
```

The last file (99-revision) draws this fully.
