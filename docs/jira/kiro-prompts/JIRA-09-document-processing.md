# Hugging Face Model Browser Screen

> **Roadmap module:** 09 — Hugging Face & Open-Source LLMs
> **Epic:** Agentic AI Full Stack — Android Client
> **Backend dependency:** NAI-AI-09 (HuggingFace model serving + dataset demo endpoint)

---

You are implementing JIRA-09 — Hugging Face Model Browser Screen for Nexus AI.

## Jira Title

Hugging Face Model Browser Screen

## Description

Deliver a Hugging Face model browser screen where users can view available open-source models
with their licenses and hardware requirements, run a text generation request via the HF backend,
and explore a sample HF dataset. This is the Android client for the Hugging Face module.

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
2. Inspect JIRA-08 FastAPI dashboard — this screen uses the same primary Retrofit client.
3. Inspect JIRA-02 design system — reuse `NexusCard`, `NexusChip`, and loading components.
4. Model weights are never downloaded to the device — all inference happens on the backend.

## Out of Scope

- Downloading model weights to the device
- Fine-tuning or training models
- Browsing Hugging Face Spaces from within the app

## Architecture Rules

- Model metadata is fetched from the FastAPI backend — not hardcoded in Android.
- `HFModelInfo` domain model: id, name, license, minRamGb, minVramGb, task.
- Inference request uses the same `/v1/chat/completions` endpoint with the selected HF model.
- Dataset sample viewer calls a dedicated `/v1/huggingface/dataset-sample` endpoint.
- Model weight disclaimer is always shown — model files are never stored on-device.

## Implementation Task

### Feature: HF Model Browser (`feature/hf-browser/`)
- Model list screen:
  - Search field to filter models by name
  - Model cards: name, license chip, task chip, min RAM badge, min VRAM badge
  - Select model → opens model detail bottom sheet
- Model detail bottom sheet:
  - Full name, license, task description
  - RAM / VRAM requirement chips
  - Model weight disclaimer text
  - "Run Inference" button → navigates to inference screen
- Inference screen:
  - Prompt field, Submit button
  - Response card with content and generation latency badge
  - Back to browser action
- Dataset explorer tab:
  - Calls `GET /v1/huggingface/dataset-sample`
  - Shows dataset name, split, feature list
  - One sample row displayed in a key-value card

## Acceptance Criteria

| # | Criterion |
|---|---|
| AC1 | Model list is fetched from the backend and displayed as cards. |
| AC2 | Each card shows name, license, task, and minimum RAM/VRAM. |
| AC3 | Model detail bottom sheet opens on card tap and shows full metadata. |
| AC4 | A model weight disclaimer is always visible on the detail sheet. |
| AC5 | User can run an inference request and see the response with latency. |
| AC6 | Dataset explorer shows dataset name, feature names, and one sample row. |
| AC7 | Search field filters the model list by name. |
| AC8 | Loading and error states are handled. |
| AC9 | Use cases and `HFBrowserViewModel` are unit tested with fake repositories. |
| AC10 | Debug build succeeds. |

## Workflow

1. Define `HFModelInfo` and `HFDatasetSample` domain models.
2. Implement `GetHFModelsUseCase`, `RunHFInferenceUseCase`, `GetHFDatasetSampleUseCase`.
3. Implement `HFBrowserViewModel`.
4. Implement `HFBrowserScreen` with model list and search.
5. Implement model detail bottom sheet.
6. Implement inference screen.
7. Implement dataset explorer tab.
8. Write unit tests.
9. Run: `./gradlew testDebugUnitTest assembleDebug`.
10. Verify every AC individually.

## Final Response Format

Provide:
- Implementation Summary
- Files Created / Modified
- Model Card Evidence (license + RAM shown)
- Dataset Sample Evidence
- Tests Executed and Results
- AC1–AC10 PASS/FAIL with evidence
- Known Limitations
- Recommended Next Ticket: JIRA-10 (Advanced LLM Features Screen)

> Never claim an AC is PASS without evidence.
