# Embeddings in RAG & the Vector Database

## 1. What is it?

- **Embeddings in RAG**: every chunk is converted to a vector at ingest time; the user's question is converted to a vector at query time. Same embedding model must be used for both.
- **Vector database**: a database purpose-built to store millions of vectors + metadata and answer "nearest neighbors?" fast (Pinecone, Qdrant, Weaviate, Chroma, pgvector).

## 2. Why does it exist?

FAISS (a library) is fast but you must handle storage, updates, scaling, metadata filtering, and backups yourself. A vector DB is the production version: persistence, filtering, CRUD, replication, hybrid search — all built in.

## 3. Simple Mental Model

The vector DB is a library where every book is filed by *meaning*, not by title. You ask in your own words; it fetches by meaning and hands you the actual text.

## 4. How it Works

```
INGEST (per chunk):
chunk text + metadata -> embedding model -> vector -> upsert into DB

QUERY:
question -> SAME embedding model -> query vector
         -> ANN search (+ metadata filter) -> top-k vectors
         -> map back to chunk text + metadata -> return
```

Critical rule: **embedding mismatch** (different model for docs vs query) makes vectors incomparable — results become garbage.

## 5. ASCII Architecture

```
+-------------------------------------------+
|              VECTOR DATABASE              |
|  +--------+  +---------+  +------------+  |
|  |Vectors |  |Metadata |  |Text lookup |  |
|  |(ANN)   |  |filtering|  |id -> chunk |  |
|  +--------+  +---------+  +------------+  |
+-----------------+-------------------------+
                  ^
        query vector + filter
        (e.g. source=hr_docs, date>2024)
```

## 6. Important Terms

- **Upsert**: insert or update a vector by id.
- **Metadata filtering**: pre-filter (e.g., only HR docs) before similarity search.
- **Hybrid search**: combine vector (meaning) + keyword (exact terms like error codes) search.
- **Namespace/collection**: logical separation of data (per tenant, per product).

## 7. What happens internally?

The DB keeps vectors in an ANN index; a query finds candidate clusters, scores them, applies filters, and resolves vector ids back to the stored text.

## 8. How is it different?

| | Vector DB | SQL DB |
|---|---|---|
| Query | "similar meaning" | exact conditions |
| Index | ANN vectors | B-trees |
| Example | "docs about refunds" | `WHERE status='refunded'` |

Modern apps often use **pgvector** — SQL and vector search together.

## 9. Common Misunderstandings

- ❌ "The vector DB stores answers." It stores vectors + text; intelligence stays in the LLM.
- ❌ "Any embedding model works." All text must use ONE model, same dimensions.
- ❌ "Vector search alone is enough." Keyword/hybrid helps for exact terms (IDs, error codes) that embeddings blur.

## 10. Connection to Previous

Previous: chunking produced the pieces. This file: converting pieces to vectors and storing them retrievably.

## 11. Connection to Next

Storage is ready — now the actual act of *fetching the right chunks* for a question: retrieval.

## 12. One-Minute Explanation

"In RAG, chunks are embedded once at ingest and the question is embedded at query time — with the same model, or the vectors don't align. The vector DB stores those vectors with metadata and provides fast ANN search plus filtering and persistence — the production version of FAISS. Query flow: embed question, optionally filter by metadata, find top-k nearest vectors, map ids back to chunk text. Hybrid search adds keyword matching for exact terms like error codes."
