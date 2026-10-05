# JIRA-01 — Android App Foundation & GenAI Screen

> **Roadmap module:** 01 — GenAI
> **Epic:** Agentic AI Full Stack — Android Client
> **Backend dependency:** NAI-AI-01 GenAI Foundations (FastAPI /v1/chat/completions)

---

You are implementing JIRA-01 — Android App Foundation & GenAI Screen for Nexus AI.

## Jira Title

Android App Foundation & GenAI Screen

## Description

Establish the Android application's Clean Architecture foundation (Presentation → Domain → Data),
configure Hilt DI, and deliver the first feature screen: a GenAI text-generation screen where
users enter a prompt with temperature and max_tokens controls and receive a generated response
from the FastAPI backend.

## Required Skills

- android
- clean-architecture
- testing

## Pre-Implementation Checklist

Before making any changes:

1. Read and understand:
   - This ticket and all Acceptance Criteria below
   - `.kiro/steering/` — all files
   - `.kiro/skills/android/SKILL.md`
   - `.kiro/skills/clean-architecture/SKILL.md`
   - `.kiro/skills/testing/SKILL.md`
2. Inspect existing repository structure, Gradle config, and any existing modules.
3. Confirm NAI-AI-01 FastAPI backend is running at the configured base URL.
4. Do not duplicate infrastructure that already exists.

## Architecture Rules

- Maintain `Presentation → Domain → Data` dependency direction.
- Hilt is the only DI mechanism — no manual service locators.
- All network calls go through a Retrofit service in the Data layer.
- ViewModels expose `StateFlow<UiState>` — never raw mutable state to Compose.
- Domain use cases and entities have no Android/framework imports where avoidable.
- API base URL is loaded from `local.properties` or BuildConfig — never hardcoded.

## Implementation Task

### 1. Core Foundation
- `app/` module with Application class and Hilt setup
- `core/domain/` — common `Result<T>` sealed type and base use case pattern
- `core/data/` — Retrofit + OkHttp setup with configurable base URL
- `core/ui/` — Material 3 theme scaffold (full theme implemented in JIRA-02)

### 2. GenAI Screen (`feature/genai/`)
- `GenAiScreen` Composable: prompt `TextField`, temperature slider (0.0–2.0),
  max_tokens input, Send button
- `GenAiViewModel` with `StateFlow<GenAiUiState>` (Idle / Loading / Success / Error)
- `GenerateTextUseCase` calling `GenAiRepository`
- `GenAiRepository` interface (Domain) + `GenAiRepositoryImpl` (Data) calling
  `POST /v1/chat/completions`
- Response text displayed in a scrollable card
- Loading indicator while awaiting response
- Error state with user-safe message and retry action

## Acceptance Criteria

| # | Criterion |
|---|---|
| AC1 | Clean Architecture layers (Presentation / Domain / Data) are established with correct dependency direction. |
| AC2 | Hilt DI is configured and all dependencies are injected. |
| AC3 | Common `Result<T>` / error handling types exist in `core/domain/`. |
| AC4 | GenAI screen renders with prompt input, temperature slider, and max_tokens field. |
| AC5 | User can submit a prompt and receive a generated text response from the backend. |
| AC6 | Temperature and max_tokens values are sent in the request. |
| AC7 | Loading state is shown while the backend responds. |
| AC8 | Error state shows a user-safe message and a retry action. |
| AC9 | ViewModel and `GenerateTextUseCase` are unit tested with a fake repository. |
| AC10 | Debug build succeeds. |

## Workflow

1. Set up Gradle modules: `app`, `core/domain`, `core/data`, `core/ui`, `feature/genai`.
2. Configure Hilt in `Application` class.
3. Set up Retrofit with base URL from BuildConfig.
4. Implement `Result<T>` sealed type.
5. Implement GenAI domain model, use case, repository interface.
6. Implement Data layer: Retrofit service, DTO, repository impl.
7. Implement `GenAiViewModel` and `GenAiScreen`.
8. Write unit tests for ViewModel and use case.
9. Run: `./gradlew testDebugUnitTest assembleDebug`.
10. Verify every AC individually with evidence.

## Final Response Format

Provide:
- Implementation Summary
- Files Created / Modified
- Architecture Diagram (module dependencies)
- Tests Executed and Results
- AC1–AC10 PASS/FAIL with evidence
- Known Limitations
- Recommended Next Ticket: JIRA-02

> Never claim an AC is PASS without evidence.
