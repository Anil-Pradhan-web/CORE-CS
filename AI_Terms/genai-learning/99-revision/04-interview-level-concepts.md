# Interview-Level Concepts — Say It Like This

Short, confident answers you can deliver verbally.

## Fundamentals

**Q: What is an LLM?**
"A large transformer trained on massive text to predict the next token, repeatedly, until a full response is generated. It's frozen at training time, stateless between calls, and can hallucinate because it predicts plausible text, not verified facts."

**Q: What is temperature?**
"An inference-time setting controlling randomness: low temperature picks the top token — deterministic, good for code and math; high samples wider — creative but unpredictable. It changes randomness, not correctness."

**Q: What is a hallucination and how do you reduce it?**
"The model confidently generating wrong content because it predicts plausibility, not truth. I reduce it with grounding — RAG injecting real retrieved documents — plus structured output, validation, and 'say I don't know' instructions. Reduced, never eliminated."

## Prompting & Integration

**Q: What makes a good prompt?**
"Role, instruction, context, constraints, and output format — a spec, not a question. For consistency I add few-shot examples and delimiters; for machine-readable output I use JSON schema mode and always validate."

**Q: Why is the LLM stateless and how do you handle conversations?**
"The API remembers nothing between calls — my backend stores the history and re-sends the messages array every request. Input tokens and cost grow with the conversation, so I trim or summarize old history."

**Q: Why stream responses?**
"To improve perceived latency — first tokens appear almost immediately instead of waiting seconds for the full answer. Total generation time is unchanged."

## Search & RAG

**Q: What are embeddings?**
"Vectors from an embedding model where similar meanings point in similar directions. Search becomes cosine-similarity nearest-neighbor lookup instead of keyword matching, so synonyms and paraphrases are found."

**Q: Explain RAG end-to-end.**
"Offline: chunk documents into ~200-800 token overlapping pieces, embed them, store vectors plus metadata in a vector DB. Online: embed the question, retrieve top-k chunks — optionally re-ranked — inject them into the prompt with an 'answer only from context' instruction, and generate a grounded, citable answer. It exists because LLMs lack private/recent knowledge and hallucinate. I debug RAG by checking retrieval first."

**Q: RAG vs fine-tuning?**
"RAG changes the context — instant knowledge updates, citations, cheapest way to add facts. Fine-tuning changes parameters — best for style and domain behavior, but poor for facts. Production often uses both."

**Q: RAG vs long context?**
"Long context pays for every token on every call and degrades with 'lost in the middle'. RAG retrieves only relevant chunks — cheaper, focused, and scales beyond any window."

## Tools & Agents

**Q: How does tool calling work?**
"I register tool schemas — name, description, parameters. The LLM never executes anything: it outputs a structured tool call, my backend validates the arguments, executes, and returns the result as a message; then the model composes the final answer. Validation and authorization are always server-side."

**Q: What makes something an agent?**
"Autonomy in a loop: an LLM with a goal, tools, and memory that decides its own next action, observes results via ReAct — reason, act, observe — and repeats until the goal is met or a max-steps limit hits. Each iteration is a billed LLM call, so stop conditions are mandatory."

**Q: Memory vs context?**
"Context is what fits in the request's token window; memory is what my app persists — short-term task history, long-term facts in a vector store retrieved semantically."

## Orchestration, LangGraph, Multi-Agent

**Q: Workflow vs agent?**
"A workflow's steps are predefined by me — predictable, cheap, debuggable. An agent chooses its own steps — flexible but costly and unpredictable. I use the most predictable option that solves the problem; production systems are usually workflows containing agents."

**Q: What is orchestration?**
"The control layer owning state — the shared data between steps; nodes — the steps; routing — which node runs next, fixed or conditional; parallel fan-out/fan-in; error paths with retries; and human-in-the-loop pauses that persist state and resume after approval."

**Q: What does LangGraph give me?**
"Orchestration concepts as a framework: state schema with reducers, nodes as functions, fixed and conditional edges, parallel branches, checkpoints that snapshot state every step, and interrupts for human approval with resume. I don't need it for simple flows — I use it when I need branches, loops, resumable state, and inspectable structure."

**Q: Why (or why not) multi-agent?**
"Specialized agents — each with a focused role prompt and tools — coordinated by a supervisor that routes in a loop, handoffs that pass control peer-to-peer, or a code orchestrator, over shared state merged by reducers. The honest answer: more agents don't mean better AI — each adds cost, latency, and coordination failure modes. I go multi-agent only for genuine specialization, context limits, or valuable parallel opinions."

## Resilience

**Q: How do you make LLM calls production-ready?**
"A pipeline per call: timeout, error classification — retry only transient errors like 429 and 5xx, never 400/401 — retries with exponential backoff and jitter, a circuit breaker per provider that fails fast and auto-recovers via half-open probes, failover to backup providers through my abstraction layer, a non-LLM fallback when everything fails, and validation of every response because 200-OK can still contain garbage. Plus logging of latency, tokens, cost, and agent trajectories."

**Q: Retry vs circuit breaker?**
"Retries handle single transient hiccups with spaced attempts. The breaker watches aggregate failures: after a threshold it opens and rejects instantly, then half-open probes to recover — preventing retry storms against a dead provider. They're complementary layers."

**Q: Failover vs fallback?**
"Failover switches to another LLM provider — same quality tier. Fallback is the degraded non-LLM plan — cached or template answers — when no provider works."

## The Closing Statement

"Everything I've described is one pipeline: a user request flows through an application into an orchestrator that routes it across specialized agents; agents use RAG for knowledge and tools for actions, backed by LLM calls; every call is validated, retried, and failed over; and everything is logged. Understanding each box — and why it exists — is what lets me read and design any modern GenAI architecture."

