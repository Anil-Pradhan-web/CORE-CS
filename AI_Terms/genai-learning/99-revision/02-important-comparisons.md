# Important Comparisons (Master Table File)

All the comparison tables from the curriculum, in one place.

## 1. LLM vs Generative AI

| | Generative AI | LLM |
|---|---|---|
| Scope | Any generated content (text/image/audio) | Text only |
| Examples | DALL·E, music models | GPT, Claude, Llama |

## 2. Prompt vs System Prompt

| | Prompt | System Prompt |
|---|---|---|
| Writer | Any message, per request | Developer, app-level, fixed |
| Visibility | User sees it | Usually hidden |
| Purpose | One task | Persistent behavior/rules |

## 3. Prompt Engineering vs Fine-tuning

| | Prompt engineering | Fine-tuning |
|---|---|---|
| Changes | Input text | Model parameters |
| Cost/time | Free, instant | Data + training |
| Teaches facts | Via context (RAG) | Poorly |
| Teaches style | Good | Excellent |
| Use when | Most cases | Fixed style/domain at scale |

## 4. Embedding vs Token

| | Token | Embedding |
|---|---|---|
| What | Chunk of text | Vector of numbers |
| Role | LLM input/output unit | Meaning representation |
| Used for | Generation | Search/similarity |

## 5. Embedding vs Vector Database

| | Embedding | Vector DB |
|---|---|---|
| What | A technique/model output | A storage system |
| Does | Text → vector | Stores vectors, ANN search, filtering |

## 6. Semantic vs Keyword Search

| | Keyword | Semantic |
|---|---|---|
| Matches | Exact words | Meaning |
| Misses | Synonyms | Rarely |
| Engine | SQL/Elasticsearch | FAISS/vector DB |

## 7. RAG vs Fine-tuning

| | RAG | Fine-tuning |
|---|---|---|
| Changes | Prompt context | Parameters |
| Knowledge updates | Instant (re-index) | Retraining |
| Citations | Yes | No |
| Best for | Facts, private/recent data | Style, domain behavior |

## 8. RAG vs Long Context

| | RAG | Long context |
|---|---|---|
| Cost | Low (few chunks) | High (all tokens each call) |
| Quality | Focused | "Lost in the middle" |
| Scale | Millions of docs | Window-limited |

## 9. Tool Calling vs Agent

| | Tool calling | Agent |
|---|---|---|
| Scope | One round-trip | Self-directed loop |
| Who stops | Your code | Model + stop conditions |

## 10. Agent vs Workflow

| | Workflow | Agent |
|---|---|---|
| Steps | You predefine | Model chooses |
| Predictability | High | Low |
| Cost | Lower | Higher |
| Use when | Steps known | Steps unknowable |

## 11. Workflow vs Orchestration

| | Workflow | Orchestration |
|---|---|---|
| What | A process with predefined steps | The layer managing flows (state, routing, errors, parallel, HITL) |
| Relationship | What runs | Who coordinates what runs |

## 12. Single vs Multi-Agent

| | Single | Multi |
|---|---|---|
| Cost/debug | Lower | Higher |
| Context | One window | Distributed |
| Use when | Fits one window/role | Real specialization, isolation, parallel opinions |

## 13. Sequential vs Parallel Agents

| | Sequential | Parallel |
|---|---|---|
| Dependency | Each needs previous | Independent |
| Speed | Slower | Faster + merge logic |

## 14. LangGraph vs LangChain

| | LangChain | LangGraph |
|---|---|---|
| Focus | Components/chains | Stateful graphs |
| Flow | Mostly linear | Branches, loops, parallel, resumable |
| Best for | Simple integrations | Agents, HITL, production flows |

## 15. State vs Memory vs Context

| | State | Memory | Context |
|---|---|---|---|
| Scope | One workflow run | Across steps/sessions | One request |
| Lives in | Orchestrator/graph | DB / vector store | Token window |
| Managed by | Your orchestration | Your app + stores | Auto-limited by model |

## 16. Retry vs Failover

| | Retry | Failover |
|---|---|---|
| Action | Same provider again | Different provider |
| When | Transient errors | Persistent failure |
| Risk | Amplifies load if abused | Needs abstraction layer |

## 17. Retry vs Circuit Breaker

| | Retry | Circuit breaker |
|---|---|---|
| Scope | One call | Whole provider |
| Prevents | One missed request | Stampeding a dead service |
| Recovery | Immediate next attempt | Cool-down + half-open probe |

## 18. Failover vs Fallback

| | Failover | Fallback |
|---|---|---|
| What | Another LLM provider | Non-LLM degraded plan |
| Example | Groq → Gemini | Cached/template answer |

## 19. Model Error vs System Error

| | Model error | System error |
|---|---|---|
| What | Wrong content | Call fails |
| Layer | Semantic | Infrastructure |
| Fix | Prompts, RAG, validation | Retry, backoff, breaker, failover |

## 20. LLM vs Agent (repeated because it matters most)

| | LLM | Agent |
|---|---|---|
| One response vs | Multi-step to a goal |
| Code chooses steps | Model chooses steps |
| No tools needed | Tools are the core |
