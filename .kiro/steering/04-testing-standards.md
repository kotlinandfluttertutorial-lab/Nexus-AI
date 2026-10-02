# 04 — Testing Standards

## Testing Stack

| Tool | Purpose |
|---|---|
| JUnit 4 / JUnit 5 | Unit and integration test runner |
| MockK | Kotlin-idiomatic mocking |
| kotlinx-coroutines-test | Coroutine and Flow testing (`TestCoroutineScheduler`, `runTest`, `Turbine`) |
| Turbine | Flow emission testing |
| Compose UI Testing | Compose screen and interaction testing |
| Hilt Testing | Hilt component replacement in tests |
| Robolectric | JVM-based Android unit tests where instrumentation is not required |

---

## What Must Be Tested

Every subsystem has mandatory test coverage:

| Subsystem | Required Tests |
|---|---|
| Use cases | Business logic, happy path, all failure branches |
| ViewModels | UiState transitions, event emission, use case delegation |
| Repositories | Data source coordination, mapping, caching logic |
| Mappers | Every mapping function, edge cases, nulls |
| AI Orchestration | Request routing, streaming, error propagation, cancellation |
| AI Provider abstractions | Request construction, response parsing, streaming events, errors |
| MCP | Connection lifecycle, tool discovery, invocation, timeout, cancellation, error mapping |
| Tool execution | Input validation, execution, timeout, error model |
| Agents | Step execution, step limit enforcement, cancellation, tool selection, error recovery |
| RAG pipeline | Each stage independently: extraction, chunking, embedding, retrieval, context assembly |
| Memory | Persist, retrieve, clear, boundary conditions |
| Background processing (WorkManager) | Worker logic, result handling, retry conditions |
| Compose screens | Initial state, loading, success, error, user interactions, navigation triggers |

---

## Test Doubles

Use the following hierarchy — prefer fakes over mocks for complex collaborators:

- **Fake** — a working lightweight implementation (preferred for repositories, AI providers, MCP clients, tools, embeddings, vector stores)
- **Stub** — returns fixed values, no behavior verification
- **Mock** (MockK) — verify interaction when behavior verification is the goal; use sparingly

**AI tests must never use real external AI providers.**

Required fakes for the test suite:

```
FakeAIProvider          — deterministic responses, controllable streaming
FakeMCPClient           — configurable tool list, controllable invocation results
FakeTool                — records calls, returns configurable results
FakeEmbeddingProvider   — deterministic fixed-dimension vectors
FakeVectorStore         — in-memory store, deterministic similarity results
FakeRepository<T>       — in-memory implementation of each repository interface
```

All fakes live in `core/testing/` and are reusable across all test modules.

---

## Coroutine and Flow Testing

- Use `runTest` for all coroutine-based tests
- Use `TestCoroutineScheduler` / `UnconfinedTestDispatcher` or `StandardTestDispatcher` as appropriate
- Use Turbine for testing Flow emissions: `flow.test { ... }`
- Always advance time explicitly when testing time-dependent logic
- Test cancellation: ensure coroutines cancel cleanly and release resources

---

## Compose UI Testing

Compose tests must cover for each screen:

1. **Initial state** — correct content rendered on first composition
2. **Loading state** — progress indicators shown, interactions disabled appropriately
3. **Success state** — data renders correctly
4. **Error state** — error message shown, retry available where applicable
5. **User interaction** — tap, scroll, input, submit produce correct state transitions
6. **Navigation** — navigation events emitted correctly on actions

Use `composeTestRule.onNodeWithTag()` for test-stable node targeting. Define `TestTag` constants in the component or a companion object.

Do not rely on text strings for node lookup in localized apps — use test tags.

---

## Test Quality Rules

Tests must be:

- **Deterministic** — same result every run, no dependency on time, network, or filesystem
- **Isolated** — no shared mutable state between tests; each test sets up its own state
- **Repeatable** — passes consistently in CI and on any developer machine
- **Readable** — test name describes the scenario; arrange/act/assert structure is clear
- **Fast** — unit tests run in milliseconds; avoid unnecessary delays

Naming convention:
```
fun `given [context], when [action], then [expected outcome]`()
```

Or for simpler cases:
```
fun `sendMessage emits loading then success state`()
```

---

## What Is Forbidden

- Using real network calls in unit or integration tests
- Using real AI provider APIs in any automated test
- Using `Thread.sleep()` in tests — use virtual time via `TestCoroutineScheduler`
- Weakening assertions to make a failing test pass
- Commenting out test assertions
- Skipping tests with `@Ignore` without a linked Jira ticket and expiry plan
- Sharing mutable state between test cases

---

## Test Location

| Test type | Location |
|---|---|
| Unit tests (JVM) | `src/test/` in each module |
| Instrumented tests | `src/androidTest/` in each module |
| Shared fakes and utilities | `core/testing/src/main/` |
| Compose UI tests | `src/androidTest/` in the feature module |

---

## Coverage Expectation

There is no arbitrary percentage target. Coverage is assessed by:

- All use cases covered
- All ViewModel state transitions covered
- All error branches covered
- All AI subsystem interactions covered via fakes
- No critical path left untested

Coverage gaps must be justified and tracked in Jira, not silently accepted.
