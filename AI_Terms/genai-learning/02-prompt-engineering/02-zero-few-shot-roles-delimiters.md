# Zero-shot, Few-shot, Role Prompting & Delimiters

## 1. What is it?

- **Zero-shot**: asking the task with NO examples — "Classify this review as positive or negative."
- **Few-shot**: showing a few examples INSIDE the prompt, then asking — the model copies the pattern.
- **Role prompting**: "You are a senior security auditor..." — sets tone, depth, and vocabulary.
- **Delimiters**: markers (```, """, ###) that separate instructions from data.

## 2. Why does it exist?

Words alone are ambiguous. Examples remove ambiguity better than any description — the model pattern-matches from examples. Roles focus the model's "personality" toward relevant knowledge. Delimiters prevent the model from treating your data as instructions.

## 3. Simple Mental Model

Zero-shot = asking someone to do a task blind. Few-shot = showing them 3 solved samples first: "like this, now you do it."

## 4. How it Works (few-shot pattern)

```
Classify the sentiment.  <- instruction

Review: "Great product!"     -> positive
Review: "Broke in 2 days."   -> negative
Review: "Okay for price."    -> neutral     <- examples (few-shot)

Review: "Worst purchase ever." -> ?         <- the real input
```

The model continues the established pattern. Even 2–5 examples dramatically improve format consistency.

## 5. ASCII

```
+--------------------------------------+
| SYSTEM: You are a code reviewer.     |  <- role
+--------------------------------------+
| USER:                                |
| Follow the format in the examples.   |  <- instruction
| <code>                               |  <- delimiter + data
|   your_code_here                     |
| </code>                              |
| Example: bad code -> [BUG] severity  |  <- few-shot
| Now review: <code>...new...</code>   |
+--------------------------------------+
```

## 6. Important Terms

- **Shot**: one example. Zero-shot = none, one-shot = 1, few-shot = 2–10.
- **Pattern anchoring**: examples fix the output shape in the model's attention.
- **Prompt injection**: when user data contains malicious "instructions" — delimiters + constraints are the first defense (fully covered later).

## 7. What happens internally?

All examples become part of the token sequence. Attention picks up the repeated structure, so the continuation follows it. Role text activates the probability distribution of that domain's vocabulary.

## 8. How is it different?

| | Zero-shot | Few-shot | Fine-tuning |
|---|---|---|---|
| Examples in prompt | 0 | 2–10 | 0 (trained-in) |
| Cost | Free | A few extra tokens | Training required |
| Use when | Simple task | Need consistent format | Permanent behavior |

## 9. Common Misunderstandings

- ❌ "More examples are always better." They cost tokens and can bias the model; 3 good examples usually beat 10 noisy ones.
- ❌ "Few-shot = training." No training happens — it's all within one inference.
- ❌ "Role prompting is just flavor text." It measurably changes vocabulary, depth, and caution.

## 10. Connection to Previous

Previous: prompt anatomy (role/instruction/context/constraints). This file adds the two strongest upgrades: examples and roles, plus delimiters to protect the structure.

## 11. Connection to Next

Often the output must go into another program — which needs a guaranteed format like JSON. That is **structured output**.

## 12. One-Minute Explanation

"Zero-shot asks a task with no examples; few-shot puts 2–5 solved examples in the prompt so the model pattern-matches the format. Role prompting sets expertise and tone ('you are a security auditor'). Delimiters like ``` separate data from instructions and are the first defense against prompt injection. All of these are just prompt text — no training happens."

## Hinglish Explanation

Zero-shot matlab bina example ke task dena. Few-shot matlab prompt ke andar 2–5 examples dikha dena — model wahi pattern copy karta hai. Role prompting matlab "tum ek senior engineer ho" type setting, jisse answer ka tone aur depth badalta hai. Delimiters (```) se aap data ko instructions se alag karte ho, warna user ke text mein chhupi instructions bhi model follow kar sakta hai.
