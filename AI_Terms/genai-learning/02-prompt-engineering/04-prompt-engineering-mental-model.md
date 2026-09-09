# Prompt Engineering — Mental Model (Folder Summary)

## 1. What is it?

A one-page map of everything in this folder, so it sticks as ONE skill: controlling the LLM through its input.

## 2. Why this summary exists

Four files of details collapse into a single picture you can recall in an interview.

## 3. The Mental Model

You are writing a **specification**, not a question. Every prompt answers:

```
WHO  is the model?          (role)
WHAT should it do?          (instruction)
WHAT material does it use?  (context)
WHAT are the limits/rules?  (constraints)
WHAT shape is the output?   (format / examples)
```

## 4. Technique Ladder (weakest → strongest control)

```
Vague question
  -> + Instruction & constraints      (say exactly what)
  -> + Role                            (who it should be)
  -> + Delimiters                      (separate data from instructions)
  -> + Few-shot examples               (show, don't tell)
  -> + Structured output / schema      (machine-readable answer)
  -> + Chaining                        (split into steps)
```

Each rung = more predictable behavior.

## 5. ASCII: Where prompts live in an app

```
+----------------------------------------------------+
| Application                                        |
|   System prompt (fixed, app-controlled)            |
|        +                                           |
|   Retrieved context (RAG - later folder)           |
|        +                                           |
|   Few-shot examples (optional)                     |
|        +                                           |
|   User message                                     |
+-------------------------+--------------------------+
                          v
                        LLM
```

## 6. Important Terms recap

System prompt, few-shot, delimiter, structured output, schema, chaining, prompt injection, constraint.

## 7. Internals recap

The entire prompt is one token sequence; attention weighs your structure; examples and formats shift probabilities; schema modes constrain generation.

## 8. Comparisons recap

| Prompt | Fine-tuning | Tool calling |
|---|---|---|
| Controls behavior per call | Controls behavior permanently | Lets model trigger actions |
| Free | Costly | Needs function code |

## 9. Common Misunderstandings recap

Prompts don't teach facts; longer ≠ better; JSON requests aren't guaranteed JSON; one mega-prompt loses to chained small steps.

## 10. Connection to Previous

LLM fundamentals showed WHY the model is steerable-but-unreliable. Prompts are the steering wheel.

## 11. Connection to Next

Prompts only exist inside a request. Next: how software actually calls an LLM — **LLM Integration** (API, messages, streaming, cost).

## 12. One-Minute Explanation

"Prompt engineering is writing a spec to the model: role, instruction, context, constraints, and output format. I control behavior by climbing a ladder — clear instructions, roles, delimiters, few-shot examples, structured output, and chaining. It's free and instant, unlike fine-tuning. In an application, the system prompt is fixed by me and combined with context and the user's message in every request."
