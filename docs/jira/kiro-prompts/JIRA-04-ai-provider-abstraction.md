# Prompt Engineering Studio Screen

> **Roadmap module:** 04 — Prompt Engineering
> **Epic:** Agentic AI Full Stack — Android Client
> **Backend dependency:** NAI-AI-04 (prompt engineering endpoint via /v1/chat/completions)

---

You are implementing JIRA-04 — Prompt Engineering Studio Screen for Nexus AI.

## Jira Title

Prompt Engineering Studio Screen

## Description

Deliver the Prompt Engineering studio screen where users select a prompt strategy
(zero-shot / few-shot / role-based / structured output), fill in variable chips, preview the
rendered prompt, and submit it to the FastAPI backend. Structured output responses are validated
and badged as valid/invalid JSON.

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
2. Inspect JIRA-03 chat screen — reuse the message/response display and Retrofit setup.
3. Inspect JIRA-02 design system — reuse `NexusCard`, `NexusButton`, `NexusTextField`.

## Out of Scope

- Saving prompt templates to a local database
- Sharing or exporting prompts
- Version history of prompt runs

## Architecture Rules

- Prompt template definitions are data classes in the Domain layer — not hardcoded strings
  in Composables.
- Variable substitution happens in the ViewModel via a `PromptRenderer` use case.
- Input size limit is enforced in the ViewModel before the request is sent.
- Structured output validation (JSON parse check) is in the ViewModel — not in Compose.

## Implementation Task

### Feature: Prompt Studio Screen (`feature/prompt-studio/`)
- Strategy tab row: Zero-Shot / Few-Shot / Role-Based / Structured Output
- Per-strategy form:
  - Zero-shot: single instruction field
  - Few-shot: instruction + up to 3 example input/output pairs
  - Role-based: system role field + user message field
  - Structured output: instruction + JSON schema field
- Variable chip bar: user adds `{variable}` placeholders; chip tap opens an edit dialog
- Rendered prompt preview card (read-only, monospace)
- Character count with limit warning at 90% of max
- Submit button → `POST /v1/chat/completions` with rendered prompt
- Response card:
  - Structured output tab: formatted JSON with valid/invalid badge
  - Other tabs: plain text response
- Loading and error states using design system components

### Domain
- `PromptTemplate` data class: id, strategy, version, raw template, variable names
- `PromptRenderer` use case: substitutes variables, enforces char limit, returns rendered string
- `RunPromptUseCase` calling `PromptRepository`

## Acceptance Criteria

| # | Criterion |
|---|---|
| AC1 | Strategy tab row allows switching between all four strategies. |
| AC2 | Variable chips allow adding and substituting named placeholders. |
| AC3 | Rendered prompt preview updates live as variables are filled in. |
| AC4 | Character limit is enforced; submission is blocked when exceeded. |
| AC5 | Structured output response shows a valid or invalid JSON badge. |
| AC6 | Zero-shot, few-shot, and role-based responses are displayed as plain text. |
| AC7 | Loading and error states are handled using design system components. |
| AC8 | `PromptRenderer` use case is unit tested with variable substitution and limit edge cases. |
| AC9 | `PromptStudioViewModel` is unit tested with a fake repository. |
| AC10 | Debug build succeeds. |

## Workflow

1. Define `PromptTemplate`, `PromptStrategy`, `PromptRenderer` in Domain.
2. Implement `PromptStudioViewModel` with strategy selection and variable map.
3. Implement `RunPromptUseCase` and `PromptRepository` interface + impl.
4. Implement `PromptStudioScreen` with tab row and per-strategy forms.
5. Implement variable chip bar.
6. Implement JSON validation in ViewModel.
7. Write unit tests for `PromptRenderer` and ViewModel.
8. Run: `./gradlew testDebugUnitTest assembleDebug`.
9. Verify every AC individually.

## Final Response Format

Provide:
- Implementation Summary
- Files Created / Modified
- Variable Substitution Evidence (before/after prompt preview)
- JSON Validation Evidence (valid + invalid case)
- Tests Executed and Results
- AC1–AC10 PASS/FAIL with evidence
- Known Limitations
- Recommended Next Ticket: JIRA-05 (Ollama Local Model Screen)

> Never claim an AC is PASS without evidence.
