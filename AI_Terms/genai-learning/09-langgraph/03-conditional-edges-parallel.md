# Conditional Edges, State Transitions & Parallel Branches

## 1. What is it?

- **Conditional edge**: an edge whose destination is decided AT RUNTIME by a function that inspects the state — LangGraph's routing.
- **State transition**: the move from one state snapshot to the next as nodes run.
- **Parallel branches (fan-out/fan-in)**: one node's output sent to several nodes at once; their results merged back.

## 2. Why does it exist?

Real workflows branch: urgent tickets go to human approval, simple ones auto-send; independent analyses run simultaneously. Fixed edges can't express this — conditional edges and parallel branches can, while keeping the flow explicit and visualizable.

## 3. Simple Mental Model

Conditional edge = a railway junction with a signalman who reads the cargo (state) and switches the track. Parallel branches = the cargo splits into three trains running together, merging back at the next junction.

## 4. How it Works

```
CONDITIONAL EDGE:
  def route(state):
      if state["severity"] == "urgent":
          return "human_approval"
      return "send_reply"
  # attached to node "classify": its next node
  # is whatever this function returns

PARALLEL:
  retrieve --fan-out--> [summarize_docs, check_policy]
                both run in the same super-step
           --fan-in--> merge results -> draft
```

## 5. ASCII: Branching graph

```
                 +----------+
                 | classify |
                 +----+-----+
            conditional edge (reads severity)
           +---------+---------+
           |                   |
           v                   v
   +---------------+     +-----------+
   | human approval|     | send auto |
   +-------+-------+     +-----+-----+
           |                   |
           +----+--------+-----+
                v
               END
```

Parallel version:

```
                +----------+
                | retrieve |
                +----+-----+
          +----------+----------+
          v          v          v
     summarize   translate   check_policy
          |          |          |
          +----+-----+-----+---+
               v   (merge: reducers combine)
            +---------+
            |  draft  |
            +---------+
```

## 6. Important Terms

- **Router function**: the function attached to a conditional edge; returns the name of the next node.
- **Fan-out / fan-in**: split into branches / join branches.
- **Super-step**: one round in which all parallel nodes execute together.
- **Merge conflict resolution**: reducers decide how parallel updates combine (append = all kept).

## 7. What happens internally?

After each step, LangGraph evaluates outgoing conditional edges against the latest state. For fan-out, all target nodes execute in the same super-step (concurrently); updates merge via reducers before the next step begins.

## 8. How is it different?

| | Fixed edge | Conditional edge |
|---|---|---|
| Next node | Always the same | Chosen from state at runtime |
| Use for | Linear flows | Branching, escalation, retry paths |

| Agent chooses steps | Conditional edges |
|---|---|
| LLM decides dynamically | YOU define the conditions |
| Flexible, unpredictable | Flexible but auditable |

## 9. Common Misunderstandings

- ❌ "Conditional edges are an LLM decision." They're YOUR deterministic functions reading state — that's the point: controlled flexibility.
- ❌ "Parallel nodes share live state." They see the same snapshot; merges happen at fan-in.
- ❌ "Reducers are optional." Without append-type reducers, parallel branches overwrite each other's data.

## 10. Connection to Previous

Previous: state, nodes, fixed edges. This file: making the graph adaptive (conditional) and fast (parallel).

## 11. Connection to Next

Adaptive graphs need memory across time — checkpoints and interrupts.

## 12. One-Minute Explanation

"Conditional edges are router functions attached to a node: they read the current state and return which node runs next — my deterministic logic, not the LLM's whim. Parallel branches fan out one state snapshot to several nodes that execute in the same super-step, and fan in by merging updates through field reducers — which is why append-reducers matter. Fixed edges handle linear flow, conditional edges handle branching, parallel handles independent work — together they express any orchestration pattern."

## Hinglish Explanation

Conditional edge matlab "signalman": state dekh kar decide karta hai agla node kaunsa chalega — ye aapka code hai, LLM ka nahi. Parallel branches matlab ek kaam ko 3 nodes pe ek saath bhejna (fan-out) aur unke results ko reducers se merge karna (fan-in). Fixed edge = seedha rasta, conditional edge = mod pe decision, parallel = speed.
