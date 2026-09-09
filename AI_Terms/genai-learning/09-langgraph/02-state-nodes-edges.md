# LangGraph State, Nodes & Edges

## 1. What is it?

The three primitives you define in every LangGraph app:
- **State**: a schema (usually a TypedDict-like object) describing the data that flows through the graph. Each field has a **reducer** rule for how updates merge.
- **Node**: a function `state -> partial state updates`.
- **Edge**: a declared connection from one node to another.

## 2. Why does it exist?

Nodes must exchange data safely and the graph must know the order. State+reducer makes merging explicit (critical for parallel branches); edges make the flow inspectable instead of buried in if/else.

## 3. Simple Mental Model

State = a shared whiteboard. Nodes = workers who read the whiteboard, do work, and write their results. Reducer = the rule for what happens if two workers write to the same box (overwrite? append?).

## 4. How it Works (conceptual)

```
STATE SCHEMA:
  question   (overwrite)
  documents  (overwrite)
  draft      (overwrite)
  history    (append - reducer!)

NODE:
  def retrieve(state):
      docs = vector_search(state.question)
      return {"documents": docs}       # partial update only

EDGE:
  classify -> retrieve -> draft        (fixed path)

RUN:
  invoke({question: "..."})
  -> LangGraph runs nodes in edge order,
     merging each node's updates into the state
```

## 5. ASCII: State flowing through nodes

```
        state {question}
              |
              v
        +-----------+
        | classify  |  returns {category: "billing"}
        +-----+-----+
              v   state {question, category}
        +-----------+
        | retrieve  |  returns {documents: [...]}
        +-----+-----+
              v   state {question, category, documents}
        +-----------+
        |   draft   |  returns {draft: "..."}
        +-----+-----+
              v
            END
```

## 6. Important Terms

- **Partial update**: a node returns ONLY the fields it changed — LangGraph merges the rest.
- **Reducer**: merge rule per field — overwrite vs append vs custom (e.g., dedupe).
- **START / END**: special virtual nodes marking graph entry and exit.
- **Super-step**: one round of node execution (relevant for parallelism).

## 7. What happens internally?

LangGraph keeps one state object. For the current step it runs the node function(s), applies each returned field update via that field's reducer, saves a checkpoint, then evaluates edges.

## 8. How is it different?

| | Node returns full state | Partial updates + reducers |
|---|---|---|
| Parallel safety | Branches overwrite each other | Merge rules resolve conflicts |
| Code clarity | Every node copies everything | Nodes touch only their fields |

## 9. Common Misunderstandings

- ❌ "State is a global variable." It's an explicit, versioned, checkpointed object — you can rewind it.
- ❌ "Nodes must return everything." Returning only changed fields is the design.
- ❌ "Edges are just if/else sugar." They make the whole flow visualizable and auditable as a graph.

## 10. Connection to Previous

Previous: LangGraph = orchestration concepts packaged. This file: the first three concepts (state, nodes, edges) in LangGraph terms.

## 11. Connection to Next

Fixed edges can't adapt — conditional edges add runtime routing.

## 12. One-Minute Explanation

"In LangGraph, I define a state schema describing the data that flows; nodes are functions that receive state and return partial updates; edges connect nodes in execution order. Each state field has a reducer — the merge rule — overwrite or append, which becomes crucial when parallel branches write to the same field. LangGraph runs a node, merges its updates, saves a checkpoint, then follows the outgoing edge. The graph structure makes the whole workflow inspectable rather than hidden in code."
