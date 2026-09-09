# Common Confusions — Cleared One by One

Each confusion: the wrong idea, the correct idea, and a one-liner to remember.

## 1. "LLM = Generative AI"
- **Wrong**: they're the same.
- **Right**: Generative AI = any content-generating AI (images, audio). LLM = the text subset.
- **Remember**: LLM ⊂ Generative AI.

## 2. "An agent is a smart LLM"
- **Wrong**: better model = agent.
- **Right**: agent = LLM + goal + tools + memory + loop. Same model, plus autonomy structure.
- **Remember**: loop + tools + goal = agent.

## 3. "Prompt and system prompt are the same"
- **Right**: system prompt is a special role set by the developer app-wide; a prompt is any input message.
- **Remember**: system = app's rules; user = human's words.

## 4. "Prompt engineering vs fine-tuning — when?"
- **Right**: prompts control behavior per request (free); fine-tuning bakes style/domain into parameters (costly). Facts → RAG, not fine-tuning.
- **Remember**: behavior → prompt; style → fine-tune; facts → RAG.

## 5. "Embedding and token are similar"
- **Right**: token = text chunk (generation unit); embedding = meaning vector (search unit).
- **Remember**: tokens for talking, embeddings for searching.

## 6. "Embedding = vector database"
- **Right**: embeddings are data; a vector DB stores and searches them at scale.
- **Remember**: embedding = photo, vector DB = photo album with search.

## 7. "Semantic search vs keyword search"
- **Right**: keyword matches spelling; semantic matches meaning via vectors. Hybrid combines both.
- **Remember**: keyword = Ctrl+F; semantic = "find me things like this".

## 8. "RAG vs fine-tuning"
- **Right**: RAG injects knowledge at runtime (citable, instantly updatable); fine-tuning changes the model (style, no citations).
- **Remember**: changing facts → RAG; fixed style → fine-tune; often both.

## 9. "Long context removes the need for RAG"
- **Right**: long context is expensive per call and degrades ("lost in the middle"); RAG retrieves only what's needed and scales to millions of docs.
- **Remember**: big window ≠ focused reading.

## 10. "Tool calling = agent"
- **Right**: tool calling is one decision (call this function); an agent is a loop of think→act→observe.
- **Remember**: tool call = one swing; agent = the whole boxing match.

## 11. "Agent vs workflow"
- **Right**: workflow steps are predefined by you (predictable); agent steps are chosen by the model (flexible, riskier).
- **Remember**: script vs improviser. Prefer the script when possible.

## 12. "Workflow vs orchestration"
- **Right**: workflow is the process; orchestration is the managing layer (state, routing, retries, HITL) that runs workflows.
- **Remember**: workflow = the play; orchestrator = the director.

## 13. "Single agent vs multi-agent — more is better?"
- **Right**: multi-agent adds cost, latency, and coordination failures. Only for real specialization, context limits, or valuable parallel opinions.
- **Remember**: more agents ≠ better AI.

## 14. "Sequential vs parallel agents"
- **Right**: sequential when each step needs the previous output; parallel for independent work, merged at fan-in.
- **Remember**: assembly line vs three workers on separate parts.

## 15. "LangGraph = LangChain"
- **Right**: separate libraries. LangChain = components/chains; LangGraph = stateful graphs with checkpoints and interrupts.
- **Remember**: chains for linear, graphs for stateful control.

## 16. "State = memory"
- **Right**: state = shared data within one run; memory = retention across steps/sessions (short-term history, long-term vector store).
- **Remember**: state = this run's whiteboard; memory = the filing cabinet.

## 17. "Memory = context"
- **Right**: context = what's in the token window THIS request; memory = what the app stores and re-injects.
- **Remember**: memory is managed by your app; context is limited by the model.

## 18. "Retry = failover"
- **Right**: retry = same provider again (transient errors); failover = different provider (persistent failure).
- **Remember**: knock again vs try another shop.

## 19. "Retry = circuit breaker"
- **Right**: retry handles one failed call; breaker stops ALL calls to a dead provider after a threshold, then auto-probes recovery.
- **Remember**: retry = one knock; breaker = "shop closed, come back later" board.

## 20. "Failover = fallback"
- **Right**: failover = another LLM provider; fallback = non-LLM degraded plan (cache/template).
- **Remember**: spare tyre vs calling a cab.

## 21. "Model error = system error"
- **Right**: system error = call fails (timeout/429/500) → retry/failover. Model error = bad content → prompts/RAG/validation.
- **Remember**: plumbing problems vs bad cooking.

## 22. "HTTP 200 means the answer is good"
- **Right**: garbage content also returns 200 — always validate parse/fields/sanity.
- **Remember**: delivered ≠ edible.
