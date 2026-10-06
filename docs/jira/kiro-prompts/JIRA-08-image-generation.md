# FastAPI Service Dashboard Screen

> **Roadmap module:** 08 — Building LLM Services with FastAPI
> **Epic:** Agentic AI Full Stack — Android Client
> **Backend dependency:** NAI-AI-08 (primary FastAPI backend)

---

You are implementing JIRA-08 — FastAPI Service Dashboard Screen for Nexus AI.

## Jira Title

FastAPI Service Dashboard Screen

## Description

Deliver the FastAPI service dashboard screen — the primary connection hub for all subsequent
Android features. Users configure the FastAPI base URL, monitor health and readiness, list
available models, send chat and embedding requests, and view response times.

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
2. Inspect JIRA-01 Retrofit setup — this is the primary Retrofit client used by most features.
3. Inspect JIRA-02 design system — reuse all components.
4. FastAPI base URL stored in DataStore; used by JIRA-09–20 as the primary backend.

## Out of Scope

- Streaming on the dashboard screen (demonstrated in JIRA-03 Multi-Provider Chat)
- Per-model capability detail (covered in JIRA-09 HF Browser)

## Architecture Rules

- FastAPI base URL is the single shared `HttpUrl` in the primary Retrofit module.
- Health and readiness polling use `repeatOnLifecycle(STARTED)` — not `launchWhenStarted`.
- Response time measured from request dispatch to response receipt in ViewModel.
- Embedding vector dimension is computed from the response list size — not hardcoded.

## Implementation Task

### Feature: FastAPI Dashboard (`feature/fastapi-dashboard/`)
- Base URL `TextField` with Save button — updates the shared Retrofit client
- Endpoint status cards (auto-refresh every 30 s):
  - `/health` — liveness: green dot / red dot
  - `/readiness` — readiness: green / amber / red based on provider health
- Models card: `GET /v1/models` → horizontal chip list
- Chat card:
  - Model selector dropdown, prompt field, Submit button
  - Response card with content and response time badge
- Embeddings card:
  - Input text field, Embed button
  - Result card showing vector dimension and first 5 values
- Auth config: API key input field (masked), saved to `EncryptedSharedPreferences`

### Shared infrastructure
- Update `core/data/` primary Retrofit client to use URL from DataStore
- `FastApiStatusRepository` for health + readiness polling
- `FastApiChatRepository`, `FastApiEmbeddingsRepository`

## Acceptance Criteria

| # | Criterion |
|---|---|
| AC1 | FastAPI base URL is configurable, persisted in DataStore, and used by the primary Retrofit client. |
| AC2 | `/health` status card auto-refreshes and shows a live green or red indicator. |
| AC3 | `/readiness` status card shows provider health detail. |
| AC4 | Available models are listed from `GET /v1/models`. |
| AC5 | User can send a chat request and see the response with response time. |
| AC6 | User can send an embedding request and see the returned vector dimension. |
| AC7 | API key is stored in `EncryptedSharedPreferences` and sent as `X-API-Key`. |
| AC8 | All error states show clear messages with retry actions. |
| AC9 | Use cases and `FastApiDashboardViewModel` are unit tested with fake repositories. |
| AC10 | Debug build succeeds. |

## Workflow

1. Update primary Retrofit client to source base URL from DataStore.
2. Implement `FastApiStatusRepository` with health + readiness polling.
3. Implement `FastApiChatRepository` and `FastApiEmbeddingsRepository`.
4. Implement `FastApiDashboardViewModel`.
5. Implement `FastApiDashboardScreen` with all cards.
6. Write unit tests.
7. Run: `./gradlew testDebugUnitTest assembleDebug`.
8. Verify every AC individually.

## Final Response Format

Provide:
- Implementation Summary
- Files Created / Modified
- Health/Readiness Card Evidence
- Embeddings Response Evidence (dimension shown)
- Tests Executed and Results
- AC1–AC10 PASS/FAIL with evidence
- Known Limitations
- Recommended Next Ticket: JIRA-09 (Hugging Face Model Browser Screen)

> Never claim an AC is PASS without evidence.
