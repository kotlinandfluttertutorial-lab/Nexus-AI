# 03 — Coding Standards

## Kotlin Standards

### Immutability First
- Prefer `val` over `var` at all times. Use `var` only when mutation is genuinely required and document why.
- Prefer immutable collections (`List`, `Map`, `Set`) over mutable variants. Expose mutable collections only within the scope that owns them.
- Use `copy()` on data classes to produce modified versions rather than mutating fields.

### Types and Nullability
- Avoid unnecessary nullable types. If a value is always present, do not make it nullable.
- Never use `!!` (force unwrap). Use `?: error(...)`, `requireNotNull()`, safe calls, or explicit null handling instead.
- Use `sealed interface` or `sealed class` for exhaustive state, event, and result types.
- Use `data class` for value objects and data models.
- Use `Result<T>` or a project-defined `Outcome<T>` sealed type for operations that can fail.

### Functions
- Keep functions focused on one responsibility.
- Prefer `suspend` functions for asynchronous single-value operations.
- Prefer `Flow` for asynchronous streams of values.
- Use extension functions to add behavior to existing types when it improves clarity — not to bypass encapsulation.
- Use named arguments when a function has more than two parameters of the same type or when intent is unclear from position.
- Keep function bodies short. Extract private helper functions rather than building deeply nested logic.

### Classes
- Avoid God objects. A class that coordinates everything is a design smell — split responsibilities.
- Keep classes focused. If a class requires more than ~5 injected dependencies, consider whether it has too many responsibilities.
- Do not place business logic in UI classes, data classes, or DTOs.
- Use `object` only for stateless utilities and companion objects. Never use `object` to hold mutable state.

### Structured Concurrency
- Always launch coroutines in a defined scope (`viewModelScope`, `lifecycleScope`, a Hilt-injected `CoroutineScope`).
- Use `supervisorScope` when sibling coroutine failures must not cancel each other.
- Never call `GlobalScope.launch` in application code.
- Never use `runBlocking` on the main thread or in production coroutine code.
- Always handle `CancellationException` correctly — rethrow it, do not swallow it.

---

## Naming Conventions

| Construct | Convention | Example |
|---|---|---|
| Classes / Interfaces | PascalCase | `ChatRepository`, `AIProvider` |
| Functions / Methods | camelCase | `sendMessage()`, `fetchChunks()` |
| Variables / Properties | camelCase | `messageList`, `isLoading` |
| Constants (top-level / companion) | UPPER_SNAKE_CASE | `MAX_RETRY_COUNT`, `DEFAULT_TIMEOUT_MS` |
| Packages | lowercase, no underscores | `com.nexusai.feature.chat` |
| Compose functions | PascalCase (as per Compose convention) | `ChatScreen()`, `MessageBubble()` |

---

## Code Organization

- One top-level class per file. Closely related small classes (sealed subclasses, etc.) may co-locate.
- Keep files under ~300 lines as a guideline. Larger files signal a class doing too much.
- Order class members: properties → init → public functions → internal/protected functions → private functions → companion object.
- Place constants in a companion object or a top-level `object` in the same file as the class they belong to. Avoid scattered magic numbers.

---

## Comments and Documentation

Comments explain **why**, not **what**. The code explains what.

Write KDoc on:
- Public API interfaces and their methods
- Non-obvious domain entities and their fields
- Use cases (what business operation they represent)
- Complex algorithms or non-obvious logic

Avoid:
- Comments that restate the code (`// increment counter`)
- Commented-out code in committed files (delete it; Git preserves history)
- TODO comments without a linked Jira ticket reference

---

## Error Handling

- Never swallow exceptions silently. Always log (at minimum) or propagate.
- Do not expose raw `Exception` or stack traces to users. Map errors to domain error types before surfacing to UI.
- Define structured error types per domain area (e.g., `AIError`, `RAGError`, `MCPError`, `AgentError`).
- Use sealed classes/interfaces for error hierarchies so callers are forced to handle all cases.
- Include enough context in error types to allow meaningful recovery or user messaging.

---

## Logging

Use structured logging with appropriate log levels.

Never log:
- API keys
- Auth tokens or refresh tokens
- Passwords or PINs
- Encryption keys
- Sensitive user prompts or personal data
- Full AI model responses that may contain sensitive data

Use tags consistently. Each module/component should have a defined log tag constant.

Log levels:
- `ERROR` — unexpected failures that need investigation
- `WARN` — recoverable issues or degraded functionality
- `INFO` — significant lifecycle events (app start, session start)
- `DEBUG` — development-time diagnostics (strip or disable in release builds)
- `VERBOSE` — detailed traces (never in release builds)

---

## Imports and Dependencies

- Do not import provider-specific SDK types into Domain or Presentation layers.
- Do not import Room, Retrofit, or OkHttp types outside the Data layer.
- Do not import Compose into Domain or Data layers.
- Keep build.gradle dependencies minimal. Add a dependency only when it provides genuine value; document why non-obvious dependencies are included.

---

## Provider-Specific Logic Isolation

Provider-specific code (Gemini SDK models, OpenAI SDK models, on-device model APIs) must live entirely within the Data layer provider implementation classes.

Mappers convert provider-specific types to domain types at the boundary. Domain types must never reference provider-specific type names.
