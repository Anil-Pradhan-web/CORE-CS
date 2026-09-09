# Document Ingestion & Chunking

## 1. What is it?

- **Document ingestion**: the offline phase where raw documents (PDFs, web pages, markdown, DB rows) are loaded, cleaned, and converted into searchable pieces.
- **Chunking**: splitting each document into small, meaningful pieces (chunks), because whole documents are too big for embeddings and prompts to work well.

## 2. Why does it exist?

You can't embed a 100-page PDF as one vector — the meaning gets diluted and the chunk won't fit in context. Retrieval precision requires small, self-contained pieces: you want to retrieve the *paragraph about refunds*, not the whole manual.

## 3. Simple Mental Model

Chunking = cutting a book into index cards. Each card should make sense alone. If a card contains half a sentence, it's useless; if it contains 10 chapters, it's too vague.

## 4. How it Works — Ingestion Pipeline

```
1. LOAD      - read PDFs/HTML/DOCX/DB -> plain text
2. CLEAN     - remove headers, boilerplate, broken characters
3. CHUNK     - split into ~200-800 token pieces
4. OVERLAP   - repeat ~10-20% of each chunk in the next
5. ATTACH METADATA - source file, page, section, date
6. EMBED     - each chunk -> vector
7. STORE     - vector + text + metadata in vector store
```

## 5. ASCII: Why overlap matters

```
Bad (hard cut):
[...refund policy within 30 days of purch][ase from the official store...]
   meaning split across chunks -> both chunks weak

Good (overlap):
[...refund policy within 30 days of purchase...]
                    [...days of purchase from the official store...]
   shared boundary text -> each chunk self-contained
```

## 6. Chunking Strategies

| Strategy | How | Best for |
|---|---|---|
| Fixed size | every N tokens | quick baseline |
| Recursive | split by paragraph → sentence → word | general purpose (most common) |
| Semantic | split where topic changes | high precision |
| Structure-based | by headings/sections | docs, manuals, code |

## 7. Important Terms

- **Chunk size**: tokens per chunk — small = precise retrieval but missing context; large = context but diluted vectors.
- **Overlap**: repeated text between consecutive chunks to protect boundary meaning.
- **Metadata**: labels (source, page, section) stored with each chunk for filtering and citations.

## 8. What happens internally?

Embedding a chunk produces ONE vector summarizing its meaning. A large mixed-topic chunk produces a blurred vector that matches nothing well. This is the mathematical reason chunk size matters.

## 9. Common Misunderstandings

- ❌ "Smaller chunks are always better." Too small = the retrieved text lacks context to answer from.
- ❌ "Chunking is a solved detail." It's often THE difference between a great and terrible RAG system.
- ❌ "Ingestion is one-time forever." Documents change; you need an update/re-index strategy.

## 10. Connection to Previous

Previous: RAG architecture overview. This file is Phase 1 — without good chunks, Phase 2 retrieves noise.

## 11. Connection to Next

Chunks exist to be embedded and stored — next: embeddings in RAG and the vector database.

## 12. One-Minute Explanation

"Ingestion converts raw documents into searchable pieces: load, clean, chunk, attach metadata, embed, store. Chunking splits text into roughly 200–800 token self-contained pieces, often with 10–20% overlap so boundary meaning survives. Chunk size is a precision-vs-context trade-off, and strategy matters — recursive splitting by paragraph/sentence is the common default. Retrieval quality, and therefore answer quality, depends heavily on this phase."

## Hinglish Explanation

Ingestion matlab documents ko process karna: load karo, saaf karo, chhote chunks mein todo, metadata lagao, embed karo, store karo. Chunking matlab book ko index cards mein kaatna — har card khud ka meaning ho. Overlap matlab do chunks ke beech thoda text repeat karna, taaki sentence ka meaning na kate. Chhota chunk = precise search, bada chunk = zyada context — dono ke beech balance chahiye.
