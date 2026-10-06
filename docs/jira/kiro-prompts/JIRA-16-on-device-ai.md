# LangGraph Workflow Screen

> **Roadmap module:** 16 — LangGraph
> **Epic:** Agentic AI Full Stack — Android Client
> **Backend dependency:** NAI-AI-16 (LangGraph graph runtime — POST /v1/agents/graph/run)

---

You are implementing JIRA-16 — LangGraph Workflow Screen for Nexus AI.

## Jira Title

LangGraph Workflow Screen

## Description

Deliver a LangGraph workflow screen where users submit a research query and watch a stateful
graph execute node by node. The screen shows vertical workflow node cards with live status,
supports a human approval gate, displays a step counter, and shows the final summary result.
Calls the FastAPI LangGraph endpoint from NAI-AI-16.

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
2. Inspect JIRA-11 agent screen — reuse the step counter pattern and cancellation flow.
3. Inspect JIRA-02 design system — reuse status badges and `NexusCard`.
4. Human approval gate pauses execution — the screen must remain interactive while paused.

## Out of Scope

- Building or editing graph definitions from within the app
- Persisting workflow run history

## Architecture Rules

- `GraphNodeStatus` sealed class: Pending / Running / Completed / Failed / AwaitingApproval.
- `WorkflowState` data class: nodes, currentNodeName, stepCount, maxSteps, requiresApproval, summary.
- Approval and rejection are separate API calls — `POST /v1/agents/graph/{session_id}/approve`
  and `POST /v1/agents/graph/{session_id}/reject`.
- Checkpoint session ID is shown in the UI and persisted in ViewModel — allows resume.
- Execution polling or streaming is handled in ViewModel on `Dispatchers.IO`.

## Implementation Task

### Feature: LangGraph Screen (`feature/langgraph/`)
- Query `TextField` (multi-line) + Max Steps stepper
- Run Workflow button → `POST /v1/agents/graph/run`
- Workflow node column (vertical):
  - Node card per graph node (Planner / Researcher / Human Review / Summariser):
    - Node icon, node name
    - Status badge: Pending (grey) / Running (blue spinner) / Done (green ✓) / Failed (red ✗) / Awaiting (amber ⏸)
    - Vertical connector line between cards (coloured by status)
- Step counter chip: "Step 2 / 10"
- Checkpoint ID chip (shown once first step completes)
- Human Approval Gate card (shown when `requiresApproval = true`):
  - "Workflow paused for review" message
  - Approve button (green) → calls approve endpoint
  - Reject button (red) → calls reject endpoint
- Result card (shown on completion):
  - Status badge: COMPLETED / STEP_LIMIT_REACHED / REJECTED
  - Final summary text
- Cancel button (cancels the running coroutine)

### Domain
- `GraphRunRequest` data class: query, maxSteps
- `GraphNode` data class: name, status, icon
- `WorkflowResult` sealed class: Completed(summary) / StepLimitReached / Rejected / Failed(error)
- `RunGraphWorkflowUseCase`, `ApproveGraphStepUseCase`, `RejectGraphStepUseCase`

## Acceptance Criteria

| # | Criterion |
|---|---|
| AC1 | Workflow node column shows all nodes with Pending status at start. |
| AC2 | Nodes update to Running and then Done or Failed as execution progresses. |
| AC3 | Step counter chip updates with each completed step. |
| AC4 | Human Approval Gate card appears with Approve and Reject buttons. |
| AC5 | Approving resumes the workflow and nodes continue updating. |
| AC6 | Rejecting stops the workflow and shows REJECTED status. |
| AC7 | Checkpoint ID is displayed once available. |
| AC8 | Final summary result is shown when the workflow completes. |
| AC9 | Use cases and `LangGraphViewModel` are unit tested with fake repositories. |
| AC10 | Debug build succeeds. |

## Workflow

1. Define domain models.
2. Implement `GraphRepository` interface and impl.
3. Implement three use cases.
4. Implement `LangGraphViewModel` with node status polling.
5. Implement `LangGraphScreen` with node column and approval gate.
6. Write unit tests including approval flow.
7. Run: `./gradlew testDebugUnitTest assembleDebug`.
8. Verify every AC individually.

## Final Response Format

Provide:
- Implementation Summary
- Files Created / Modified
- Node Column Evidence (all status colours shown)
- Approval Gate Evidence
- Tests Executed and Results
- AC1–AC10 PASS/FAIL with evidence
- Known Limitations
- Recommended Next Ticket: JIRA-17 (MCP Server & Tools Screen)

> Never claim an AC is PASS without evidence.
