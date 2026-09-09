# Key Terminology (Quick Glossary)

Short, simple definitions. Full explanations come in their own files.

## Model Basics

| Term | Simple meaning |
|---|---|
| AI | Machines doing tasks that need human-like thinking |
| ML | Learning patterns from data instead of hard-coded rules |
| Deep Learning (DL) | ML using many-layered neural networks |
| Neural Network | A stack of math layers that transforms input to output |
| Transformer | The neural network architecture behind all modern LLMs |
| LLM | A huge transformer trained to predict the next token |
| Token | A small chunk of text (word piece) the LLM reads/writes |
| Context Window | Max tokens the model can "see" at once |
| Parameters | The learned numbers inside the model (its "knowledge") |
| Temperature | A setting that controls randomness of output |
| Pre-training | Learning language from massive internet text |
| Fine-tuning | Extra training on specific data to specialize the model |
| Inference | Using a trained model to generate output |
| Hallucination | Confidently generating wrong information |

## Prompting

| Term | Simple meaning |
|---|---|
| Prompt | The text input you send to the LLM |
| System Prompt | Hidden instructions that set behavior/role |
| User Message | What the end user types |
| Few-shot | Showing examples inside the prompt |
| Structured Output | Forcing output into a fixed format (like JSON) |
| Prompt Chaining | Multiple prompts connected in sequence |

## Integration & Search

| Term | Simple meaning |
|---|---|
| API | A way for software to call the LLM over the network |
| Message Roles | system / user / assistant labels in a chat request |
| Streaming | Receiving tokens as they are generated |
| Embedding | Text turned into a list of numbers (vector) with meaning |
| Vector | An ordered list of numbers, e.g. [0.2, -1.1, 0.4, ...] |
| Cosine Similarity | A score (1 to -1) of how similar two vectors point |
| Vector Store / Vector DB | A database built to store and search vectors |
| FAISS | A fast library for vector similarity search |

## RAG & Tools

| Term | Simple meaning |
|---|---|
| RAG | Retrieval-Augmented Generation: retrieve docs, then generate |
| Chunking | Splitting documents into small pieces |
| Retrieval | Finding the most relevant chunks for a query |
| Context Injection | Putting retrieved text into the prompt |
| Tool / Function Calling | LLM outputs a request to run your function |
| Tool Schema | The name + description + parameter list of a tool |
| Tool Result | The data your function returns back to the LLM |

## Agents & Orchestration

| Term | Simple meaning |
|---|---|
| Agent | LLM + goal + tools + loop (decides its own next step) |
| ReAct | Reason then Act, step by step, in a loop |
| Agent Loop | Think -> Act -> Observe -> repeat |
| Memory | Information an agent keeps across steps |
| State | The shared data a workflow carries between steps |
| Node | One step in a workflow (a function) |
| Edge | The path from one node to the next |
| Conditional Edge | An edge chosen at runtime based on state |
| Orchestrator | The coordinator that routes work between steps |
| Checkpoint | A saved snapshot of state (for pause/resume/recovery) |
| Human-in-the-Loop | Pausing a workflow for human approval |

## Multi-Agent & Resilience

| Term | Simple meaning |
|---|---|
| Multi-Agent System | Several specialized agents working together |
| Supervisor | An agent that routes work to other agents |
| Handoff | Passing control from one agent to another |
| Timeout | Max time you wait before giving up on a call |
| Rate Limit | Max requests allowed per minute by a provider |
| Retry | Trying a failed call again |
| Exponential Backoff | Waiting longer between each retry |
| Circuit Breaker | Temporarily stop calling a failing provider |
| Failover | Switching to a backup provider on failure |
| Fallback | A degraded but available plan (e.g., cached answer) |
| Failover vs Fallback | Failover = another provider; Fallback = non-LLM plan |
