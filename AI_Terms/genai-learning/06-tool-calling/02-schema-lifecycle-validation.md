# Tool Schemas, Execution Lifecycle, Results & Validation

## 1. What is it?

- **Tool schema**: the contract you give the LLM — name, description, parameters (names, types, which are required).
- **Execution lifecycle**: the full journey register → decide → execute → return → respond.
- **Tool results & validation**: feeding results back safely and handling errors/invalid arguments.

## 2. Why does it exist?

The LLM can't read your code — it only knows what the schema's description says. A vague description = wrong tool chosen or wrong arguments. And since the LLM *generates* arguments, they can be wrong, incomplete, or even dangerous — so your code must validate before executing.

## 3. Simple Mental Model

A schema is a job card for an intern: tool name, what it does, what inputs it needs, examples. Garbage job card → garbage work. Validation is you checking the intern's filled form before stamping it.

## 4. How a Tool Schema Looks (conceptually)

```
name:        "get_order_status"
description: "Returns shipping status for an order ID.
              Use when the user asks where their order is."
parameters:
  order_id  (string, required)  - e.g. "A-1023"
```

The **description is the steering wheel** — it's how the model decides when to use this tool and which other tool NOT to use.

## 5. ASCII: Full lifecycle

```
1. REGISTER     app sends tool schemas with the request
       |
       v
2. DECIDE       LLM picks a tool + generates arguments
       |          (finish_reason = "tool_calls")
       v
3. VALIDATE     your code checks: right tool? args valid?
       |            invalid -> send error back to LLM
       v
4. EXECUTE      run the real function (DB/API/whatever)
       |
       v
5. RETURN       result (or error) as a message
       |
       v
6. RESPOND      LLM reads result -> writes final answer
                  (or requests another tool -> loop)
```

## 6. Important Terms

- **Argument validation**: checking types, required fields, allowed values — never trust generated args.
- **Error-as-message pattern**: instead of crashing, send `{"error": "order not found"}` back — the LLM can explain gracefully to the user.
- **Authorization**: permission checks happen in YOUR code, not the LLM ("the LLM must never decide what the user is allowed to do").
- **Idempotency & confirmation**: for dangerous actions (send money, delete), require human confirmation.

## 7. What happens internally?

Schemas are part of the prompt context; the model was fine-tuned to choose tools and emit arguments in a strict format. The provider parses and returns them as structured objects.

## 8. How is it different?

| | Valid arguments | Invalid arguments |
|---|---|---|
| Your code | Execute | Return error message to LLM → it retries/clarifies |
| Never | — | Execute blindly, or crash the app |

## 9. Common Misunderstandings

- ❌ "Better model = fewer schema problems." Still validate everything — models mis-arg constantly.
- ❌ "Descriptions don't matter." They are the ONLY thing the model knows about the tool.
- ❌ "The LLM handles security." Absolutely not — validation and authorization are always server-side.

## 10. Connection to Previous

Previous: the concept of tools and the round-trip. This file: making the round-trip *correct and safe* with schemas and validation.

## 11. Connection to Next

One tool round-trip is still not an agent. Wrap think→act→observe in a loop that the model controls — **Agentic AI**.

## 12. One-Minute Explanation

"A tool schema tells the LLM what a function does and what arguments it needs — the description is the only thing the model sees, so it steers tool choice. The lifecycle: register schemas, the model emits a tool call, my code validates the arguments, executes, returns the result as a message, and the model writes the final answer. Invalid arguments go back as error messages instead of crashing, authorization is always in my code, and dangerous actions need human confirmation."
