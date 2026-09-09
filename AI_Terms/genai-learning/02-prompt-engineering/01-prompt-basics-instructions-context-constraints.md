# What is Prompt Engineering? Instructions, Context & Constraints

## 1. What is it?

**Prompt engineering** = designing the text you send to the LLM so it produces the output you actually want.

A good prompt usually has 4 parts:

```
1. ROLE       - "You are a helpful SQL expert."
2. INSTRUCTION- "Convert the question into SQL."
3. CONTEXT    - schema of the tables, past messages
4. CONSTRAINTS- "Return ONLY the SQL. No explanations."
```

## 2. Why does it exist?

The LLM has no context about your app, your goals, or your format needs. It only sees your text. A vague prompt = a guess. A structured prompt = a specification. It is the cheapest, fastest way to control an LLM (vs fine-tuning which needs data, money, training).

## 3. Simple Mental Model

The LLM is a smart intern with amnesia: brilliant but zero background. Every prompt must carry everything needed: who you are, what to do, what you know, and what NOT to do.

## 4. How it Works

- **Instructions** say the task: "Summarize this email in 2 lines."
- **Context** gives material: the email text, user data, retrieved documents.
- **Constraints** limit behavior: "If unsure, say unknown. Do not invent names."
- The model treats the entire prompt as one token sequence — clear, ordered prompts let attention find the right parts.

## 5. ASCII

```
Bad prompt:                         Good prompt:
"tell me about dogs"                "You are a vet assistant.
                                     Explain in 3 bullet points,
                                     max 20 words each, suitable
                                     for a 10-year-old, the topic: dogs"
        |                                   |
        v                                   v
  random-length answer              exactly the shape you asked for
```

## 6. Important Terms

- **System prompt**: instructions set by the app, hidden from/above the user's message (detailed in LLM-integration folder).
- **Delimiter**: a marker like `"""` or `###` that separates "instructions" from "data" so the model doesn't confuse them.
- **Instruction vs data confusion**: if user text is pasted raw, the model may follow instructions hidden inside it — delimiters and constraints prevent this.

## 7. What happens internally?

The whole prompt is tokenized together. Attention weighs your instruction against your context. Clear structure and explicit constraints shift probability mass toward the behavior you want.

## 8. How is it different?

| | Prompt Engineering | Fine-tuning |
|---|---|---|
| Change cost | Free, instant | Data + money + training |
| Best for | Behavior, format, per-request tasks | Fixed style/domain at scale |
| Knowledge update | Yes, via context | No (learns patterns, not facts) |

## 9. Common Misunderstandings

- ❌ "Longer prompt = better." Clear beats long.
- ❌ "Politeness matters." Structure and constraints matter.
- ❌ "One magic prompt works forever." Prompts are engineered per task and iterated.

## 10. Connection to Previous

Previous folder ended with hallucination — "a blind model". Prompts are your first control lever: tell it exactly what to do and what not to invent.

## 11. Connection to Next

Two powerful prompt upgrades: showing **examples** (zero-shot vs few-shot) and giving the model a **role** — next file.

## 12. One-Minute Explanation

"Prompt engineering is designing the LLM's input like a spec: role, instruction, context, and constraints. Since the model only sees text and has no memory of your app, a structured prompt is the cheapest way to control output format, tone, and limits — much cheaper than fine-tuning. I think of the LLM as a smart intern who needs complete, precise briefings every single time."

## Hinglish Explanation

Prompt engineering matlab LLM ko aisa input dena jo uska output aapke hisaab se shape kare. Har prompt mein 4 cheezein honi chahiye: role (kaun hai tu), instruction (kya karna hai), context (material kya hai), constraints (kya nahi karna). LLM ko smart intern samjho jisko har baar poora brief dena padta hai.
