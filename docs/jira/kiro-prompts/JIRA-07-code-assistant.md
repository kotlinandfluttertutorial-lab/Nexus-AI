# JIRA-07 — API Test Runner Screen

> **Roadmap module:** 07 — LLM Service Testing (Postman & Bruno)
> **Epic:** Agentic AI Full Stack — Android Client
> **Backend dependency:** NAI-AI-07 (Flask + FastAPI test suite; services must be running)

---

You are implementing JIRA-07 — API Test Runner Screen for Nexus AI.

## Jira Title

API Test Runner Screen

## Description

Deliver an on-device API test runner screen that executes a predefined suite of API checks
against the LLM services and displays pass/fail results with response details. Demonstrates
structured API testing concepts (validation, auth, error handling, streaming) from module 07.

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
2. Inspect JIRA-06 (Flask screen) and JIRA-08 (FastAPI screen) — reuse their Retrofit
   clients for the test targets.
3. Inspect JIRA-02 design system — reuse `NexusCard`, status badges.

## Architecture Rules

- Test case definitions are data classes in Domain — not hardcoded in Compose.
- Each test case runs in a coroutine via `ApiTestRunner` use case.
- Tests run sequentially with a delay between them to avoid overwhelming the service.
- Test execution must not block the UI thread.
- Results are immutable data classes accumulated in ViewModel StateFlow.

## Implementation Task

### Feature: API Test Runner (`feature/api-tester/`)
- Header card: target service selector (Flask / FastAPI), base URL display
- Run All Tests button with a progress indicator (x/total)
- Test case list:
  1. Health check → expect 200
  2. Chat completion → expect non-empty response body
  3. Models list → expect non-empty array
  4. Missing messages body → expect 400/422
  5. Invalid API key → expect 401
  - Each row: test name, status badge (PASS / FAIL / ERROR / RUNNING), HTTP status code
- Expandable detail per test:
  - Request method + URL
  - Request body (truncated)
  - Response body (truncated)
  - Expected vs actual assertion result
- Export Results button: copies a text summary to clipboard

### Domain
- `ApiTestCase` data class: id, name, description, expectedStatus
- `ApiTestResult` data class: testCase, actualStatus, passed, responseBody, durationMs
- `RunApiTestSuiteUseCase` — sequential execution returning `Flow<ApiTestResult>`

## Acceptance Criteria

| # | Criterion |
|---|---|
| AC1 | At least 5 predefined test cases are shown (health, chat, models, missing body, invalid key). |
| AC2 | Run All Tests executes the suite and shows progress (x/5). |
| AC3 | Each test row shows PASS / FAIL / ERROR badge and HTTP status code. |
| AC4 | The running test is highlighted while executing. |
| AC5 | Failed tests show expected vs actual in the expandable detail. |
| AC6 | Test execution does not block the UI thread. |
| AC7 | Export Results copies a text summary to the clipboard. |
| AC8 | `RunApiTestSuiteUseCase` is unit tested with a fake HTTP client returning controlled responses. |
| AC9 | ViewModel state transitions (Idle → Running → Completed) are unit tested. |
| AC10 | Debug build succeeds. |

## Workflow

1. Define `ApiTestCase` and `ApiTestResult` domain models.
2. Implement `RunApiTestSuiteUseCase` with sequential coroutine execution.
3. Implement `ApiTestRunnerViewModel` with `StateFlow<ApiTestUiState>`.
4. Implement `ApiTestRunnerScreen` with test list and expandable detail.
5. Add Export Results clipboard action.
6. Write unit tests for use case and ViewModel.
7. Run: `./gradlew testDebugUnitTest assembleDebug`.
8. Verify every AC individually.

## Final Response Format

Provide:
- Implementation Summary
- Files Created / Modified
- Test Suite Execution Evidence (pass/fail screenshot or test output)
- Export Results Evidence
- Tests Executed and Results
- AC1–AC10 PASS/FAIL with evidence
- Known Limitations
- Recommended Next Ticket: JIRA-08

> Never claim an AC is PASS without evidence.
