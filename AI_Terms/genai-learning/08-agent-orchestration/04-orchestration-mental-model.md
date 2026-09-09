# Orchestration — Mental Model (Folder Summary)

## 1. What is it?

One page compressing orchestration into a single picture — the concepts LangGraph will later implement.

## 2. Why this summary exists

This folder is the conceptual foundation for the LangGraph folder. If this page is clear, LangGraph is trivial.

## 3. The Complete Orchestration Picture

```
                      +-----------------+
  Input ------------->|  ORCHESTRATOR   |
                      | - owns state    |
                      | - owns routing  |
                      | - owns errors   |
                      +--------+--------+
                               |
        +----------------------+----------------------+
        v                      v                      v
   [node: classify]      [node: RAG]         [node: agent]
        |                      |                      |
        +----------+-----------+----------+-----------+
                   v
             [route: urgent?]
              |            |
             yes          no
              |            |
   [HITL approval]      [send]
              |            |
              +-----+------+
                    v
              final result
   (any node fails -> retry / error path / abort)
```

## 4. The Checklist (what every orchestrated system decides)

| Question | Concept |
|---|---|
| What data moves between steps? | State |
| What are the steps? | Nodes |
| Which step runs next? | Routing (fixed/conditional) |
| Which steps run together? | Parallel + merge |
| What if a step fails? | Retry / error path |
| Where must a human decide? | HITL |
| When is the whole thing done? | Termination |

## 5. Key Decisions recap

| Workflow | Agent |
|---|---|
| Steps predefined | Steps self-chosen |
| Predictable, cheap | Flexible, costly |
| Default choice | Use only when steps unknowable |

| Sequential | Parallel |
|---|---|
| Dependent steps | Independent steps |
| Simple | Faster + merge logic |

## 6. Important Terms recap

Orchestrator, state, node, routing, conditional edge (preview), fan-out/fan-in, merge, retry, error path, checkpoint, interrupt/resume, human-in-the-loop, termination.

## 7. Internals recap

The orchestrator is structured code: hold state → call node → merge update → route → handle errors → repeat. Parallel = concurrent node calls. HITL = persist state + stop + resume.

## 8. Comparisons recap

State (this run's data) vs Memory (across runs) vs Context (one request). Workflow vs Agent. Retry vs Fallback vs HITL.

## 9. Common Misunderstandings recap

Orchestration contains agents rather than replacing them; predictability beats autonomy when possible; error paths are designed, not improvised.

## 10. Connection to Previous

Agentic AI gave you the worker; orchestration gives the factory around the worker.

## 11. Connection to Next

LangGraph = these exact concepts (state, nodes, edges, conditional routing, checkpoints, interrupts) packaged as a graph framework. You already know the ideas — next folder just names them.

## 12. One-Minute Explanation

"Orchestration is the control layer of an AI system: it owns state — the shared data between steps; nodes — the steps themselves; routing — which node runs next, fixed or conditional; parallel branches with fan-out and fan-in; error paths with retries; and human-in-the-loop pauses that persist state and resume after approval. Workflows predefine steps for predictability; agents choose steps for flexibility; production systems usually mix both — workflows containing agents. Every concept here maps directly onto LangGraph's terminology."
