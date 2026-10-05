# JIRA-15 — Vector Database Explorer Screen

> **Roadmap module:** 15 — Vector Databases
> **Epic:** Agentic AI Full Stack — Android Client
> **Backend dependency:** NAI-AI-15 (ChromaDB vector store — POST /v1/rag/search)

---

You are implementing JIRA-15 — Vector Database Explorer Screen for Nexus AI.

## Jira Title

Vector Database Explorer Screen

## Description

Deliver a vector database explorer screen where users can run similarity searches, filter by
document metadata, inspect individual chunk passages with relevance scores, and view collection
statistics. Calls the ChromaDB-backed FastAPI search endpoints from NAI-AI-15.

## Required Skills

- android
- compose
- testing

## Pre-Implementation Checklist

Before making any changes:

1. Read and understand:
   - This ticket and all Acceptance Criteria below
   - `.kiro/steering/` — all files
   - `.kiro/skills/android/SKILL.md`, `.kiro/skills/compose/SKILL.md`
2. Inspect JIRA-08 FastAPI dashboard — reuse the primary Retrofit client.
3. Inspect JIRA-14 RAG screen — reuse `SourceReference` domain model and relevance score bar.
4. Relevance scores are always normalised [0–1] — never raw distances.

## Architecture Rules

- `VectorSearchRequest` domain model: query, topK, documentIdFilter (optional).
- `VectorSearchResult` domain model: chunkId, documentId, source, page, content, relevanceScore.
- Document ID filter is a chip multi-select — applied as the `where` filter in the request body.
- Collection stats are fetched separately from `GET /v1/rag/stats`.

## Implementation Task

### Feature: Vector DB Explorer (`feature/vector-db/`)
- Search input `TextField` with Search button
- Top-K stepper (1–20, default 5)
- Document filter section:
  - "Load Documents" button → `GET /v1/rag/documents` → populates filter chips
  - Multi-select chips for document IDs (max 3 visible, "+N more" overflow)
- Collection stats card (auto-loaded): total chunks, collection name
- Search results list:
  - Result card: rank number, relevance score bar (colour-coded), source, page
  - Expand button → shows full chunk content in a monospace text block
  - Copy Chunk button → copies content to clipboard
- Empty results card: "No matching chunks found"
- Relevance score legend: green ≥ 0.8 "High", amber ≥ 0.5 "Medium", red < 0.5 "Low"

### Domain
- `VectorSearchRequest` data class: query, topK, documentIdFilter
- `VectorSearchResult` data class: rank, chunkId, documentId, source, page, content, relevanceScore
- `CollectionStats` data class: collectionName, totalChunks
- `SearchVectorStoreUseCase`, `GetCollectionStatsUseCase`, `GetDocumentListUseCase`

## Acceptance Criteria

| # | Criterion |
|---|---|
| AC1 | Search input submits a query and displays top-K results. |
| AC2 | Each result card shows rank, relevance score bar, source, and page. |
| AC3 | Relevance score bars are colour-coded (green / amber / red) with a legend. |
| AC4 | Document filter chips restrict results to selected documents. |
| AC5 | Collection stats card shows total chunk count and collection name. |
| AC6 | Full chunk content is shown on expand in a monospace text block. |
| AC7 | Copy Chunk copies the full content to the clipboard. |
| AC8 | Empty results show a clear notice card. |
| AC9 | Use cases and `VectorDbViewModel` are unit tested with fake repositories. |
| AC10 | Debug build succeeds. |

## Workflow

1. Define `VectorSearchRequest`, `VectorSearchResult`, `CollectionStats` domain models.
2. Implement `VectorDbRepository` interface and impl.
3. Implement three use cases.
4. Implement `VectorDbViewModel`.
5. Implement `VectorDbScreen` with search, filter chips, result cards.
6. Write unit tests.
7. Run: `./gradlew testDebugUnitTest assembleDebug`.
8. Verify every AC individually.

## Final Response Format

Provide:
- Implementation Summary
- Files Created / Modified
- Search Result Cards Evidence (score bar shown)
- Filter Evidence (filtered vs unfiltered result counts)
- Tests Executed and Results
- AC1–AC10 PASS/FAIL with evidence
- Known Limitations
- Recommended Next Ticket: JIRA-16

> Never claim an AC is PASS without evidence.
