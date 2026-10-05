# JIRA-19 — AI Evaluation Dashboard Screen

> **Roadmap module:** 19 — AI Evaluation
> **Epic:** Agentic AI Full Stack — Android Client
> **Backend dependency:** NAI-AI-19 (evaluation reports — GET /v1/evaluation/reports, GET /v1/evaluation/reports/{run_id})

---

You are implementing JIRA-19 — AI Evaluation Dashboard Screen for Nexus AI.

## Jira Title

AI Evaluation Dashboard Screen

## Description

Deliver an AI evaluation dashboard screen showing RAG and agent evaluation reports with metric
cards for faithfulness, relevance, hallucination score, latency, and cost. Users can browse
evaluation runs, compare two reports side-by-side, and drill into metric detail. Calls the
FastAPI evaluation endpoints from NAI-AI-19.

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
2. Inspect JIRA-08 FastAPI dashboard — reuse the primary Retrofit client.
3. Inspect JIRA-02 design system — reuse metric badges, `NexusCard`, colour coding.
4. Evaluation report data may come from a test suite — metrics are always [0–1] floats.

## Architecture Rules

- `EvaluationReport` domain model: runId, timestamp, modelVersion, datasetVersion, metrics, operational.
- `MetricScore` data class: name, value (0.0–1.0), threshold.
- `OperationalMetrics` data class: p50Ms, p95Ms, estimatedCostUsd, promptTokens, completionTokens.
- Colour coding: green ≥ 0.8, amber ≥ 0.5, red < 0.5 — consistent with JIRA-15.
- Two-report comparison is shown on a dedicated detail screen — not a dialog.

## Implementation Task

### Feature: Evaluation Dashboard (`feature/evaluation/`)
- Report list screen:
  - Filter chips: All / RAG / Agent
  - Report summary cards: model version, dataset version, date, overall score badge
  - Compare button (enabled when exactly 2 cards are selected)
- Report detail screen (tapped from list):
  - Header: model version, dataset version, run date
  - RAG metric cards: Faithfulness, Answer Relevance, Hallucination Score, Context Precision, Context Recall
  - Operational metrics card: P50 latency, P95 latency, cost/request, token usage
  - Agent metrics card (visible for agent reports): Task Completion Rate, Avg Steps, Step Limit Hit Rate
  - Each metric card: score bar (colour-coded), value label, threshold indicator
- Compare screen (from Compare button):
  - Side-by-side metric scores for two selected reports
  - Delta indicators: ↑ (green) / ↓ (red) / = (grey) per metric
  - Report A and Report B headers with model version

### Domain
- `EvaluationReport` data class (see above)
- `MetricScore`, `OperationalMetrics`, `AgentMetrics` data classes
- `GetEvaluationReportsUseCase`, `GetEvaluationReportDetailUseCase`

## Acceptance Criteria

| # | Criterion |
|---|---|
| AC1 | Report list shows available runs with model version, dataset version, and date. |
| AC2 | RAG metric cards show faithfulness, relevance, and hallucination score with colour coding. |
| AC3 | Operational metrics show P50 and P95 latency and cost per request. |
| AC4 | Agent metrics are shown for agent evaluation reports. |
| AC5 | Metric score bars are colour-coded (green / amber / red). |
| AC6 | Two reports can be selected and compared side-by-side with delta indicators. |
| AC7 | Filter chips filter the report list by type (RAG / Agent). |
| AC8 | Loading and error states are handled. |
| AC9 | Use cases and `EvaluationViewModel` are unit tested with fake repositories. |
| AC10 | Debug build succeeds. |

## Workflow

1. Define `EvaluationReport`, `MetricScore`, `OperationalMetrics`, `AgentMetrics` domain models.
2. Implement `EvaluationRepository` interface and impl.
3. Implement two use cases.
4. Implement `EvaluationViewModel`.
5. Implement report list screen with filter chips.
6. Implement report detail screen with all metric cards.
7. Implement compare screen with delta indicators.
8. Write unit tests.
9. Run: `./gradlew testDebugUnitTest assembleDebug`.
10. Verify every AC individually.

## Final Response Format

Provide:
- Implementation Summary
- Files Created / Modified
- Metric Card Evidence (colour coding shown)
- Compare Screen Evidence (delta indicators)
- Tests Executed and Results
- AC1–AC10 PASS/FAIL with evidence
- Known Limitations
- Recommended Next Ticket: JIRA-20

> Never claim an AC is PASS without evidence.
