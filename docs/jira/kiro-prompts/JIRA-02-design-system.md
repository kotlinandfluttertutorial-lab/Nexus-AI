# Transformer & Embeddings Explorer Screen

> **Roadmap module:** 02 — Transformer Architecture & Embeddings
> **Epic:** Agentic AI Full Stack — Android Client
> **Backend dependency:** NAI-AI-02 (embeddings endpoint at /v1/embeddings)

---

You are implementing JIRA-02 — Transformer & Embeddings Explorer Screen for Nexus AI.

## Jira Title

Transformer & Embeddings Explorer Screen

## Description

Establish the Material 3 design system used by all subsequent screens, then deliver the
Transformer & Embeddings explorer screen. Users can tokenise text, request embeddings from the
FastAPI backend, compare two sentences by cosine similarity, and see embedding dimensions.

## Required Skills

- android
- compose
- testing

## Pre-Implementation Checklist

Before making any changes:

1. Read and understand:
   - This ticket and all Acceptance Criteria below
   - `.kiro/steering/` — all files
   - `.kiro/skills/android/SKILL.md`
   - `.kiro/skills/compose/SKILL.md` and `.kiro/skills/testing/SKILL.md`
2. Inspect JIRA-01 foundation — reuse the Retrofit setup, `Result<T>`, and Hilt modules.
3. Read `02-android-development.md` for Compose rules (stateless composables, state hoisting).

## Architecture Rules

- All UI in Jetpack Compose — no XML layouts.
- Material 3 only — no Material 2 imports.
- Design tokens (colors, typography, shapes, spacing) centralised in `core/ui/theme/`.
- Composables are stateless — all state lives in ViewModels.
- Use `collectAsStateWithLifecycle()` for Flow collection in Compose.
- No business logic inside composables.

## Out of Scope

- Authentication and API key management
- Full transformer attention mechanism visualisation (backend concern)
- Any screen other than the Embeddings Explorer

## Implementation Task

### 1. Design System (`core/ui/`)
- `NexusAiTheme` with `darkColorScheme` / `lightColorScheme` using the Figma token set:
  - Dark bg `#090D18`, card `#151F32`, primary blue `#5B8CFF`, violet `#9575FF`
  - Light bg `#F5F7FC`, card `#FFFFFF`, primary `#3867E8`
- `NexusTypography` — Inter/Roboto, Medium headings, monospace for technical content
- `NexusShapes` — large 24 dp, medium 18 dp, small 12 dp, button 16 dp
- `NexusSpacing` — 4 / 8 / 12 / 16 / 24 / 32 dp scale
- Reusable components: `NexusCard`, `NexusButton`, `NexusTextField`, `LoadingIndicator`,
  `ErrorCard` with retry action

### 2. Embeddings Explorer Screen (`feature/embeddings/`)
- Two `TextField` inputs: Sentence A and Sentence B
- "Get Embeddings" button → `POST /v1/embeddings` for both sentences
- Display embedding dimension returned
- Cosine similarity computed on-device from the two vectors and shown as a score (0.00–1.00)
- Similarity score colour-coded: green ≥ 0.8, amber ≥ 0.5, red < 0.5
- Token count label (character-based approximation on device)

## Acceptance Criteria

| # | Criterion |
|---|---|
| AC1 | `NexusAiTheme` with dark and light colour schemes is applied globally. |
| AC2 | Typography, spacing, and shape tokens are centralised in `core/ui/`. |
| AC3 | `NexusCard`, `NexusButton`, `NexusTextField`, `LoadingIndicator`, and `ErrorCard` components exist. |
| AC4 | All components meet accessibility guidelines (content descriptions, minimum 48 dp touch targets). |
| AC5 | Embeddings screen renders two sentence inputs and a Get Embeddings button. |
| AC6 | Embeddings request is sent to `/v1/embeddings` and the returned vector dimension is displayed. |
| AC7 | Cosine similarity is computed on-device and shown with colour coding. |
| AC8 | Loading and error states are handled using the design system components. |
| AC9 | Compose tests cover `NexusCard`, `NexusButton`, and the Embeddings screen states. |
| AC10 | Debug build succeeds. |

## Workflow

1. Create `core/ui/` module with theme, tokens, and reusable components.
2. Apply `NexusAiTheme` in `MainActivity`.
3. Create `feature/embeddings/` module.
4. Implement `EmbeddingsViewModel`, use case, repository interface and impl.
5. Implement `EmbeddingsScreen` using design system components.
6. Add cosine similarity calculation in the ViewModel.
7. Write Compose tests for design system components and screen states.
8. Run: `./gradlew testDebugUnitTest assembleDebug`.
9. Verify every AC individually.

## Final Response Format

Provide:
- Implementation Summary
- Files Created / Modified
- Design Token Reference (key color/type/shape values)
- Tests Executed and Results
- AC1–AC10 PASS/FAIL with evidence
- Known Limitations
- Recommended Next Ticket: JIRA-03 (Multi-Provider LLM Chat Screen)

> Never claim an AC is PASS without evidence.
