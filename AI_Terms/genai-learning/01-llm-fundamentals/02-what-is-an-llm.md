# What is an LLM?

## 1. What is it?

**LLM = Large Language Model.** A transformer neural network, made *large* (billions of parameters) and trained on *massive text* (internet, books, code), so it can generate human-like text.

Examples: GPT-4, Claude, Gemini, Llama, Groq-hosted Llama models.

## 2. Why does it exist?

Transformers were good at language, but small models were limited. Scaling up (more data + more parameters) made models generalize: translate, summarize, write code, answer questions — all with ONE model, without being specifically programmed for each task.

## 3. Simple Mental Model

An LLM is an **extremely well-read autocomplete**. It has "read" enormous text and learned patterns of how language flows, so given your text, it predicts the most likely continuation.

**Important:** It generates text token by token — one small piece at a time.

## 4. How it Works (generation loop)

```
Input: "The capital of France is"
  -> model predicts: " Paris"
Input: "The capital of France is Paris"
  -> model predicts: "."
Input: "The capital of France is Paris."
  -> model predicts: " It"
  ... continues until a stop condition
```

Each step:
1. All tokens so far are fed in.
2. Model outputs probability for every possible next token.
3. One token is chosen (controlled by temperature — next files).
4. Repeat.

## 5. ASCII Architecture

```
+----------------+        +-------------------+        +----------------+
|   Your text    | -----> |   LLM (transformer| -----> | Generated text |
|   (prompt)     |        |   layers + params)| <----- | (token by token|
+----------------+        +-------------------+  loop  |  until done)   |
                                                    +----+-----------+
```

## 6. Important Terms

- **Generative model**: creates new content (vs classifiers that only pick a label). LLMs are generative.
- **Generative AI**: the whole field of models that generate text, images, audio, code. LLM is the *text* part of Generative AI.

### LLM vs Generative AI (common confusion!)

| | Generative AI | LLM |
|---|---|---|
| Scope | Any content generation | Text only |
| Examples | Image models (DALL·E), music models | GPT, Claude, Llama |
| Relationship | The big umbrella | One important member |

## 7. What happens internally?

Your text → tokens → embeddings (numbers) → many transformer layers with attention → probability distribution over next token → pick → repeat.

## 8. How is it different?

| | Search engine | LLM |
|---|---|---|
| Output | Links to existing pages | Brand-new generated text |
| Knowledge | Live web index | Frozen at training date |
| Failure mode | No results found | Confident wrong answer (hallucination) |

## 9. Common Misunderstandings

- ❌ "It has a database of answers." No — knowledge is baked into parameters as patterns.
- ❌ "It knows the current date/news." Training data has a cutoff.
- ❌ "One LLM is best at everything." Models trade off size, speed, cost, capability.

## 10. Connection to Previous

Previous file: transformer = the architecture. This file: a transformer scaled up + trained on huge text = LLM.

## 11. Connection to Next

To actually use an LLM you must understand its input unit — **tokens** — and its memory limit — the **context window**.

## 12. One-Minute Explanation

"An LLM is a large transformer trained on massive text to predict the next token, again and again, until a full response is generated. It's generative — it creates text fresh each time rather than fetching stored answers. Generative AI is the broader field (images, audio too); LLM is its text-focused part. Its knowledge is frozen at training time, which is why it can hallucinate."
