# Flask LLM Service Integration Screen

> **Roadmap module:** 06 — LLM as a Service with Flask
> **Epic:** Agentic AI Full Stack — Android Client
> **Backend dependency:** NAI-AI-06 (Flask LLM service)

---

You are implementing JIRA-06 — Flask LLM Service Integration Screen for Nexus AI.

## Jira Title

Flask LLM Service Integration Screen

## Description

Deliver a Flask service connection screen where users configure the Flask backend URL, check
health, list available models, send chat requests, and inspect the raw JSON request/response.
This screen demonstrates consuming a standalone Flask LLM API from Android.

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
   - `.kiro/skills/testing/SKILL.md`
2. Inspect JIRA-03 chat screen and JIRA-05 Ollama screen — reuse URL config and
   connection status patterns.
3. Inspect JIRA-02 design system — reuse `NexusCard`, `NexusTextField`.
4. Flask base URL stored in DataStore — never hardcoded.

## Out of Scope

- Streaming responses (Flask service is non-streaming in this module)
- Saving or exporting chat history

## Architecture Rules

- Flask service is a separate Retrofit client with its own base URL (not shared with FastAPI).
- `FlaskServiceRepository` (Domain interface) + `FlaskServiceRepositoryImpl` (Data).
- JSON inspector card renders raw request and response strings — no business logic in Compose.
- Authentication error (401) maps to a domain `AuthError` type, not a raw exception.

## Implementation Task

### Feature: Flask Service Screen (`feature/flask-service/`)
- Service URL `TextField` with Test Connection button (calls `GET /health`)
- Connection status indicator with last-checked timestamp
- Models card: fetches `GET /v1/models` and lists model names as chips
- Chat card:
  - Simple single-turn prompt field + Submit button
  - Response displayed below
  - Response time badge (ms)
- JSON Inspector card (expandable):
  - Request JSON (syntax-highlighted with monospace font)
  - Response JSON (syntax-highlighted)
- Authentication error card: shown when API key is missing/invalid with a Configure Key action

### Domain
- `FlaskServiceConfig` data class: baseUrl, apiKey
- `CheckFlaskHealthUseCase`
- `GetFlaskModelsUseCase`
- `SendFlaskChatUseCase`

## Acceptance Criteria

| # | Criterion |
|---|---|
| AC1 | Flask base URL is configurable and persisted in DataStore. |
| AC2 | Test Connection calls `GET /health` and shows Connected or Disconnected. |
| AC3 | Available models are fetched from `GET /v1/models` and listed as chips. |
| AC4 | User can send a chat request and see the response. |
| AC5 | Response time in milliseconds is shown on the response card. |
| AC6 | Raw request and response JSON are shown in an expandable inspector card. |
| AC7 | Authentication error (401) shows a clear message with a Configure Key action. |
| AC8 | Loading and error states are handled. |
| AC9 | Use cases and `FlaskServiceViewModel` are unit tested with a fake repository. |
| AC10 | Debug build succeeds. |

## Workflow

1. Create `FlaskServiceConfig` and repository interface in Domain.
2. Implement separate Retrofit client for Flask base URL.
3. Implement `FlaskServiceRepositoryImpl` with health, models, and chat calls.
4. Implement `FlaskServiceViewModel` and use cases.
5. Implement `FlaskServiceScreen` with all cards.
6. Implement JSON inspector with monospace rendering.
7. Write unit tests.
8. Run: `./gradlew testDebugUnitTest assembleDebug`.
9. Verify every AC individually.

## Final Response Format

Provide:
- Implementation Summary
- Files Created / Modified
- JSON Inspector Screenshot Evidence
- Auth Error Evidence
- Tests Executed and Results
- AC1–AC10 PASS/FAIL with evidence
- Known Limitations
- Recommended Next Ticket: JIRA-07 (API Test Runner Screen)

> Never claim an AC is PASS without evidence.
