# Multi-Agent Patterns: Orchestrator, Supervisor, Handoff, Sequential & Parallel

## 1. What is it?

The standard coordination patterns:
- **Orchestrator pattern**: a coordinator (code or LLM) plans, delegates, and collects results.
- **Supervisor pattern**: an LLM-driven boss that READS the state and ROUTES to specialist agents in a loop.
- **Handoff pattern**: an agent that finishes its part PASSES CONTROL directly to the next agent (peer-to-peer).
- **Sequential / parallel multi-agent**: specialists in a chain vs working simultaneously.

## 2. Why does it exist?

Coordination is the hard part of multi-agent systems. These patterns are the reusable answers to: who decides which agent runs next, and how do results move between them?

## 3. Simple Mental Model

- Orchestrator = project manager with a plan.
- Supervisor = receptionist routing customers: "billing? go to agent B."
- Handoff = colleagues passing a file directly to the right teammate.

## 4. How Each Works

**Supervisor (LLM decides routing):**
```
loop: supervisor reads state -> picks next agent
      agent works -> result into state -> supervisor again
      until supervisor says FINISH
```

**Handoff (agent decides):**
```
Agent A: "this is a billing issue" -> HANDOFF to Agent B
Agent B: works, answers, maybe hands off again
```

**Sequential:** A -> B -> C (each refines previous output)
**Parallel:** A, B, C simultaneously -> merge (faster, independent views)

## 5. ASCII Diagrams

```
SUPERVISOR:                 HANDOFF:
      +------------+              +-----+  handoff  +-----+
      | SUPERVISOR |<----loop     |  A  |---------> |  B  |
      +-----+------+              +-----+           +--+--+
     routing |                        <--------------+
   +----+----+----+              (agents pass control
   v    v         v               directly to each other)
  A    B         C

SEQUENTIAL:  A -> B -> C -> result
PARALLEL:    A -+
             B  +--> merge -> result
             C -+
```

## 6. Important Terms

- **Router/supervisor prompt**: the supervisor's instructions describing each specialist and when to use them.
- **Handoff message**: control + context passed from one agent to the next.
- **Shared state**: the common data object all agents read/write (LangGraph state).
- **FINISH node**: the supervisor's explicit "task complete" signal.

## 7. What happens internally?

Supervisor patterns map naturally onto LangGraph: the supervisor is a node with conditional edges to each specialist; specialists write to shared state; a FINISH conditional edge ends the loop. Handoffs are conditional edges chosen by the specialist itself.

## 8. How is it different?

| | Supervisor | Handoff | Orchestrator (code) |
|---|---|---|---|
| Who routes | LLM in a loop | Current agent | Your code |
| Predictability | Medium | Low-medium | High |
| Flexibility | High | High | Fixed plan |
| Debugging | Trace the loop | Trace the chain | Read the code |

| Sequential | Parallel |
|---|---|
| Each builds on previous | Independent work |
| Slower, ordered | Faster, needs merge |

## 9. Common Misunderstandings

- ❌ "Supervisor and handoff are totally different systems." Both are routing — supervisor routes from the center, handoff routes peer-to-peer.
- ❌ "Parallel agents share live edits." They work from a snapshot and merge at fan-in (same as LangGraph branches).
- ❌ "The supervisor is smarter." It's a focused LLM call doing one job: choosing the next agent.

## 10. Connection to Previous

Previous: why multiple agents and roles. This file: the concrete coordination patterns that make a team work.

## 11. Connection to Next

Coordination has a dark side: communication failures and agent failures — next file.

## 12. One-Minute Explanation

"The three coordination patterns: an orchestrator — usually my code — plans and delegates; a supervisor — an LLM node in a loop — reads shared state and routes to specialists until it emits FINISH; and handoff — where an agent passes control directly to a peer, like transferring a call. Sequential chains refine one another's output; parallel agents work independently and merge at fan-in. All of these map directly to LangGraph nodes and conditional edges: the supervisor is just a node with conditional edges to every specialist."
