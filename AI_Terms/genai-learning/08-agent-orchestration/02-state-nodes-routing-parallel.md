# State, Nodes, Routing & Sequential vs Parallel Execution

## 1. What is it?

The four building blocks of any orchestrated flow:
- **State**: the shared data carried through the workflow (question, intermediate results, decisions, errors).
- **Nodes**: individual steps (functions) that read state, do work, and return updated state.
- **Routing**: deciding which node runs next (fixed or conditional).
- **Sequential vs parallel**: steps one-after-another vs several at once.

## 2. Why does it exist?

Steps must share data ("classification result" needed by "drafting step") and the flow must adapt ("if urgent → human approval branch"). Without explicit state and routing, steps lose each other's context.

## 3. Simple Mental Model

A factory conveyor belt: the product (state) moves along, each station (node) adds its part, and junction points (routing) send it down different lines.

## 4. How it Works

```
STATE = {
  question:      "...",
  category:      "billing",       <- set by node 1
  retrieved_docs: [...],          <- set by node 2
  draft:         "...",           <- set by node 3
  errors:        [],              <- appended by any node
  approved:      null             <- set by human node
}

NODE  = read state -> do work -> return state updates
ROUTE = if state.urgent -> approval_node else -> send_node
```

## 5. ASCII: Sequential vs Parallel

```
SEQUENTIAL (steps depend on each other):
  classify -> retrieve -> draft -> send

PARALLEL (independent steps):
              +--> summarize docs -->+
  retrieve -->+                      +--> merge --> final
              +--> check policy ----+

PARALLEL = faster (steps run together)
MERGE    = combine all branch results into one state
```

**When to parallelize:** steps that don't need each other's output (e.g., summarize 3 documents at once). **When to sequence:** each step needs the previous one's result.

## 6. Important Terms

- **State update**: a node returns only its changes; the orchestrator merges them.
- **Merge/fan-in**: joining parallel results back into one state.
- **Fan-out**: splitting into parallel branches.
- **Conditional routing**: choosing the next node at runtime based on state.
- **Deterministic routing**: your code decides (vs agent choosing its own path).

## 7. What happens internally?

The orchestrator holds the state object; each node is a function call; routing is if/else or a mapping from conditions to next nodes. Parallel steps run concurrently (async) and results merge when all complete.

## 8. How is it different?

| | State | Memory |
|---|---|---|
| Scope | Within ONE workflow run | Across steps/sessions |
| Shape | Structured object (you define) | Messages / stored facts |
| Owner | Orchestrator | App / vector store |

| Sequential | Parallel |
|---|---|
| Step N needs step N-1 | Steps independent |
| Simple, predictable | Faster, needs merging |

## 9. Common Misunderstandings

- ❌ "State and memory are the same." State = this run's shared data; memory = broader retention (state vs memory vs context confusion is a classic interview trap).
- ❌ "Parallel always better." Parallel adds merge complexity; use only for genuinely independent steps.
- ❌ "Nodes must be LLM calls." A node can be plain code, a DB query, or a human approval.

## 10. Connection to Previous

Previous: why orchestration exists. This file: its actual mechanics — state, nodes, routing, sequencing.

## 11. Connection to Next

Flows also fail — next: retries, error paths, and human-in-the-loop.

## 12. One-Minute Explanation

"An orchestrated workflow has three core concepts: state — the shared data object that flows through the system; nodes — steps that read state and return updates; and routing — the logic choosing the next node, fixed or conditional. Independent steps run in parallel (fan-out) and their results merge (fan-in); dependent steps stay sequential. State is within-run shared data, distinct from memory, which persists across runs. These concepts are framework-independent — which is exactly why LangGraph can map onto them later."

## Hinglish Explanation

State = workflow ke andar sab steps ka shared data (ek object jo aage badhta hai). Node = ek step (function) jo state padhta hai, kaam karta hai, aur update return karta hai. Routing = decide karna agla node kaunsa chale — if/else se. Jo steps independent hain unhe parallel chalao (fan-out) aur results merge karo (fan-in); dependent steps sequentially chalao. State ≠ memory: state sirf is run ka data, memory sessions ke across hoti hai.
