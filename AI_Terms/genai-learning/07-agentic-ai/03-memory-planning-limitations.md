# Memory, Planning, Limitations & the Agentic Mental Model

## 1. What is it?

- **Memory**: what the agent retains across steps (short-term = the current task's state; long-term = stored across sessions, often in a vector store).
- **Planning**: the agent decomposing a goal into steps — either up-front (plan first, then execute) or step-by-step inside the loop.
- **Agent limitations**: the honest failure list — and the folder's mental model.

## 2. Why does it exist?

Without memory the agent repeats work and forgets findings. Without planning it wanders. And without understanding limitations, teams ship unreliable agents to production.

## 3. Simple Mental Model

```
Short-term memory = notepad on the desk (current task)
Long-term memory  = filing cabinet (past sessions, searchable)
Planning          = making a to-do list (and updating it)
```

## 4. How Memory Works

```
SHORT-TERM (within one task):
messages list = [goal, step1 result, step2 result, ...]
grows every loop -> eventually hits context limit
fix: summarize old steps, keep recent ones

LONG-TERM (across sessions):
facts/preferences stored as embeddings in a vector store
retrieved into context when relevant
(exactly the semantic-search machinery from earlier!)
```

### Memory vs Context (classic confusion!)

| | Memory | Context |
|---|---|---|
| Scope | Across steps AND sessions | One request |
| Managed by | Your app (DB, vector store) | The API call |
| Size | Unlimited (stored) | Hard token limit |

## 5. Planning Styles

| Style | How | Risk |
|---|---|---|
| Plan-then-execute | Make full plan first, then run steps | Plan breaks when reality differs |
| Reactive (ReAct) | Plan one step at a time in the loop | Can wander without a goal-check |
| Hybrid | Rough plan + re-plan as needed | More complex |

## 6. ASCII: Agent with memory + plan

```
              +-------------------+
   Goal ----->|      AGENT        |
              |  plan (to-do list)|
              |  short-term memory|----> grows each step
              |  LLM + tools      |
              +--------+----------+
                       |
                 long-term memory
              (vector store, sessions)
```

## 7. Agent Limitations (memorize this list)

| Limitation | Meaning | Mitigation |
|---|---|---|
| Compounding errors | One wrong step derails the rest | Validation, checkpoints |
| Loops / wandering | Repeats or drifts off-goal | Max steps, goal checks |
| Cost & latency | Many LLM calls per task | Small models per step, caching |
| Unreliable tool use | Wrong tool / bad args | Schemas + validation |
| Over-autonomy | Does things user didn't want | Human-in-the-loop for risky actions |
| Debugging difficulty | Non-deterministic paths | Logging every step (observability) |

## 8. How is it different?

| | Chatbot | Workflow | Agent |
|---|---|---|---|
| Steps | Single turn | Fixed, you coded them | Dynamic, model decides |
| Predictability | High | High | Lower |
| Good for | Q&A | Known pipelines | Open-ended tasks |

## 9. Common Misunderstandings

- ❌ "Agents will figure it out." Only within their tools, memory, and limits.
- ❌ "Long-term memory is automatic." You build and manage the store.
- ❌ "One agent for everything." Complex jobs may need orchestration — next folder.

## 10. Connection to Previous

Previous: the loop. This file: what feeds the loop (memory), how it stays on track (planning), and why production agents are hard.

## 11. Connection to Next

A single agent works for contained tasks. When logic gets complex — fixed steps, approvals, parallel branches — you need **orchestration** around the agent.

## 12. One-Minute Explanation

"Agent memory is short-term — the task's message history, summarized when it grows too long — and long-term — facts stored in a vector store across sessions, retrieved semantically when needed. Planning can be plan-first or step-by-step (ReAct); hybrids re-plan as reality diverges. The key limitations: compounding errors, wandering loops, multiplying cost, unreliable tool use, and hard debugging — so real agents need max-step limits, validation, human-in-the-loop for risky actions, and step-level logging."

## Hinglish Explanation

Memory do type ki hoti hai: short-term (current task ka notepad — messages list) aur long-term (vector store mein saved purani baatein, jo semantic search se wapas milti hain). Planning matlab agent apna to-do list banata hai — pehle poora plan ya ek-ek step karke (ReAct). Limitations yaad rakho: galtiyan jama hoti hain, loops mein phas sakta hai, cost multiply hoti hai, aur risky actions ke liye human approval chahiye.
