# Why Multiple Agents? Single vs Multi-Agent, Agent Roles

## 1. What is it?

- **Multi-agent system**: several agents, each with a focused role, cooperating on one task — coordinated by an orchestrator/supervisor or by handoffs.
- **Agent roles**: each agent has its own goal, prompt, tools, and sometimes its own LLM/model choice.

## 2. Why does it exist?

One agent doing "research + write code + review + test" has a bloated prompt, conflicting instructions, and context overflow. Splitting by **specialization** gives each agent a small, sharp job — like a company: you don't hire one person to be CEO, accountant, and lawyer.

## 3. Simple Mental Model

```
Single agent = one person doing everything (fine for small tasks)
Multi-agent  = a small team with a coordinator
```

## 4. Why one agent may be enough (be honest!)

```
USE ONE AGENT when:
- the task fits in one context window
- steps don't conflict with each other
- cost/latency matters (multi-agent = N× calls)
- debugging simplicity matters

CONSIDER MULTIPLE when:
- distinct SPECIALTIES (researcher vs coder vs reviewer)
- massive context that one window can't hold (split the work)
- you want INDEPENDENT opinions (parallel critique)
- one agent's role must be isolated/security-separated
```

## 5. ASCII: Single vs Multi

```
SINGLE:                      MULTI:
  goal                        goal
   |                           |
   v                    +------ v ------+
[one big agent]         | ORCHESTRATOR  |
   |                    +--+----+----+--+
   v                       v    v    v
 result               researcher coder reviewer
                          +----+----+
                               v
                            result
```

## 6. Important Terms

- **Specialization**: each agent tuned (prompt + tools) for one job.
- **Delegation**: giving a subtask to a specialist.
- **Coordination**: orchestrating who does what, when.
- **Role prompt**: each agent's system prompt defining its identity and limits.

## 7. What happens internally?

Same machinery as before — LLMs, tools, loops — but multiple instances with different prompts/tools, plus a coordination layer deciding which runs when and passing state between them.

## 8. How is it different?

| | Single agent | Multi-agent |
|---|---|---|
| Prompt size | Large, mixed | Small, focused |
| Cost | Lower | Higher (N agents) |
| Context | One window limit | Distributed |
| Debugging | One trajectory | Multiple + coordination |
| Failure risk | One point of confusion | Coordination bugs |

## 9. Common Misunderstandings

- ❌ "More agents = better AI." **More agents ≠ better.** Each added agent adds cost, latency, and coordination failure modes. It's a trade-off, not an upgrade.
- ❌ "Multi-agent is the advanced/proper way." Many production systems are single-agent or workflow-based for good reason.
- ❌ "Agents must be separate LLMs." An "agent" can be the same model with a different role prompt and toolset.

## 10. Connection to Previous

Previous: single agents and orchestration. This file: when to multiply agents and how to think about roles.

## 11. Connection to Next

How do agents actually coordinate? The standard patterns: orchestrator, supervisor, handoff — next file.

## 12. One-Minute Explanation

"A multi-agent system splits a task across specialized agents — researcher, coder, reviewer — each with its own role prompt, tools, and state, coordinated by an orchestrator. The honest engineering point: more agents do NOT mean better AI. Each agent adds cost, latency, and coordination failure modes. I go multi-agent only for genuine specialization, context that can't fit one window, or when I need independent parallel opinions. Otherwise a single agent or a plain workflow is the better system."

## Hinglish Explanation

Multi-agent matlab kaam ko specialists mein baantna — researcher, coder, reviewer — jaise ek company mein alag-alag departments hote hain. Lekin sach ye hai: zyada agents = behtar AI NAHI hai. Har agent extra cost, extra latency, aur coordination ki nayi galtiyan laata hai. Sirf tab multiple agents use karo jab specialization sach mein zaroori ho, ya kaam ek context window mein fit na ho.
