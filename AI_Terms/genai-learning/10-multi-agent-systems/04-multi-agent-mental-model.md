# Multi-Agent Systems — Mental Model (Folder Summary)

## 1. What is it?

One page compressing multi-agent systems into a single picture.

## 2. Why this summary exists

Three files → one diagram + decision rules you can quote.

## 3. The Complete Multi-Agent Picture

```
                     +------------------+
   Task ------------>|  ORCHESTRATOR /  |
                     |   SUPERVISOR     |
                     +--------+---------+
              (routes based on state)
        +------------+------------+------------+
        v            v            v            v
   Agent A       Agent B       Agent C     (parallel or
   research      code          review       sequential)
        |            |            |
        +------------+-----+------+
                           v
                  SHARED STATE (reducers merge)
                           |
                     [FINISH / HITL / failover]
                           v
                      Final result

   Failure path: any agent fails -> retry / re-route /
                 failover / fallback / human alert
```

## 4. The Pattern Cheat Sheet

| Pattern | Who routes | Best for |
|---|---|---|
| Orchestrator | Your code | Known plans |
| Supervisor | LLM in a loop | Dynamic specialist selection |
| Handoff | Current agent | Peer expertise transfer |
| Sequential | Fixed chain | Refinement pipelines |
| Parallel | Fan-out/fan-in | Independent opinions |

## 5. The Golden Rules

```
1. More agents ≠ better AI (cost + coordination failures grow)
2. Prefer: prompt -> workflow -> single agent -> multi-agent
3. Design communication (shared state + reducers), don't let agents "chat freely"
4. Design recovery for every specialist
5. Every agent = focused role prompt + minimal tools
```

## 6. Important Terms recap

Specialization, delegation, coordination, role prompt, orchestrator, supervisor, handoff, shared state, reducer, fan-out/fan-in, failover, FINISH node.

## 7. What happens internally recap

It's all the same machinery: nodes (agents) + state + conditional edges. A supervisor is a node routing to specialists; handoffs are conditional edges chosen by agents.

## 8. Comparisons recap

Single vs Multi: cost/debugging vs specialization/context. Supervisor vs Handoff: central routing vs peer routing. Shared state vs message passing.

## 9. Common Misunderstandings recap

Multi-agent isn't an upgrade by default; agents don't fix each other without designed recovery; free-form agent chatter multiplies errors.

## 10. Connection to Previous

Agents + orchestration = the pieces; this folder = assembling a team.

## 11. Connection to Next

A team of agents makes MANY LLM calls — more calls = more failures. The final engineering layer: keeping it all alive — **resilience**.

## 12. One-Minute Explanation

"A multi-agent system is specialized agents coordinated through a pattern: orchestrator (code plans), supervisor (LLM loops and routes), or handoff (agents pass control peer-to-peer), running sequentially or in parallel over a shared state with reducer-based merging. The core engineering truths: more agents don't mean better AI — they mean more cost and failure modes — communication must be designed, every specialist needs a recovery plan, and I only go multi-agent for real specialization, context limits, or valuable parallel opinions."

## Hinglish Explanation

Multi-agent system = specialized agents ka team. Coordination ke 3 tarike: orchestrator (aapka code plan banata hai), supervisor (ek LLM loop mein decide karta hai kaunsa agent chalega), handoff (agent direct doosre agent ko kaam saup deta hai). Golden rule: zyada agents = behtar AI nahi. Communication design karo (shared state + reducers), har agent ka recovery plan rakho, aur multi-agent sirf tab jab sach mein zaroorat ho.
