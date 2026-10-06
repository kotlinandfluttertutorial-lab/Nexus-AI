# MCP Server & Tools Screen

> **Roadmap module:** 17 — Model Context Protocol (MCP)
> **Epic:** Agentic AI Full Stack — Android Client
> **Backend dependency:** NAI-AI-17 (MCP endpoints — GET /v1/mcp/tools, GET /v1/mcp/resources, POST /v1/mcp/tools/{name}/invoke)

---

You are implementing JIRA-17 — MCP Server & Tools Screen for Nexus AI.

## Jira Title

MCP Server & Tools Screen

## Description

Deliver an MCP integration screen with two tabs — MCP Tools and MCP Resources. Users configure
the MCP server URL (through the FastAPI backend), view discovered tools and resources, invoke
a tool by filling in its input schema, and read resource content. Calls the FastAPI MCP
endpoints from NAI-AI-17.

## Required Skills

- android
- compose
- testing

## Pre-Implementation Checklist

Before making any changes:

1. Read and understand:
   - This ticket and all Acceptance Criteria below
   - `.kiro/steering/05-security-standards.md` — tool inputs may contain sensitive data, never log them.
   - `.kiro/skills/android/SKILL.md`, `.kiro/skills/compose/SKILL.md`
2. Inspect JIRA-08 FastAPI dashboard — reuse Retrofit client.
3. Inspect JIRA-02 design system — reuse `NexusCard`, status chips.
4. The Android app does NOT connect to MCP directly — it calls the FastAPI MCP proxy endpoints.

## Out of Scope

- Registering or publishing MCP servers from within the app
- Tool or resource creation

## Architecture Rules

- `MCPTool` domain model: name, description, inputSchema (Map<String, Any>).
- `MCPResource` domain model: uri, name, description, mimeType.
- `MCPToolInvocationRequest` domain model: toolName, arguments (Map<String, String>).
- Tool invocation input fields are rendered dynamically from `inputSchema`.
- Tool arguments are never logged.
- Timeout error maps to a domain `TimeoutError` — not a raw exception.

## Implementation Task

### Feature: MCP Screen (`feature/mcp/`)
- Connection status header:
  - Server label + Connection status chip (Connected / Disconnected / Reconnecting)
  - Refresh button
- `TabRow`: Tools tab | Resources tab

#### Tools Tab
- Tool cards: name, description (first 80 chars), input schema summary chip
- Tap tool card → Tool Invocation bottom sheet:
  - Dynamic input form: one `TextField` per property in `inputSchema`
  - Invoke button → `POST /v1/mcp/tools/{name}/invoke`
  - Result card: success content or error message
  - Timeout error card with retry action

#### Resources Tab
- Resource cards: URI, name, MIME type chip
- Tap resource card → Resource Content bottom sheet:
  - `GET /v1/mcp/resources/{uri}`
  - Content rendered in a scrollable monospace text block (for text MIME types)
  - MIME type badge

### Domain
- `MCPTool` data class: name, description, inputSchema
- `MCPResource` data class: uri, name, description, mimeType
- `MCPToolInvocationRequest` / `MCPToolInvocationResult`
- `GetMCPToolsUseCase`, `GetMCPResourcesUseCase`, `InvokeMCPToolUseCase`, `ReadMCPResourceUseCase`

## Acceptance Criteria

| # | Criterion |
|---|---|
| AC1 | Connection status chip shows live Connected or Disconnected status. |
| AC2 | Tools tab lists all discovered MCP tools as cards. |
| AC3 | Tool invocation bottom sheet renders dynamic input fields from the tool's schema. |
| AC4 | Tool invocation result is displayed in the bottom sheet. |
| AC5 | Timeout error shows a retry action. |
| AC6 | Resources tab lists all discovered MCP resources as cards. |
| AC7 | Resource content is shown in a scrollable monospace text block. |
| AC8 | Tool arguments are never written to logs. |
| AC9 | Use cases and `MCPViewModel` are unit tested with fake repositories. |
| AC10 | Debug build succeeds. |

## Workflow

1. Define domain models.
2. Implement `MCPRepository` interface and impl for all four operations.
3. Implement four use cases.
4. Implement `MCPViewModel`.
5. Implement `MCPScreen` with connection header, TabRow, and bottom sheets.
6. Write unit tests — verify no argument logging.
7. Run: `./gradlew testDebugUnitTest assembleDebug`.
8. Verify every AC individually.

## Final Response Format

Provide:
- Implementation Summary
- Files Created / Modified
- Tool Invocation Evidence (dynamic form + result)
- Resource Content Evidence
- Tests Executed and Results
- AC1–AC10 PASS/FAIL with evidence
- Known Limitations
- Recommended Next Ticket: JIRA-18 (Multi-Agent System Screen)

> Never claim an AC is PASS without evidence.
