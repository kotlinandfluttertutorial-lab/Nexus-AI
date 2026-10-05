# JIRA-14 — RAG Pipeline Screen

> **Roadmap module:** 14 — Retrieval-Augmented Generation
> **Epic:** Agentic AI Full Stack — Android Client
> **Backend dependency:** NAI-AI-14 (RAG pipeline — POST /v1/rag/ingest + POST /v1/rag/query)

---

You are implementing JIRA-14 — RAG Pipeline Screen for Nexus AI.

## Jira Title

RAG Pipeline Screen

## Description

Deliver the RAG pipeline screen where users pick a document, watch the ingestion pipeline
progress through Extract → Chunk → Embed → Store stages, then ask a question and receive an
answer with source references. Hybrid search mode can be toggled. Calls the FastAPI RAG
endpoints from NAI-AI-14.

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
2. Inspect JIRA-08 FastAPI dashboard — reuse Retrofit client.
3. Inspect JIRA-02 design system — reuse `NexusCard`, status step indicators.
4. Document content must never be logged — only document name and processing status.

## Architecture Rules

- Document is selected via Android Storage Access Framework (SAF) — not a custom file picker.
- Document bytes are uploaded via multipart form to `/v1/rag/ingest`.
- Pipeline stage status is polled or streamed — displayed as a stepper composable.
- `RagAnswer` domain model: answer, sources, emptyRetrieval, truncated.
- `SourceReference` domain model (reused from JIRA-12): documentId, source, page, relevanceScore.
- Hybrid search mode is a boolean sent in the query request body.

## Implementation Task

### Feature: RAG Pipeline Screen (`feature/rag/`)

#### Section 1 — Document Ingestion
- Pick Document button → SAF launcher (PDF / TXT / MD filter)
- Selected document chip: filename + size
- Ingest button → `POST /v1/rag/ingest` (multipart)
- Pipeline stepper (horizontal):
  - Extract → Chunk → Embed → Store
  - Each stage: idle / running (spinner) / done (checkmark) / failed (X with error message)
- Success card: "Document indexed — N chunks created"
- Failure card: failing stage name + error message

#### Section 2 — Query
- Question `TextField` + Hybrid Search toggle chip
- Ask button → `POST /v1/rag/query`
- Answer card:
  - Answer text
  - Truncated notice badge if context was truncated
  - Empty retrieval notice if no sources found
- Source reference cards (reuse pattern from JIRA-12):
  - Document name, page, relevance score bar
  - Expandable passage preview

### Domain
- `RagIngestRequest` data class: documentName, documentBytes, mimeType
- `RagIngestionStatus` sealed class: Idle / InProgress(stage) / Success(chunkCount) / Failed(stage, error)
- `RagQuery` data class: question, topK, hybridSearch
- `RagAnswer` data class: answer, sources, emptyRetrieval, truncated
- `IngestDocumentUseCase`, `QueryRagUseCase`

## Acceptance Criteria

| # | Criterion |
|---|---|
| AC1 | User can pick a document via SAF (PDF / TXT / MD). |
| AC2 | Pipeline stepper shows Extract → Chunk → Embed → Store with live status per stage. |
| AC3 | Ingestion failure shows the failing stage with an error message. |
| AC4 | Success card shows the number of chunks created. |
| AC5 | User can enter a question and receive an answer. |
| AC6 | Source cards show document name, page, and relevance score bar. |
| AC7 | Hybrid search toggle changes the query mode badge. |
| AC8 | Empty retrieval shows a clear notice card. |
| AC9 | Use cases and `RagViewModel` are unit tested with fake repositories. |
| AC10 | Debug build succeeds. |

## Workflow

1. Define domain models.
2. Implement SAF launcher integration.
3. Implement `RagRepository` interface and impl with multipart upload.
4. Implement `IngestDocumentUseCase` and `QueryRagUseCase`.
5. Implement `RagViewModel` with pipeline status accumulation.
6. Implement `RagScreen` with stepper, answer card, and source cards.
7. Write unit tests.
8. Run: `./gradlew testDebugUnitTest assembleDebug`.
9. Verify every AC individually.

## Final Response Format

Provide:
- Implementation Summary
- Files Created / Modified
- Pipeline Stepper Evidence (all 4 stages shown)
- Answer + Source Card Evidence
- Tests Executed and Results
- AC1–AC10 PASS/FAIL with evidence
- Known Limitations
- Recommended Next Ticket: JIRA-15

> Never claim an AC is PASS without evidence.
