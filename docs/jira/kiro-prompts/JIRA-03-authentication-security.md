# JIRA-03 — Multi-Provider LLM Chat Screen

> **Roadmap module:** 03 — Working with LLMs
> **Epic:** Agentic AI Full Stack — Android Client
> **Backend dependency:** NAI-AI-03 (multi-provider FastAPI /v1/chat/completions)

---

You are implementing JIRA-03 — Multi-Provider LLM Chat Screen for Nexus AI.

## Jira Title

Multi-Provider LLM Chat Screen

## Description

Deliver the multi-provider chat screen that allows users to select an LLM provider (OpenAI /
Gemini), configure the active model, and have a streaming conversation backed by the FastAPI
multi-provider service. API keys are stored securely using EncryptedSharedPreferences.

## Required Skills

- android
- compose
- security
- testing

## Pre-Implementation Checklist

Before making any changes:

1. Read and understand:
   - This ticket and all Acceptance Criteria below
   - `.kiro/steering/05-security-standards.md` — secrets storage rules (mandatory)
   - `.kiro/skills/android/SKILL.md`, `.kiro/skills/compose/SKILL.md`
   - `.kiro/skills/security/SKILL.md`, `.kiro/skills/testing/SKILL.md`
2. Inspect JIRA-01 (Retrofit setup) and JIRA-02 (design system) — reuse both.
3. Never hardcode API keys — store in `EncryptedSharedPreferences`.

## Architecture Rules

- UI → ViewModel → UseCase → Repository → Retrofit (Data layer only).
- Provider selection is a domain-level concept — `LLMProvider` enum in Domain.
- Streaming uses `Flow<StreamChunk>` from repository; ViewModel accumulates tokens.
- API keys stored in `EncryptedSharedPreferences` — never in plain `SharedPreferences`,
  `DataStore`, source code, or logs.
- `ChatMessage` domain model has no Android/framework imports.

## Implementation Task

### Feature: Chat Screen (`feature/chat/`)
- `ChatScreen` Composable:
  - Provider selector chip row (OpenAI / Gemini)
  - Scrollable message list (user bubbles right-aligned, AI bubbles left)
  - Floating composer bar: `TextField` + Send button + attachment placeholder
  - Streaming text renders tokens progressively with a blinking cursor
  - Thinking indicator (animated dots) while waiting for first token
- `ChatViewModel`:
  - `StateFlow<ChatUiState>` (Idle / Loading / Streaming / Error)
  - `SharedFlow<ChatUiEvent>` for one-shot events (scroll to bottom)
  - Provider selection persisted to DataStore
- `SendMessageUseCase`, `ChatRepository` interface + impl
- Retrofit streaming via chunked transfer / SSE to `/v1/chat/completions`

### Security
- `ApiKeyRepository` using `EncryptedSharedPreferences` for OpenAI and Gemini keys
- Key entry bottom sheet: obscured text field, save action, clear action
- Keys never appear in logs

## Acceptance Criteria

| # | Criterion |
|---|---|
| AC1 | Provider selector (OpenAI / Gemini) is shown; active provider is highlighted. |
| AC2 | User can type and send a message; it appears in the message list. |
| AC3 | AI response tokens stream progressively — no full-response wait. |
| AC4 | Thinking indicator shows while waiting for the first token. |
| AC5 | Switching provider changes the active backend without restarting the screen. |
| AC6 | API keys are stored in `EncryptedSharedPreferences` — never logged or hardcoded. |
| AC7 | Loading and error states are handled with user-safe messages. |
| AC8 | The UI does not call provider SDKs or Retrofit directly. |
| AC9 | `ChatViewModel` and `SendMessageUseCase` are unit tested with a fake repository. |
| AC10 | Debug build succeeds. |

## Workflow

1. Implement `ApiKeyRepository` with `EncryptedSharedPreferences`.
2. Define `ChatMessage`, `LLMProvider`, `ChatUiState` domain/UI models.
3. Implement `SendMessageUseCase` and `ChatRepository` interface + impl.
4. Implement streaming Retrofit call returning `Flow<StreamChunk>`.
5. Implement `ChatViewModel` with token accumulation.
6. Implement `ChatScreen` with provider selector and message list.
7. Add API key bottom sheet.
8. Write unit tests for ViewModel and use case.
9. Run: `./gradlew testDebugUnitTest assembleDebug`.
10. Verify every AC individually.

## Final Response Format

Provide:
- Implementation Summary
- Files Created / Modified
- Security Implementation Evidence (EncryptedSharedPreferences in use)
- Streaming Test Evidence
- Tests Executed and Results
- AC1–AC10 PASS/FAIL with evidence
- Known Limitations
- Recommended Next Ticket: JIRA-04

> Never claim an AC is PASS without evidence.
