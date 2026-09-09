# Tool Calling — Mental Model (Folder Summary)

## 1. What is it?

One page compressing tool calling into a single picture.

## 2. Why this summary exists

Three files → one recallable diagram.

## 3. The Mental Model

```
LLM = the DECIDER (which tool? which arguments?)
YOUR CODE = the EXECUTOR (validate, run, return result)
```

The LLM never runs anything. It negotiates with your code in a structured way.

## 4. The Complete Round-Trip

```
User question
     |
     v
+------ schemas + messages ------+
|              LLM               |
+------+-------------------------+
       | tool call {name, args}
       v
  [validate] --invalid--> error message back to LLM
       | valid
       v
  [execute] real function (DB / API / action)
       |
       v
  result message
       |
       v
  LLM final answer (or another tool call -> repeat)
```

## 5. Key Design Rules

| Rule | Why |
|---|---|
| Write rich descriptions | Only thing the model knows |
| Validate every argument | Model-generated = untrusted |
| Return errors as messages | LLM can recover gracefully |
| Auth in code, never in LLM | LLM must not grant permissions |
| Confirm dangerous actions | Irreversible actions need a human |

## 6. Important Terms recap

Tool, schema, description, arguments, tool call, tool result, round-trip, validation, authorization, finish_reason="tool_calls".

## 7. Internals recap

Schemas travel in the request; the fine-tuned model emits structured tool calls; the provider parses them; your loop completes the cycle.

## 8. Comparisons recap

| RAG | Tools |
|---|---|
| Gives knowledge | Gives action/live data |

| Tool calling | Agent |
|---|---|
| One round-trip | Self-directed loop of round-trips |

| LLM | Your code |
|---|---|
| Decides | Executes + validates + authorizes |

## 9. Common Misunderstandings recap

The LLM never executes; descriptions matter enormously; security is never the LLM's job.

## 10. Connection to Previous

RAG completed "LLM + knowledge". Tools complete "LLM + actions". The LLM can now know and do.

## 11. Connection to Next

Give the model a goal, tools, and a loop where it observes results and decides the next step — that loop is an **agent**.

## 12. One-Minute Explanation

"Tool calling splits responsibility: the LLM decides which function to call and with what arguments; my code validates, authorizes, executes, and returns results as messages. One round-trip is tool calling; a self-directed loop of think-act-observe is an agent. Combined with RAG — knowledge — the LLM can now both know and do, which is exactly the substrate agents are built on."
