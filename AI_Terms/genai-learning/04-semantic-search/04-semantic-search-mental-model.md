# Semantic Search — Mental Model (Folder Summary)

## 1. What is it?

One page that compresses the whole folder: searching by meaning using embeddings.

## 2. Why this summary exists

Four files → one recallable picture.

## 3. The Mental Model

```
Text  --embedding model-->  Vector (meaning as direction)
Query --embedding model-->  Query vector
Vector search:  "which stored vectors point the same way?"
Result: top-k texts, ranked by meaning similarity
```

Meaning becomes geometry. Search becomes nearest-neighbor lookup.

## 4. The Complete Flow

```
[INDEXING]                        [QUERY]
docs -> chunks -> vectors         query -> query vector
              |                            |
              v                            v
        vector index  <----search------ top-k matches
                                           |
                                           v
                                 ranked text pieces
```

## 5. Key Decisions in Any Semantic Search System

| Decision | Options | Effect |
|---|---|---|
| Embedding model | small vs strong | quality of "meaning" |
| Chunk size | 100 vs 1000 tokens | precision vs context |
| Index type | flat vs ANN | exact vs fast |
| Top-k / threshold | 3–10 / 0.7 | recall vs noise |

## 6. Important Terms recap

Embedding, vector, dimension, cosine similarity, top-k, ANN, index, FAISS, vector DB, metadata.

## 7. Internals recap

Embedding models are transformers trained so similar-context text → similar directions. ANN indexes cluster vectors to skip most comparisons.

## 8. Comparisons recap

| Keyword | Semantic |
|---|---|
| Matches words | Matches meaning |
| Misses synonyms | Handles paraphrase |
| SQL/Elasticsearch | FAISS / vector DB |

| Token | Embedding |
|---|---|
| LLM input unit | Meaning unit for search |

## 9. Common Misunderstandings recap

Vectors don't store text; similarity ≠ correctness; embeddings ≠ tokens; FAISS ≠ database.

## 10. Connection to Previous

LLM integration let you call the model; it still doesn't know your data. Embeddings made your data searchable by meaning.

## 11. Connection to Next

Semantic search finds *relevant text*. RAG = relevant text **injected into the prompt** so the LLM answers from it. Same engine, new purpose.

## 12. One-Minute Explanation

"Semantic search converts documents and queries into embedding vectors and returns the top-k nearest by cosine similarity — so it matches meaning, not keywords. Indexing happens once; searching happens per query. FAISS or a vector DB provides fast approximate nearest-neighbor lookup at scale. This machinery is exactly what RAG reuses: retrieve relevant chunks, then let the LLM answer using them."
