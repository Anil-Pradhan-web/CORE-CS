# Shared State, Communication, Failure & Recovery, When to Use Multi-Agent

## 1. What is it?

- **Shared state & communication**: how agents exchange information — a common state object, messages between agents, or handoff payloads.
- **Agent failure & recovery**: what happens when one agent fails or produces garbage — and how the system recovers.
- **When to use multi-agent**: the final honest decision guide.

## 2. Why does it exist?

Agents don't share a brain. Without explicit communication design, agents contradict each other, repeat work, or build on hallucinations. And a team is only as reliable as its weakest member's failure handling.

## 3. Simple Mental Model

Shared state = the team's whiteboard everyone reads/writes. Communication = meetings (direct messages between agents). Failure handling = the backup plan when a teammate calls in sick.

## 4. How Communication Works

```
Option 1 - SHARED STATE (most common):
  all agents read/write one state object
  {question, research, draft, review, errors}

Option 2 - DIRECT MESSAGE:
  agent A's output = agent B's input (message passing)

Option 3 - BLACKBOARD (lightweight shared memory):
  agents post findings; others read when relevant
```

```
Risks: conflicting writes (solved by reducers),
       context bloat (agent reads everything),
       and building on another agent's hallucination.
```

## 5. ASCII: Failure in a team

```
   supervisor -> researcher -> coder --> FAILS (bad tool / bad output)
                                   |
                    +--------------+---------------+
                    v              v               v
              retry coder    different agent   fallback path
              (same task)    (more capable)    (partial result
                                               + human review)
   supervisor should ALWAYS have a plan for each specialist failing
```

## 6. Failure Handling Patterns

| Failure | Response |
|---|---|
| Transient (timeout, rate limit) | Retry with backoff |
| Bad output | Validation + one retry with feedback ("your JSON was invalid") |
| Agent stuck/loops | Supervisor overrides, re-routes, or skips |
| Agent unavailable | Failover to another agent/model |
| All failed | Degrade: partial answer + human alert |

## 7. The Honest Decision Guide

```
Use multi-agent ONLY when at least one is true:
  [ ] genuine distinct specializations needed
  [ ] task too big for one context window
  [ ] independent parallel opinions add real value
  [ ] isolation needed (security/separation of concerns)

Otherwise prefer (in order):
  single prompt -> workflow -> single agent -> multi-agent
  (each step right = cheaper, faster, easier to debug)
```

## 8. How is it different?

| Shared state | Message passing |
|---|---|
| One source of truth | Explicit point-to-point |
| Risk: bloat/conflicts | Risk: lost context |
| Natural in LangGraph | Natural in handoff patterns |

## 9. Common Misunderstandings

- ❌ "Agents should talk freely like humans." Unstructured chatter multiplies errors — design the interfaces.
- ❌ "A failing agent will fix itself." Recovery must be DESIGNED (retry/re-route/fallback).
- ❌ "Multi-agent is more reliable (redundancy!)." More components = more failure points, unless recovery is designed in.

## 10. Connection to Previous

Previous: coordination patterns. This file: the data flow and failure reality that make those patterns production-safe.

## 11. Connection to Next

Everything you've built — LLM, RAG, tools, agents, graphs — must survive the real world: timeouts, rate limits, provider outages. Final folder: **resilience**.

## 12. One-Minute Explanation

"Multi-agent communication is designed, not improvised: usually a shared state object with reducers resolving write conflicts, sometimes direct message passing or handoff payloads. Failures need designed recovery: retries for transient errors, validation-with-feedback for bad outputs, supervisor override for stuck agents, failover to another agent, and human alerts as the last resort. And the honest rule: choose multi-agent only for real specialization, context limits, or valuable parallel opinions — otherwise prefer a single prompt, workflow, or single agent."

## Hinglish Explanation

Agents ke beech communication design karke hoti hai — sabse common hai ek shared state (team ka whiteboard) jo sab padhte/likhte hain, conflicts reducers se solve hote hain. Koi agent fail ho to recovery plan pehle se hona chahiye: retry, doosre agent pe failover, ya human ko alert. Aur yaad rakho — multi-agent team redundancy NAHI laata; bina recovery design ke components badhne se failure points badhte hain.
