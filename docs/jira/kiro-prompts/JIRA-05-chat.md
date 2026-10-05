# JIRA-05 — Ollama Local Model Screen

> **Roadmap module:** 05 — Self-Hosted LLMs with Ollama
> **Epic:** Agentic AI Full Stack — Android Client
> **Backend dependency:** NAI-AI-05 (Ollama adapter in FastAPI; Ollama running locally)

---

You are implementing JIRA-05 — Ollama Local Model Screen for Nexus AI.

## Jira Title

Ollama Local Model Screen

## Description

Deliver an Ollama connection screen where users configure the local Ollama server URL, select
a downloaded model, chat with streaming responses, view per-response latency, and toggle
OpenAI-compatible mode. All calls are routed through the FastAPI backend's OllamaProvider.

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
2. Inspect JIRA-03 chat screen — reuse the message list and streaming display.
3. Inspect JIRA-02 design system — reuse `NexusCard`, `NexusTextField`.
4. Ollama server URL is stored in DataStore — never hardcoded.

## Architecture Rules

- Ollama server URL stored in `DataStore<Preferences>` via `OllamaSettingsRepository`.
- Model list fetched from the FastAPI backend (which queries `/api/tags` on the Ollama instance).
- Active provider selection (Ollama vs cloud) is a domain-level config, not a UI flag.
- Latency measured in ViewModel from request start to first-token received.

## Implementation Task

### Feature: Ollama Screen (`feature/ollama/`)
- Server URL `TextField` with a Test Connection button
- Connection status indicator: Connected (green) / Disconnected (red) / Checking (amber)
- Model dropdown populated from `GET /v1/models` (filtered to Ollama models)
- Chat interface (reuse `ChatScreen` components from JIRA-03):
  - Messages list, composer, streaming token display
  - Latency badge on each AI response card (e.g. "423 ms")
- OpenAI-compatible mode toggle:
  - When ON: sends requests to `/v1/chat/completions` in OpenAI format via Ollama's `/v1/` endpoint
  - Badge on the screen showing active mode
- `OllamaSettingsViewModel` for URL + mode persistence
- `OllamaChatViewModel` extending the base chat pattern

### Domain
- `OllamaSettings` data class: serverUrl, activeModel, openAiCompatMode
- `TestOllamaConnectionUseCase` → `GET /health` via FastAPI
- `GetOllamaModelsUseCase` → `GET /v1/models`

## Acceptance Criteria

| # | Criterion |
|---|---|
| AC1 | Ollama server URL is configurable, validated, and persisted in DataStore. |
| AC2 | Test Connection button shows Connected / Disconnected status. |
| AC3 | Model list is populated from the backend and a model can be selected. |
| AC4 | User can send a chat message and receive a streaming response via Ollama. |
| AC5 | Latency (ms from send to first token) is shown on each AI response card. |
| AC6 | OpenAI-compatible mode toggle changes the request format badge. |
| AC7 | Connection failure shows a clear error with a retry action. |
| AC8 | Loading and error states are handled. |
| AC9 | `OllamaSettingsViewModel` and use cases are unit tested with fake repositories. |
| AC10 | Debug build succeeds. |

## Workflow

1. Implement `OllamaSettings` domain model and `OllamaSettingsRepository`.
2. Persist settings in DataStore.
3. Implement `TestOllamaConnectionUseCase` and `GetOllamaModelsUseCase`.
4. Implement `OllamaScreen` reusing chat components from JIRA-03.
5. Add latency measurement in ViewModel.
6. Add OpenAI-compat mode toggle.
7. Write unit tests.
8. Run: `./gradlew testDebugUnitTest assembleDebug`.
9. Verify every AC individually.

## Final Response Format

Provide:
- Implementation Summary
- Files Created / Modified
- Connection Status Evidence
- Latency Display Evidence
- OpenAI-Compat Mode Evidence
- Tests Executed and Results
- AC1–AC10 PASS/FAIL with evidence
- Known Limitations
- Recommended Next Ticket: JIRA-06

> Never claim an AC is PASS without evidence.
