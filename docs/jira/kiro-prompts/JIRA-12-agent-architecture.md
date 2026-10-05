# JIRA-12 — LangChain Integration Screen

> **Roadmap module:** 12 — LangChain
> **Epic:** Agentic AI Full Stack — Android Client
> **Backend dependency:** NAI-AI-12 (LangChain RAG chain + tool endpoints — POST /v1/rag/query)

---

You are implementing JIRA-12 — LangChain Integration Screen for Nexus AI.

## Jira Title

LangChain Integration Screen

## Description

Deliver a LangChain demo screen showing RAG chain execution with retrieved source references,
document Q&A, and tool invocation results from the FastAPI LangChain endpoints. Users can select
the chain type, enter a question, and see the answer alongside cited sources.

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
3. Inspect JIRA-02 design system — reuse `NexusCard`, relevance score chips.
4. Source reference cards must always show at minimum: document name, page, and relevance score.

## Architecture Rules

- `LangChainQuery` domain model: question, chainType, topK.
- `LangChainResponse` domain model: answer, sources (list of `SourceReference`).
- `SourceReference` domain model: documentId, source, page, relevanceScore.
- Empty retrieval (no sources returned) is an explicit `empty_retrieval` flag — not an empty list.
- Chain type is a domain enum: RAG / DOCUMENT_QA / TOOL_BASED.

## Implementation Task

### Feature: LangChain Screen (`feature/langchain/`)
- Chain type selector row: RAG / Document QA / Tool-Based chips
- Question `TextField` (multi-line) with Submit button
- Streaming response text (tokens appear progressively where chain supports streaming)
- Answer card:
  - Full answer text
  - Empty retrieval notice if `empty_retrieval = true`
- Source reference cards (shown below the answer):
  - Document name, page number
  - Relevance score bar (colour-coded: green ≥ 0.8, amber ≥ 0.5, red < 0.5)
  - Expandable passage preview (first 200 chars of the retrieved chunk)
- Tool result card (shown when chain type = TOOL_BASED):
  - Tool name chip
  - Tool result text
- Empty state: "No relevant sources found" card

### Domain
- `LangChainQuery` data class: question, chainType, topK (default 5)
- `SourceReference` data class: documentId, source, page, relevanceScore, passagePreview
- `LangChainResponse` data class: answer, sources, emptyRetrieval
- `RunLangChainQueryUseCase` calling `LangChainRepository`

## Acceptance Criteria

| # | Criterion |
|---|---|
| AC1 | Chain type selector (RAG / Document QA / Tool-Based) is shown. |
| AC2 | User can submit a question and receive an answer. |
| AC3 | Streaming tokens appear progressively where the chain supports it. |
| AC4 | Source reference cards show document name, page, and relevance score bar. |
| AC5 | Relevance score bars are colour-coded (green / amber / red). |
| AC6 | Source passage preview is expandable. |
| AC7 | Empty retrieval shows a "No relevant sources found" card. |
| AC8 | Tool result card appears when chain type is Tool-Based. |
| AC9 | `RunLangChainQueryUseCase` and `LangChainViewModel` are unit tested. |
| AC10 | Debug build succeeds. |

## Workflow

1. Define `LangChainQuery`, `SourceReference`, `LangChainResponse` domain models.
2. Implement `LangChainRepository` interface and impl.
3. Implement `RunLangChainQueryUseCase`.
4. Implement `LangChainViewModel`.
5. Implement `LangChainScreen` with chain selector, answer card, and source cards.
6. Write unit tests.
7. Run: `./gradlew testDebugUnitTest assembleDebug`.
8. Verify every AC individually.

## Final Response Format

Provide:
- Implementation Summary
- Files Created / Modified
- Source Reference Card Evidence (score + passage)
- Empty Retrieval Evidence
- Tests Executed and Results
- AC1–AC10 PASS/FAIL with evidence
- Known Limitations
- Recommended Next Ticket: JIRA-13

> Never claim an AC is PASS without evidence.
