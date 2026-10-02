# 01 — Architecture

## Clean Architecture Rule

Dependency direction is strictly one way:

```
Presentation → Domain → Data
```

Never reverse this direction. Domain must never import from Presentation or Data. Data must never import from Presentation.

---

## Presentation Layer

Contains:
- Compose UI (screens, components)
- ViewModels
- UiState (sealed interfaces or data classes)
- UiEvent (one-shot events via SharedFlow where appropriate)
- StateFlow / SharedFlow for UI consumption
- Presentation-specific display models

Must not contain:
- Business rules or domain logic
- Direct Retrofit calls
- Direct Room access
- Direct AI provider SDK calls
- Direct MCP transport access
- Database entity types

---

## Domain Layer

Contains:
- Entities (pure Kotlin, no Android/framework imports where practical)
- Repository interfaces
- Use cases (one primary responsibility each)
- Business rules
- Application-level tool abstractions
- AI orchestration interfaces
- Agent interfaces
- RAG pipeline interfaces
- Memory repository interfaces

Domain must remain framework-independent where practical. Android imports in domain are acceptable only when unavoidable (e.g., Uri for document handling).

---

## Data Layer

Contains:
- Repository implementations
- DTOs (network response models)
- Database entities (Room)
- Network clients (Retrofit services)
- Remote and local data sources
- Mappers (DTO → Entity, DB entity → Domain entity)
- AI provider implementations
- MCP adapter implementations
- Embedding implementations
- Vector store implementations

---

## UI Architecture Pattern

```
Composable
    ↓  (user event)
ViewModel
    ↓  (calls)
UseCase
    ↓  (calls)
Repository Interface
    ↓  (implemented by)
Data Source
```

Rules:
- Composables are stateless where possible; state is hoisted to ViewModel
- UiState is immutable (data class with `val` fields)
- ViewModels expose `StateFlow<UiState>` and `SharedFlow<UiEvent>`
- One-shot side effects (navigation, snackbar) use SharedFlow/Channel
- Unidirectional data flow always

---

## Dependency Injection

Use Hilt throughout. Every dependency is injected, not constructed inline.

Never use:
- Manual service locators
- Global mutable singletons (unless technically unavoidable and documented)
- `object` singletons holding mutable state
- Hidden dependencies (dependencies not visible in constructor)

Hilt modules live in the `:di` or feature-specific module as appropriate.

---

## AI Dependency Chain

Feature code must never import AI provider SDKs directly.

```
Feature (Presentation / Domain)
    ↓
AI Orchestration (Domain interface)
    ↓
AI Provider Interface (Domain)
    ↓
Provider Implementation (Data: Gemini, OpenAI, On-device, etc.)
```

Adding or swapping a provider must not require changes to feature code.

---

## MCP Dependency Chain

```
Agent / AI Orchestration (Domain)
    ↓
Application Tool (Domain abstraction)
    ↓
MCP Adapter (Data)
    ↓
MCP Transport (Data — network/stdio/SSE)
```

Prohibited:
```
Compose → MCP Transport   ✗
Compose → MCP Adapter     ✗
ViewModel → MCP Transport ✗
```

---

## Tool Architecture

Every executable capability exposed to AI or agents must implement a common application-level tool abstraction.

Every tool must define:
- Unique name
- Human-readable description
- Typed input schema
- Typed output
- Input validation (reject invalid input before execution)
- Execution timeout
- Structured error model

Never execute arbitrary unvalidated tool input.

---

## Agent Architecture

Agents must have:
- Structured task definition
- Explicit context collection step
- Planning step before tool selection
- Tool selection with validation
- Tool execution with timeout
- Observation of tool results
- Step counter with enforced maximum
- Cancellation via coroutine cancellation
- Structured error handling and recovery strategy

Never implement unbounded recursive agent execution. Always enforce a maximum step limit.

---

## RAG Architecture

RAG pipeline stages must preserve identity throughout:
- Document ID and source
- Chunk ID and position within document
- Page metadata where available
- Retrieval score / relevance metadata

Context size passed to AI must be bounded — never pass unbounded text.

Every pipeline stage must handle its own failure mode:
- Extraction failure → structured error, no silent skip
- Embedding failure → structured error, retry policy defined
- Vector-store failure → structured error
- Empty retrieval → explicit empty-result handling (do not hallucinate context)
- AI generation failure → surface error to caller

---

## Memory Architecture

Conversation memory is always accessed through a repository interface defined in Domain. Presentation and Orchestration never access Room, DataStore, or any storage mechanism directly for memory operations.

---

## Module Structure (Target)

```
app/                     — Application entry point, DI setup, navigation root
core/domain/             — Entities, use cases, repository interfaces, tool abstractions
core/data/               — Repository implementations, data sources, mappers
core/ui/                 — Shared Compose components, theme, design system
core/testing/            — Shared test fakes, utilities, base test classes
feature/chat/            — Chat feature module
feature/voice/           — Voice feature module
feature/code/            — Code assistant feature module
feature/image/           — Image generation feature module
feature/documents/       — Document processing feature module
ai/orchestration/        — AI orchestration coordination
ai/providers/            — Provider implementations (Gemini, OpenAI, on-device)
ai/rag/                  — RAG pipeline implementation
ai/agents/               — Agent engine implementation
ai/mcp/                  — MCP adapter and transport
ai/memory/               — Memory data implementation
ai/tools/                — Application-level tool registry and implementations
```

Module boundaries enforce the dependency rules above. Feature modules must not cross-import each other directly.
