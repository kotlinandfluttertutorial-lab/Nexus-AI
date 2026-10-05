# Skill: Vector Databases

## Purpose

Provides reusable engineering expertise for integrating ChromaDB (and other vector stores) into
the Nexus AI backend, covering collection management, document insertion, similarity search,
metadata filtering, index persistence, and a replaceable adapter interface.

## When to Use

Activate this skill when:
- Setting up or configuring a ChromaDB collection
- Inserting document chunks with embeddings and metadata
- Running similarity searches with optional metadata filters
- Implementing the `VectorStore` domain interface
- Writing a `FakeVectorStore` for deterministic tests
- Evaluating retrieval quality by comparing configurations

## Core Rules

1. **Domain interface first.** The `VectorStore` interface is defined in the domain layer. ChromaDB is an infrastructure implementation behind an adapter.
2. **Metadata is mandatory.** Every inserted document must carry at minimum: `document_id`, `chunk_id`, `source`, `page` (where available).
3. **Collections are named and isolated.** Use separate collections per document corpus — never mix unrelated documents in one collection.
4. **Embeddings are injected.** The vector store adapter receives pre-computed embeddings — it does not generate them.
5. **Top-K is bounded.** Never return unbounded results; always specify `n_results`.
6. **Distance is normalized.** Map raw distance scores to a `[0, 1]` relevance score before returning to callers.
7. **Persistence is explicit.** In production, use a persistent ChromaDB client with a defined `persist_directory`. Never rely on in-memory state in production.

## Project Structure

```
backend/rag/
├── vector_store/
│   ├── interface.py              — VectorStore protocol
│   ├── chroma_adapter.py         — ChromaDB implementation
│   ├── fake_vector_store.py      — In-memory fake for tests
│   └── collection_config.py     — Collection naming and settings
├── embedding/
│   ├── interface.py              — EmbeddingProvider protocol
│   ├── openai_embedding.py
│   └── sentence_transformer_embedding.py
└── tests/
    ├── test_chroma_adapter.py
    └── test_retrieval_quality.py
```

## Implementation Patterns

### VectorStore Interface

```python
from typing import Protocol
from dataclasses import dataclass

@dataclass
class DocumentChunk:
    chunk_id: str
    document_id: str
    source: str
    page: int | None
    content: str
    embedding: list[float]
    metadata: dict

@dataclass
class SearchResult:
    chunk: DocumentChunk
    relevance_score: float  # 0.0 (low) to 1.0 (high)

class VectorStore(Protocol):
    async def insert(self, chunks: list[DocumentChunk]) -> None: ...
    async def search(
        self,
        query_embedding: list[float],
        top_k: int = 5,
        filters: dict | None = None,
    ) -> list[SearchResult]: ...
    async def delete(self, document_id: str) -> None: ...
    async def count(self) -> int: ...
```

### ChromaDB Adapter

```python
import chromadb
from chromadb.config import Settings

class ChromaVectorStoreAdapter:
    def __init__(self, persist_directory: str, collection_name: str):
        self._client = chromadb.PersistentClient(
            path=persist_directory,
            settings=Settings(anonymized_telemetry=False),
        )
        self._collection = self._client.get_or_create_collection(
            name=collection_name,
            metadata={"hnsw:space": "cosine"},
        )

    async def insert(self, chunks: list[DocumentChunk]) -> None:
        self._collection.upsert(
            ids=[c.chunk_id for c in chunks],
            embeddings=[c.embedding for c in chunks],
            documents=[c.content for c in chunks],
            metadatas=[{
                "document_id": c.document_id,
                "source": c.source,
                "page": c.page or 0,
                **c.metadata,
            } for c in chunks],
        )

    async def search(
        self,
        query_embedding: list[float],
        top_k: int = 5,
        filters: dict | None = None,
    ) -> list[SearchResult]:
        kwargs = {"query_embeddings": [query_embedding], "n_results": top_k, "include": ["documents", "metadatas", "distances"]}
        if filters:
            kwargs["where"] = filters
        results = self._collection.query(**kwargs)
        return self._map_results(results)

    def _map_results(self, raw) -> list[SearchResult]:
        out = []
        for doc, meta, dist in zip(raw["documents"][0], raw["metadatas"][0], raw["distances"][0]):
            # cosine distance: 0 = identical, 2 = opposite → relevance = 1 - dist/2
            relevance = max(0.0, 1.0 - dist / 2.0)
            chunk = DocumentChunk(
                chunk_id=meta.get("chunk_id", ""),
                document_id=meta["document_id"],
                source=meta["source"],
                page=meta.get("page"),
                content=doc,
                embedding=[],
                metadata=meta,
            )
            out.append(SearchResult(chunk=chunk, relevance_score=relevance))
        return out
```

### Fake Vector Store for Tests

```python
class FakeVectorStore:
    def __init__(self, fixed_results: list[SearchResult] | None = None):
        self._stored: list[DocumentChunk] = []
        self._fixed_results = fixed_results or []

    async def insert(self, chunks: list[DocumentChunk]) -> None:
        self._stored.extend(chunks)

    async def search(self, query_embedding, top_k=5, filters=None) -> list[SearchResult]:
        return self._fixed_results[:top_k]

    async def delete(self, document_id: str) -> None:
        self._stored = [c for c in self._stored if c.document_id != document_id]

    async def count(self) -> int:
        return len(self._stored)
```

## Testing Guidance

- Unit-test retrieval logic using `FakeVectorStore` with controlled result sets
- Integration-test the `ChromaVectorStoreAdapter` with an in-memory ChromaDB client (`chromadb.EphemeralClient()`)
- Test metadata filters by inserting chunks with different `document_id` values and verifying filter isolation
- Test `delete` by inserting, deleting, then confirming `count()` decreases
- Test `top_k` boundary: insert 3 chunks, request 5, verify 3 returned

## Common Mistakes

| Mistake | Correct Approach |
|---|---|
| Importing ChromaDB in domain layer | Use the `VectorStore` protocol; ChromaDB stays in infrastructure |
| Generating embeddings inside the adapter | Embeddings are computed separately and passed in |
| Forgetting `anonymized_telemetry=False` | Always disable telemetry in production |
| Using in-memory client in production | Use `PersistentClient` with a defined `persist_directory` |
| Returning raw distance as a score | Normalize to `[0, 1]` relevance before returning |
| Not setting `n_results` | Always bound the query with an explicit `top_k` |

## Relationship to Steering

This skill governs `backend/rag/vector_store/`. ChromaDB types must not appear in domain interfaces. The Android RAG implementation (`rag` skill) uses a separate in-app vector store abstraction.
