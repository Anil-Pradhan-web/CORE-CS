# Tokens and Context Window

## 1. What is it?

- **Token**: the small piece of text an LLM actually reads and writes. A token is roughly a word, part of a word, or punctuation. "unbelievable" might be 3 tokens: "un" + "believ" + "able".
- **Context Window**: the maximum number of tokens the model can process in ONE request (input + output together). Like the model's short-term memory.

## 2. Why does it exist?

Models process numbers, not words. Text is split into tokens and mapped to numbers. And because the model computes attention across all tokens at once, there must be a hard limit — that limit is the context window (e.g., 8K, 128K, 1M tokens depending on the model).

## 3. Simple Mental Model

Tokens = Lego bricks of text. Context window = the size of the table you can lay bricks on. Anything that doesn't fit on the table gets ignored — the model literally cannot see it.

## 4. How it Works

```
"Artificial Intelligence is fun"
        |
        v  Tokenizer
["Art", "ificial", " Intelligence", " is", " fun"]   (5 tokens)
        |
        v
[123, 4562, 8765, 374, 2291]   (numbers fed to the model)
```

Rules of thumb:
- ~1 token ≈ 4 characters in English ≈ 0.75 words.
- Context = system prompt + conversation history + your data + the answer. ALL of it shares the same window.

## 5. ASCII Architecture

```
+-------------------- CONTEXT WINDOW (e.g. 8,192 tokens) --------+
|  System prompt | Chat history | Retrieved documents | Question |  <- New answer |
+----------------------------------------------------------------+
        ^ older stuff may need trimming as conversation grows ^^^
```

## 6. Important Terms

- **Tokenizer**: the component that splits text into tokens.
- **Context overflow**: when input exceeds the window → request fails, or old content gets truncated.
- **"Lost in the middle"**: models recall the start and end of long context better than the middle.

## 7. What happens internally?

Tokens → token IDs → vectors → attention compares every token with every other token (this is why cost/limit scales with token count).

## 8. How is it different?

| | Token | Embedding |
|---|---|---|
| What | A chunk of text | A vector of numbers |
| Role | Unit of input/output | Unit of MEANING (used for search) |
| Where | Tokenizer | Embedding model |

(Big interview confusion — embeddings come in detail in the semantic search folder.)

## 9. Common Misunderstandings

- ❌ "Context is unlimited chat memory." It resets every request; long chats must be truncated or summarized.
- ❌ "1 word = 1 token." Often wrong — that's why costs surprise people.
- ❌ "Bigger context = the model reads everything carefully." Long context can degrade quality.

## 10. Connection to Previous

Previous: the LLM generates token by token. Now you know what a token IS and the window that limits everything you can send.

## 11. Connection to Next

There are two big "training stages" behind a model like this — pre-training, fine-tuning — and the moment you actually use it is called inference.

## 12. One-Minute Explanation

"LLMs don't read words — they read tokens, small chunks of text mapped to numbers. The context window is the max tokens the model can see in one request; system prompt, history, documents, and the answer all share it. When context overflows, content must be trimmed. This limit is exactly why we later need chunking and retrieval in RAG."
