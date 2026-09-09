# GenAI Learning Map — The Complete Story

This folder is ONE continuous story, not 12 separate topics.
Read the folders in number order. Every file tells you why the previous topic matters and what comes next.

## The Big Picture Chain

```
LLM
 |
 v
Prompt
 |
 v
LLM Application
 |
 v
Embeddings
 |
 v
Semantic Search
 |
 v
RAG
 |
 v
Tool Calling
 |
 v
Agent
 |
 v
Agent Orchestration
 |
 v
LangGraph
 |
 v
Multi-Agent Systems
 |
 v
Reliable / Resilient AI Systems
```

## The Story in One Paragraph

A raw LLM is just a text predictor. You control it with **prompts**. To use it in real software, you call it through an **API** inside an application. But the LLM does not know your private data, so you convert text into **embeddings**, search by meaning (**semantic search**), and feed the right knowledge into the prompt — that is **RAG**. The LLM also cannot take real actions, so you give it **tools**. When an LLM can think, use tools, observe results, and decide the next step on its own, it becomes an **agent**. When the logic becomes bigger than one agent, you need **orchestration** (state, nodes, routing). **LangGraph** is a framework that implements orchestration concepts. When one agent is not enough, you build **multi-agent systems**. Finally, because LLM calls fail (timeouts, rate limits, bad output), you add **resilience** — retries, backoff, circuit breakers, failover, fallbacks.

## Folder Map

| Folder | What you learn | Key question it answers |
|---|---|---|
| 00-roadmap | The map itself | "What am I learning and in what order?" |
| 01-llm-fundamentals | AI, ML, Transformers, LLMs, tokens, temperature, hallucination | "What IS this thing I am talking to?" |
| 02-prompt-engineering | Instructions, roles, examples, structured output, chaining | "How do I control the LLM?" |
| 03-llm-integration | APIs, messages, cost, streaming, providers | "How do I connect an LLM to software?" |
| 04-semantic-search | Embeddings, vectors, cosine similarity, vector search | "How do I search by MEANING?" |
| 05-rag | Chunking, vector DB, retrieval, context injection | "How do I give the LLM my knowledge?" |
| 06-tool-calling | Function calling, schemas, execution lifecycle | "How do I let the LLM DO things?" |
| 07-agentic-ai | Agent loop, ReAct, memory, planning | "What makes an LLM an AGENT?" |
| 08-agent-orchestration | Workflows, state, nodes, routing, parallel steps | "How do I control complex AI logic?" |
| 09-langgraph | State, nodes, edges, checkpoints, human-in-the-loop | "How does a framework implement orchestration?" |
| 10-multi-agent-systems | Roles, supervisor, handoff, shared state | "When and how do I use MANY agents?" |
| 11-llm-resilience | Timeouts, retries, backoff, circuit breaker, failover | "How do I keep AI systems alive?" |
| 99-revision | Full mental map, comparisons, interview answers | "Can I explain ALL of it as one system?" |

## How to Read

1. Read folders in order. Do not skip — every folder builds on the last.
2. Every file ends with "One-Minute Explanation" — practice saying it out loud.
3. Focus on the ASCII diagrams. If you can redraw a diagram from memory, you understood the concept.
