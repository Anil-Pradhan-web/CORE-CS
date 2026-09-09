# Goals, State, Actions, Observations & the Agent Loop (ReAct)

## 1. What is it?

The engine of every agent:
- **Goal**: what success looks like.
- **State**: everything known so far (messages, results, facts).
- **Action**: one step the agent takes (call a tool).
- **Observation**: the result of that action, fed back into state.
- **Agent loop**: think → act → observe → repeat.
- **ReAct**: the classic pattern — **Rea**son + **Act**, interleaved.

## 2. Why does it exist?

One-shot LLM calls can't handle tasks whose next step depends on the previous step's outcome. The loop makes the agent *react to reality*: it plans a little, acts, sees what happened, and re-plans.

## 3. Simple Mental Model

Cooking without tasting: you follow the recipe blind (plain LLM call). Cooking with tasting: you taste (observe), then adjust salt (next action). The loop is taste-and-adjust, repeated.

## 4. How the Loop Works

```
LOOP:
  1. THINK    (reason about goal + state -> pick next action)
  2. ACT      (execute that action via a tool)
  3. OBSERVE  (append the result to state)
  4. repeat until: goal met | max steps | needs human
```

**ReAct trace example** (goal: "cheapest flight Delhi→Goa on Friday"):

```
Thought: I need flight options.           <- Reason
Action:  search_flights(Delhi, Goa, Fri)  <- Act
Observation: 5 flights found, min ₹6,200  <- Observe
Thought: Under ₹8k. Check baggage rules.
Action:  get_flight_details(flight_3)
Observation: includes 15kg cabin bag
Thought: Fits the budget and constraints. Task done.
Answer: Book flight_3 at ₹6,200 ...
```

## 5. ASCII: The loop diagram

```
        +---------------------+
        |        Goal         |
        +----------+----------+
                   v
        +---------------------+
   +--->|  THINK (reason)     |
   |    +----------+----------+
   |               v
   |    +---------------------+
   |    |  ACT (use tool)     |
   |    +----------+----------+
   |               v
   |    +---------------------+
   +----+  OBSERVE (result)    |
        +----------+----------+
                   |
        goal met? / max steps?
                   |
                   v
               Final answer
```

## 6. Important Terms

- **State**: the growing record — messages + tool results. In practice: the messages list.
- **Observation**: tool result text appended as a message.
- **ReAct**: a prompting/loop pattern from research, now standard in agent frameworks.
- **Max steps**: hard safety limit (e.g., 15) so agents can't loop forever.
- **Termination condition**: how the agent signals "done".

## 7. What happens internally?

Every loop iteration = one LLM API call where messages contain everything so far. Tools execute between calls. Memory = that messages list (state).

## 8. How is it different?

| | Chain (prompt chaining) | Agent loop |
|---|---|---|
| Steps | Fixed in advance | Decided at runtime |
| Branching | You coded it | Model chooses |
| Adapts to results | No | Yes |

## 9. Common Misunderstandings

- ❌ "ReAct is a product/library." It's a pattern: reason-act-observe, repeatable with plain API calls.
- ❌ "The agent remembers everything forever." Only what's in state/context; old steps may need summarizing.
- ❌ "Loops are free." Each iteration is a billed LLM call — cost multiplies.

## 10. Connection to Previous

Previous: what an agent IS (components). This file: how it RUNS — the loop that gives it autonomy.

## 11. Connection to Next

The loop is powered by state — but what is memory exactly, and how does the agent plan? Next file.

## 12. One-Minute Explanation

"An agent runs a loop: reason about the goal and current state, take one action via a tool, observe the result, and repeat until done or a max-steps limit is hit. ReAct is that pattern formalized — reasoning and acting interleaved, like a trace of thoughts, tool calls, and observations. Each iteration is a real LLM call with the full history in context, so cost and latency grow with steps, which is why max-steps limits and termination conditions are mandatory."

## Hinglish Explanation

Agent loop = socho → karo → dekho → repeat. ReAct isi ka formal naam hai: Reason (sochna) + Act (karna) + Observation (result dekhna). Har step ke baad agent result dekh kar agla step khud decide karta hai — jaise khana banate waqt taste karke namak adjust karna. Yaad rakho: har loop iteration ek paid LLM call hai, isliye max-steps limit lagana zaroori hai.
