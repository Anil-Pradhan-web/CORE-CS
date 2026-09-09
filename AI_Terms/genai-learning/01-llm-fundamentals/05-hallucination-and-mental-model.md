# Hallucination, Limitations & the LLM Mental Model

## 1. What is it?

**Hallucination**: when an LLM confidently produces information that is wrong or invented — fake facts, fake citations, made-up API names.

## 2. Why does it happen?

Because of HOW the model works, not a bug:
- It predicts *likely* text, not *verified true* text.
- Its knowledge is frozen at training time.
- If it doesn't know, the most "likely sounding" continuation may still be fiction.

```
Question: "Who wrote the book 'Xylophonic Dreams'?"
LLM thinking: "A book title like this usually has an author name next to it..."
Answer:      "It was written by Professor Alan Reeves."   <- invented, said confidently
```

## 3. Simple Mental Model

An LLM is a brilliant improviser, not a librarian. Ask it about something it never read, it will improvise a plausible-sounding answer instead of saying "I don't know".

## 4. Core Limitations List

| Limitation | Meaning | Later fix |
|---|---|---|
| Hallucination | Confident wrong answers | RAG, validation |
| Knowledge cutoff | Doesn't know recent events | RAG, tools (web search) |
| No private data | Doesn't know your company docs | RAG |
| Can't act | Can only produce text, not run things | Tool calling |
| Context limit | Forgets beyond the window | Chunking, retrieval |
| Math/reasoning slips | Token prediction ≠ calculator | Tools (calculator), lower temperature |

## 5. ASCII: Hallucination Risk Zones

```
Question about TRAINING knowledge  -> low risk
Question about obscure/private data -> HIGH risk (will improvise!)
Question about recent events        -> HIGH risk (data is frozen)
Calculation                          -> risk (it "guesses" patterns, not computes)
```

## 6. Important Terms

- **Grounding**: anchoring the answer in real provided facts (RAG does this).
- **Knowledge cutoff**: the date of the newest data in training.

## 7. What happens internally?

The model fills the statistically most plausible next tokens. For unknown topics, "plausible" and "true" are different things — that gap is a hallucination.

## 8. How is it different?

| | Lying | Hallucination |
|---|---|---|
| Intent | Deliberate | None — model has no intent |
| Cause | Choice | Statistical generation |
| Fix | Trust | Grounding, RAG, validation |

## 9. Common Misunderstandings

- ❌ "Bigger models never hallucinate." They reduce it, not eliminate it.
- ❌ "It hallucinates randomly." Risk is highest exactly where it lacks knowledge.
- ❌ "I'll just ask it to 'only tell the truth'." Instructions reduce but cannot remove hallucination — you need real facts in the context (RAG).

## 10. Connection to Previous

Previous file: how the model generates (prediction, temperature). Hallucination is the direct side effect of prediction — now you see the model's honest limits.

## 11. Connection to Next

Since hallucination comes from *asking a blind model*, the first tool to control it is a well-designed **prompt** — the next folder.

## 12. One-Minute Explanation

"Hallucination is when an LLM confidently invents information. It happens because the model predicts plausible text, not verified facts, and its knowledge is frozen at training time. We reduce it with grounding — like RAG, which injects real retrieved documents into the prompt — and by validating outputs. But we can never fully remove it, so production systems must always assume the model can be wrong."

## Hinglish Explanation

Hallucination matlab LLM confident hokar galat ya banawati baat bolna. Ye bug nahi hai — model ka kaam hi "likely text predict karna" hai, "verified truth" nahi. Isliye jab usko answer nahi pata, wo sensible-sounding guess bana deta hai. Iska main ilaaj hai RAG — real documents context mein dena — aur output validate karna.
