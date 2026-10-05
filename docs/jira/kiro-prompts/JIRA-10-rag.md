# JIRA-10 — Advanced LLM Features Screen

> **Roadmap module:** 10 — Advanced LLM Features
> **Epic:** Agentic AI Full Stack — Android Client
> **Backend dependency:** NAI-AI-10 (FastAPI advanced features: structured output, streaming, tool calling, vision)

---

You are implementing JIRA-10 — Advanced LLM Features Screen for Nexus AI.

## Jira Title

Advanced LLM Features Screen

## Description

Deliver an advanced features screen with four tabs demonstrating structured JSON output,
end-to-end streaming, function/tool calling round-trip, and vision input on Android.
Each tab calls the corresponding FastAPI advanced endpoint from NAI-AI-10.

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
2. Inspect JIRA-08 FastAPI dashboard — reuse the primary Retrofit client and base URL.
3. Inspect JIRA-03 chat screen — reuse streaming display components.
4. Inspect JIRA-02 design system — reuse all components.

## Architecture Rules

- Each tab has its own ViewModel — not one shared ViewModel for all four tabs.
- Structured output validation (JSON schema check) happens in the ViewModel — not Compose.
- Vision input: image is read from the gallery, converted to base64 in a coroutine on
  `Dispatchers.IO` — never on the main thread.
- Feature-not-supported error (`feature_not_supported` code) maps to a domain `FeatureError`
  type and shows an info card — never a crash.

## Implementation Task

### Feature: Advanced LLM Features (`feature/advanced-llm/`)
Implement a `TabRow` with four tabs:

#### Tab 1 — Structured Output
- JSON schema input field (default: `{"type":"object","properties":{"answer":{"type":"string"}}}`)
- Prompt field
- Submit button → `POST /v1/chat/completions` with `response_format: {type: "json_object"}`
- Response card: formatted JSON, valid/invalid badge, raw content toggle

#### Tab 2 — Streaming Demo
- Prompt field
- Stream button → SSE streaming request
- Live token-by-token display (each token appended to text)
- Token counter (total tokens received)
- Time-to-first-token badge (ms)

#### Tab 3 — Tool Calling
- Tool name and description display card (pre-configured demo tool)
- Task prompt field
- Run button → `POST /v1/chat/completions` with tools array
- Trace cards:
  - Model's tool call request card (tool name + arguments JSON)
  - Tool result card (simulated result)
  - Final model response card

#### Tab 4 — Vision
- Image picker button (opens gallery, READ_MEDIA_IMAGES permission)
- Selected image thumbnail
- Prompt field (default: "What is in this image?")
- Submit button → encodes image as base64, sends multimodal request
- Response card with model's description

## Acceptance Criteria

| # | Criterion |
|---|---|
| AC1 | Structured output tab sends a JSON schema and shows a valid/invalid badge on the response. |
| AC2 | Streaming tab shows tokens appearing progressively with a token counter. |
| AC3 | Time-to-first-token is measured and displayed on the streaming tab. |
| AC4 | Tool calling tab shows the model's tool call request and the final response as separate cards. |
| AC5 | Vision tab opens the gallery, shows the selected thumbnail, and displays the model's description. |
| AC6 | Feature-not-supported errors show an info card — not a crash or generic error. |
| AC7 | Each tab handles loading and error states independently. |
| AC8 | Image base64 encoding runs on `Dispatchers.IO` — not the main thread. |
| AC9 | Each tab's ViewModel and use case are unit tested with fake repositories. |
| AC10 | Debug build succeeds. |

## Workflow

1. Define domain models: `StructuredOutputRequest`, `ToolCall`, `VisionRequest`.
2. Implement four separate ViewModels and use cases.
3. Implement `AdvancedLlmScreen` with `TabRow` and four tab composables.
4. Implement gallery picker with READ_MEDIA_IMAGES permission handling.
5. Implement base64 encoding on `Dispatchers.IO`.
6. Write unit tests for all four ViewModels.
7. Run: `./gradlew testDebugUnitTest assembleDebug`.
8. Verify every AC individually.

## Final Response Format

Provide:
- Implementation Summary
- Files Created / Modified
- Structured Output Tab Evidence (valid + invalid case)
- Tool Calling Trace Evidence
- Vision Response Evidence
- Tests Executed and Results
- AC1–AC10 PASS/FAIL with evidence
- Known Limitations
- Recommended Next Ticket: JIRA-11

> Never claim an AC is PASS without evidence.
