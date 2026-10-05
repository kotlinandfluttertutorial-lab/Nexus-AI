# JIRA-18 — Multi-Agent System Screen

> **Roadmap module:** 18 — Multi-Agent Systems
> **Epic:** Agentic AI Full Stack — Android Client
> **Backend dependency:** NAI-AI-18 (multi-agent runtime — POST /v1/agents/multi/run)

---

You are implementing JIRA-18 — Multi-Agent System Screen for Nexus AI.

## Jira Title

Multi-Agent System Screen

## Description

Deliver a multi-agent system screen where users submit a research task and watch the supervisor
delegate to specialist agents. The screen shows supervisor and specialist agent cards with live
status, a delegation trace list, round counter, per-agent step counts, and a final aggregated
result. Calls the FastAPI multi-agent endpoint from NAI-AI-18.

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
2. Inspect JIRA-11 agent screen — reuse the trace step pattern and cancellation flow.
3. Inspect JIRA-02 design system — reuse status badges and `NexusCard`.
4. Agent avatars are simple initials/icon cards — no image loading required.

## Architecture Rules

- `AgentCard` domain model: name, role (SUPERVISOR / SPECIALIST), status, stepCount, maxSteps.
- `DelegationTraceEntry` domain model: round, supervisorDecision, delegatedToAgent, agentResult.
- `MultiAgentResult` sealed class: Success(output, delegationTrace) / StepLimitReached / Cancelled / Failed.
- The UI only shows the communication summary — internal reasoning traces are never displayed.
- Each agent card updates its status independently via the accumulated state.

## Implementation Task

### Feature: Multi-Agent Screen (`feature/multi-agent/`)
- Task input `TextField` + Max Rounds stepper (1–10, default 5)
- Run Multi-Agent button → `POST /v1/agents/multi/run`
- Agent network section:
  - Supervisor agent card (distinct visual — larger, with "Coordinator" label)
  - Specialist agent cards in a row: Researcher, Writer (or as configured by backend)
  - Each card: agent name, role chip, status badge, step count / max steps
- Round counter chip: "Round 2 / 5"
- Delegation trace list (appears below agent network):
  - Trace entry rows: round number → supervisor decision → agent name → agent status badge
- Aggregated result card (shown on completion):
  - Status badge: SUCCESS / STEP_LIMIT_REACHED / CANCELLED
  - Final output text
  - Total rounds taken
- Communication summary card (collapsible): shared messages exchanged between agents
- Cancel button (cancels the running coroutine)

### Domain
- `MultiAgentRunRequest` data class: task, maxRounds
- `AgentCard` data class: name, role, status, stepCount, maxSteps
- `DelegationTraceEntry` data class: round, decision, delegatedTo, result
- `MultiAgentResult` sealed class
- `RunMultiAgentUseCase` calling `MultiAgentRepository`

## Acceptance Criteria

| # | Criterion |
|---|---|
| AC1 | Task input and max rounds stepper are shown. |
| AC2 | Supervisor and specialist agent cards are displayed with distinct visual styling. |
| AC3 | Agent status badges update live during execution. |
| AC4 | Round counter chip updates with each completed round. |
| AC5 | Delegation trace entries appear as rounds complete. |
| AC6 | Final aggregated result is shown with status badge and output text. |
| AC7 | Communication summary card is collapsible. |
| AC8 | Cancel button cancels execution and shows CANCELLED status. |
| AC9 | `RunMultiAgentUseCase` and `MultiAgentViewModel` are unit tested with fake repositories. |
| AC10 | Debug build succeeds. |

## Workflow

1. Define domain models.
2. Implement `MultiAgentRepository` interface and impl.
3. Implement `RunMultiAgentUseCase`.
4. Implement `MultiAgentViewModel` with agent card and trace accumulation.
5. Implement `MultiAgentScreen` with agent network, trace list, and result card.
6. Write unit tests.
7. Run: `./gradlew testDebugUnitTest assembleDebug`.
8. Verify every AC individually.

## Final Response Format

Provide:
- Implementation Summary
- Files Created / Modified
- Agent Network Evidence (supervisor + specialists shown)
- Delegation Trace Evidence
- Tests Executed and Results
- AC1–AC10 PASS/FAIL with evidence
- Known Limitations
- Recommended Next Ticket: JIRA-19

> Never claim an AC is PASS without evidence.
