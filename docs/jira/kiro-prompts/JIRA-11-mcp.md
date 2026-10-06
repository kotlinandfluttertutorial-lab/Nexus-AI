# Agentic AI Screen

> **Roadmap module:** 11 — Agentic AI
> **Epic:** Agentic AI Full Stack — Android Client
> **Backend dependency:** NAI-AI-11 (FastAPI agent runtime — POST /v1/agents/run)

---

You are implementing JIRA-11 — Agentic AI Screen for Nexus AI.

## Jira Title

Agentic AI Screen

## Description

Deliver the Agentic AI screen where users submit a task, watch the agent execution loop plan
and select tools step by step, and see the final result. The screen calls the FastAPI
`/v1/agents/run` endpoint from NAI-AI-11 and displays a live execution trace.

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
3. Inspect JIRA-02 design system — reuse `NexusCard`, status badges, loading components.
4. Agent execution is long-running — use a coroutine with cancellation support.

## Out of Scope

- Creating or registering custom tools from within the app
- Persisting agent run history
- Agent configuration beyond max steps

## Architecture Rules

- Agent execution runs in a `viewModelScope` coroutine that can be cancelled.
- `AgentRunRequest` and `AgentRunResponse` are domain models — no Retrofit DTOs in ViewModels.
- Execution trace is accumulated in a `List<TraceStepUiModel>` held in the ViewModel StateFlow.
- Cancellation calls a cancel endpoint or simply cancels the coroutine and shows CANCELLED state.
- Never expose raw exception stack traces in the UI.

## Implementation Task

### Feature: Agent Screen (`feature/agent/`)
- Task input `TextField` (multi-line)
- Max steps stepper (1–20, default 10)
- Run Agent button → `POST /v1/agents/run`
- Execution trace live display:
  - Step counter chip: "Step 3 / 10"
  - Animated "Thinking…" indicator between steps
  - Trace step cards (appear one by one as the response streams):
    - Thought text
    - Tool name chip (if tool called)
    - Tool input summary (collapsed by default)
    - Tool output summary (collapsed by default)
- Result card (shown on completion):
  - Status badge: SUCCESS / STEP_LIMIT_REACHED / CANCELLED / FAILURE
  - Final output text
  - Total steps taken
- Cancel button (visible while running) — cancels the coroutine job

### Domain
- `AgentRunRequest` data class: task, maxSteps
- `TraceStep` data class: stepNumber, thought, toolName, toolInput, toolOutput, error
- `AgentRunResult` sealed class: Success(output, steps, trace) / StepLimitReached / Cancelled / Failure(error)
- `RunAgentUseCase` calling `AgentRepository`

## Acceptance Criteria

| # | Criterion |
|---|---|
| AC1 | Task input and max-steps stepper are shown. |
| AC2 | Run Agent calls the backend and shows a step counter. |
| AC3 | Step cards appear progressively as execution proceeds. |
| AC4 | Each step card shows thought, tool name, and collapsible tool input/output. |
| AC5 | Cancel button cancels the running coroutine and shows CANCELLED status. |
| AC6 | STEP_LIMIT_REACHED status is clearly displayed when the limit is hit. |
| AC7 | SUCCESS result shows the final output and total steps taken. |
| AC8 | FAILURE state shows a user-safe error message. |
| AC9 | `RunAgentUseCase` and `AgentViewModel` are unit tested with fake repositories. |
| AC10 | Debug build succeeds. |

## Workflow

1. Define `AgentRunRequest`, `TraceStep`, `AgentRunResult` domain models.
2. Implement `AgentRepository` interface and impl calling `/v1/agents/run`.
3. Implement `RunAgentUseCase`.
4. Implement `AgentViewModel` with trace accumulation and cancellation.
5. Implement `AgentScreen` with step cards and cancel button.
6. Write unit tests.
7. Run: `./gradlew testDebugUnitTest assembleDebug`.
8. Verify every AC individually.

## Final Response Format

Provide:
- Implementation Summary
- Files Created / Modified
- Execution Trace Evidence (step cards shown)
- Cancellation Test Evidence
- Tests Executed and Results
- AC1–AC10 PASS/FAIL with evidence
- Known Limitations
- Recommended Next Ticket: JIRA-12 (LangChain Integration Screen)

> Never claim an AC is PASS without evidence.
