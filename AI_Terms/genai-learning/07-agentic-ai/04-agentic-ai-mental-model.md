# Agentic AI — Mental Model (Folder Summary)

## 1. What is it?

One page compressing agentic AI into a single picture.

## 2. Why this summary exists

Three files → one diagram + one paragraph you can say in an interview.

## 3. The Full Agent Picture

```
                 GOAL
                  |
                  v
        +------------------+
        |      AGENT       |
        |                  |
        |   +----------+   |
        |   |   LLM    |   |     MEMORY
        |   | (decide) |   |   - short: state/messages
        |   +----+-----+   |   - long: vector store
        |        |         |
        |   PLAN (to-do / ReAct thoughts)
        |        |         |
        |   ACT (tool)     |
        |        |         |
        |   OBSERVE (result appended to state)
        |                  |
        +--------+---------+
                 |
      goal met? max steps? need human?
                 |
                 v
            RESULT
```

## 4. The One-Formula

```
Agent = LLM + Goal + Tools + Memory + Loop + Stop condition
```

Remove any piece: no goal = wandering; no tools = just a chatbot; no loop = one-shot call; no stop = infinite loop; no memory = repeats work.

## 5. Key Numbers & Rules

- Each loop step = 1 LLM call (cost × steps).
- Always set max steps (e.g., 10–20).
- Risky actions → human approval.
- Log every step — non-deterministic paths need traces.

## 6. Important Terms recap

Autonomy, trajectory, ReAct, state, observation, termination, short/long-term memory, plan-then-execute, human-in-the-loop.

## 7. Comparisons recap

| Plain LLM | Agent |
|---|---|
| One response | Multi-step to goal |
| Code controls flow | Model controls flow |

| Workflow | Agent |
|---|---|
| Fixed coded steps | Dynamic chosen steps |
| Predictable | Flexible but riskier |

## 8. Common Misunderstandings recap

Agents aren't smarter LLMs, they're LLMs plus loop/tools/memory/autonomy; they're not always better; they don't remember automatically; they need stop conditions.

## 9. When to Use an Agent (honest answer)

```
Use an agent when:      the steps CANNOT be known in advance
Use a workflow when:    the steps ARE known (cheaper, predictable)
Use a plain prompt when: one call suffices (cheapest, most reliable)
```

## 10. Connection to Previous

Tool calling (hands) + RAG (knowledge) + loop (autonomy) = agent. The natural next layer of the same story.

## 11. Connection to Next

One agent in a while-loop is fine for small tasks. But real systems need fixed sequences, parallel branches, approvals, retries — structure around the intelligence. That structure is **orchestration**.

## 12. One-Minute Explanation

"An agent is an LLM plus a goal, tools, memory, and a think-act-observe loop with a stop condition. It differs from a plain LLM call because the model, not my code, chooses the steps. Short-term memory is the task's message history; long-term memory lives in a vector store. Agents trade predictability for flexibility — they cost more per task and can wander — so I use them only when steps can't be known in advance, always with max-step limits and logging. When structure is needed around them, that's orchestration."
