# JIRA-13 — Agent Memory Screen

> **Roadmap module:** 13 — Memory in AI Agents
> **Epic:** Agentic AI Full Stack — Android Client
> **Backend dependency:** NAI-AI-13 (memory endpoints — GET/DELETE /v1/memory/{session_id})

---

You are implementing JIRA-13 — Agent Memory Screen for Nexus AI.

## Jira Title

Agent Memory Screen

## Description

Deliver the agent memory screen where users can view session and long-term memory entries,
filter by category, search by keyword, see retention information, and clear or delete entries.
Memory data is fetched from the FastAPI memory endpoints from NAI-AI-13.

## Required Skills

- android
- compose
- testing

## Pre-Implementation Checklist

Before making any changes:

1. Read and understand:
   - This ticket and all Acceptance Criteria below
   - `.kiro/steering/05-security-standards.md` — memory content must never be logged.
   - `.kiro/skills/android/SKILL.md`, `.kiro/skills/compose/SKILL.md`
2. Inspect JIRA-08 FastAPI dashboard — reuse the primary Retrofit client.
3. Inspect JIRA-02 design system — reuse filter chips and `NexusCard`.
4. Memory entries may contain sensitive conversation content — never log entry content.

## Architecture Rules

- `MemoryEntry` domain model: id, sessionId, role, content, timestamp, category, retentionTurns.
- Category is a domain enum: SESSION / LONG_TERM / CONVERSATION / AGENT.
- Search filtering is done client-side on the loaded list (not a separate API call).
- Delete confirmation dialog is required before any destructive operation.
- Sensitive content is never logged — only entry IDs and metadata in debug logs.

## Implementation Task

### Feature: Memory Screen (`feature/memory/`)
- Session ID input field with Load button → `GET /v1/memory/{session_id}`
- Category filter chip row: All / Session / Long-term / Conversation / Agent
- Search field: filters displayed entries by keyword (client-side)
- Memory entry list:
  - Entry card: role chip (user/assistant), content preview (first 100 chars, truncated),
    timestamp, retention badge ("42 turns remaining" or "Permanent")
  - Tap to expand full content
  - Swipe-to-delete with undo snackbar
- Action buttons:
  - Clear Session Memory → `DELETE /v1/memory/{session_id}` with confirmation dialog
  - Individual delete via swipe
- Empty state card: "No memory entries found"
- Privacy notice footer: "Memory may contain conversation content. Manage with care."

### Domain
- `MemoryEntry` data class: id, sessionId, role, content, timestamp, category, retentionTurns
- `MemoryCategory` enum: SESSION / LONG_TERM / CONVERSATION / AGENT
- `GetMemoryEntriesUseCase` calling `MemoryRepository`
- `ClearSessionMemoryUseCase`
- `DeleteMemoryEntryUseCase`

## Acceptance Criteria

| # | Criterion |
|---|---|
| AC1 | Memory entries are loaded for a given session ID and displayed as cards. |
| AC2 | Category filter chips filter the visible list. |
| AC3 | Search field filters entries by content keyword (client-side). |
| AC4 | Retention information is shown per entry (turns remaining or Permanent). |
| AC5 | Swipe-to-delete removes an entry with an undo snackbar. |
| AC6 | Clear Session Memory requires a confirmation dialog before executing. |
| AC7 | Empty state shows a "No memory entries found" card. |
| AC8 | Memory content is never logged (verified by log inspection in tests). |
| AC9 | All use cases and `MemoryViewModel` are unit tested with fake repositories. |
| AC10 | Debug build succeeds. |

## Workflow

1. Define `MemoryEntry`, `MemoryCategory` domain models.
2. Implement `MemoryRepository` interface and impl.
3. Implement three use cases.
4. Implement `MemoryViewModel` with search filter logic.
5. Implement `MemoryScreen` with filter chips, search, and swipe-to-delete.
6. Add confirmation dialog for clear session.
7. Write unit tests — verify no content logging.
8. Run: `./gradlew testDebugUnitTest assembleDebug`.
9. Verify every AC individually.

## Final Response Format

Provide:
- Implementation Summary
- Files Created / Modified
- Filter and Search Evidence
- Confirmation Dialog Evidence
- Tests Executed and Results
- AC1–AC10 PASS/FAIL with evidence
- Known Limitations
- Recommended Next Ticket: JIRA-14

> Never claim an AC is PASS without evidence.
