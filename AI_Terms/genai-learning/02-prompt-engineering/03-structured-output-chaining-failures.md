# Structured Output & Prompt Chaining (+ Common Failures)

## 1. What is it?

- **Structured output**: forcing the LLM to respond in a fixed machine-readable format — usually JSON.
- **Prompt chaining**: connecting multiple prompts in sequence, where each step's output feeds the next.
- **Common failures**: the predictable ways prompts break (described below).

## 2. Why does it exist?

An app cannot use free-form prose. Your code needs to parse the answer: extract fields, route logic, store results. And one prompt doing 5 tasks at once does all 5 badly — chaining splits the work so each step is simple and checkable.

## 3. Simple Mental Model

Structured output = asking the intern to fill a form, not write an essay. Chaining = an assembly line of small, reliable steps instead of one giant do-everything request.

## 4. How it Works

**Structured output:**

```
Extract the name, email, and order id.

Return ONLY valid JSON:
{"name": string, "email": string, "order_id": string}

Text: """Hi, I'm Ravi (ravi@mail.com), my order was #A-1023..."""
```

Modern APIs also support a stricter mode: give a JSON schema, provider enforces it.

**Chaining:**

```
Step 1 prompt: extract the complaint
      |  output
      v
Step 2 prompt: classify severity
      |  output
      v
Step 3 prompt: draft a reply
      |
      v
Final answer
```

## 5. ASCII: Chain with validation

```
Prompt 1 -> LLM -> output -> [valid?] --no--> retry / fix prompt
                       |yes
                       v
                   Prompt 2 -> LLM -> output -> [valid?] -> ... -> done
```

## 6. Important Terms

- **JSON schema**: a formal description of required fields and types.
- **Parser**: your code that reads the structured output.
- **Step boundary**: a deliberate split point between two prompts.

## 7. What happens internally?

Nothing magical — the format instruction and examples shift the probability of token sequences toward valid JSON. Strict schema modes constrain generation at the token level so only schema-valid tokens are allowed.

## 8. Common Prompt Failures & Fixes

| Failure | Symptom | Fix |
|---|---|---|
| Vague instruction | Random output | Add constraints, examples |
| Data/instruction mixing | Model follows text inside data | Delimiters |
| Format drift | JSON breaks sometimes | Schema mode, validate + retry |
| Hallucinated fields | Invented values | "If not found, use null" |
| Too much in one prompt | Mediocre at everything | Chain into steps |
| Context overflow | Forgotten parts of prompt | Trim/summarize (see tokens file) |

## 9. Common Misunderstandings

- ❌ "Asking for JSON guarantees JSON." Without a schema mode or validation, it usually will — but "usually" is not production-grade.
- ❌ "Chaining is slower so it's worse." Chained small steps are often more accurate AND easier to debug than one giant prompt.

## 10. Connection to Previous

Previous: examples and roles shape behavior. This file shapes *output* (structure) and *architecture* (chains) — the last prompt-layer skill.

## 11. Connection to Next

Prompt chaining runs inside software — which means real API calls, message roles, and costs. Next folder: **LLM Integration**.

## 12. One-Minute Explanation

"Structured output forces the LLM to answer in a machine-readable format like JSON — described in the prompt or enforced by a schema mode — so my application can parse it. Prompt chaining splits a complex task into sequential LLM calls where each output feeds the next, making each step simple and testable. I always validate structured output and retry on parse failure, since format drift is a real production issue."

## Hinglish Explanation

Structured output matlab LLM ko bolna "essay mat likh, sirf JSON bhar" — taaki aapka code usse parse kar sake. Prompt chaining matlab ek bada task ko chhote steps mein todna: pehla prompt output nikalta hai, wahi dusre prompt ka input banta hai. Har step chhota = zyada reliable aur debug karna easy.
