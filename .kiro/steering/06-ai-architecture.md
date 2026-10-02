# 06 — AI Architecture

## Core Principle

AI provider implementations are a detail. The application never depends on them directly. Every feature, orchestration component, agent, and RAG pipeline interacts with AI through domain-level abstractions only.

---

## Dependency Chain

```
UI (Compose)
    ↓
AI Orchestration  (Domain interface — coordinates all AI subsystems)
    ↓
AI Provider Interface  (Domain — single contract for all providers)
    ↓
Provider Implementation  (Data — Gemini, OpenAI, On-device, Future)
```

**Provider SDK models must never appear in Domain or Presentation code.**

Mappers in the Data layer convert provider-specific types to domain types at the boundary.

---

## Core Abstractions

The following domain-level concepts must exist. Naming is illustrative — exact names are defined in Specs:

```
AIProvider          — contract for a single AI backend
AIRequest           — provider-agnostic request (messages, tools, config)
AIResponse          — provider-agnostic single response
AIStreamClient      — contract for streaming responses
AIStreamEvent       — sealed type covering all streaming event kinds
```

### AIStreamEvent types

```
Started     — stream has begun
Token       — incremental text token received
ToolCall    — model requests a tool invocation
ToolResult  — result of a tool invocation fed back to the model
Completed   — stream finished successfully
Error       — stream terminated with an error
```

All streaming consumers interact with `AIStreamEvent` only, never with provider SDK streaming types.

---

## AI Orchestration

The orchestration layer coordinates all AI subsystems in one place. It is the single entry point for features that need AI capability.

Orchestration responsibilities:
- Route requests to the appropriate AI provider
- Assemble conversation context (history, memory, system prompt)
- Inject RAG-retrieved context before generation
- Invoke agent execution when required
- Register and resolve application-level tools
- Handle MCP-sourced tools (via Tool abstraction — not MCP transport directly)
- Manage streaming lifecycle
- Apply retry and fallback policies

Orchestration must not:
- Import Compose or any UI type
- Import provider-specific SDK classes
- Directly access Room, DataStore, or any storage layer
- Directly access MCP transport

---

## Provider Implementations

Each provider implementation lives in `ai/providers/` (Data layer) and:
- Implements the `AIProvider` / `AIStreamClient` domain interface
- Maps all provider-specific request/response types to domain types internally
- Handles provider-specific error codes and maps them to domain error types
- Handles provider-specific rate limiting, retries, and backoff internally
- Exposes no provider SDK types past its own class boundary

Supported providers (implemented when the corresponding Jira ticket is executed):
- Gemini (Google AI / Vertex AI)
- OpenAI (GPT models)
- On-device (MediaPipe LLM Inference / ML Kit / future on-device frameworks)
- Future providers added without changes to Orchestration or feature code

---

## Streaming

All streaming interactions use `Flow<AIStreamEvent>`. Consumers collect the flow and react to each event type.

Rules:
- Streaming flows must be cancellable — respect coroutine cancellation
- Partial responses accumulated from `Token` events must be handled by the consumer or orchestration, not the provider implementation
- `Error` events must carry structured domain error types, not raw provider exceptions
- Providers must emit `Completed` or `Error` as the terminal event — never leave a flow open indefinitely

---

## Tool Integration with AI

Application-level tools are registered with orchestration. When the AI model emits a `ToolCall` event:

1. Orchestration resolves the tool by name from the tool registry
2. Orchestration validates the tool input against the tool's schema
3. Orchestration executes the tool with timeout
4. Orchestration feeds the result back to the AI as a `ToolResult` event
5. The loop continues until `Completed` or step limit reached

MCP-sourced tools are converted to application-level Tool models by the MCP adapter before registration. Orchestration has no awareness of MCP transport.

---

## RAG Integration with AI

RAG context assembly happens before AI generation is initiated:

```
User Query
    ↓
RAG Retrieval  (via RAG use case / repository)
    ↓
Context Assembly  (retrieved chunks + metadata assembled into context block)
    ↓
AIRequest construction  (context block injected into request)
    ↓
AI Orchestration  (sends assembled request to provider)
```

RAG implementation details never appear in Compose or ViewModel. The ViewModel calls an orchestration use case; context assembly is opaque to the caller.

Context size is always bounded. Orchestration must enforce a maximum token/character budget on assembled context.

---

## Agent Integration with AI

Agents use orchestration as their AI backend, not the provider directly:

```
Agent Engine
    ↓
Orchestration  (plan step: send current state + available tools to AI)
    ↓
AI Provider  (returns plan / tool call decision)
    ↓
Agent Engine  (executes selected tool, observes result, loops)
```

Agent execution rules:
- Maximum step count enforced before execution begins
- Each step has an individual timeout
- Agent-level cancellation propagates to all in-flight tool executions
- Agent never calls the AI provider directly — always via orchestration

---

## Memory Integration with AI

Conversation memory is retrieved and injected into the AI request context by orchestration:

```
New User Message
    ↓
Memory Repository  (retrieve relevant history)
    ↓
Context Assembly  (history injected into AIRequest)
    ↓
AI Generation
    ↓
Memory Repository  (persist new turn)
```

Memory retrieval and persistence are always mediated by a repository interface. Orchestration calls the interface; it has no knowledge of Room, DataStore, or any storage detail.

---

## On-Device AI

On-device AI providers implement the same `AIProvider` / `AIStreamClient` domain interfaces as cloud providers.

Rules:
- On-device vs. cloud routing is configured at the application level (DataStore preference or build config), not hardcoded in features
- Cloud fallback when on-device inference is unavailable must be explicit — never silent or automatic without configuration
- On-device model loading, warm-up, and lifecycle management are handled within the provider implementation, not in orchestration or features
- On-device providers must handle the case where the model file is not yet downloaded

---

## Error Taxonomy

Define a sealed domain error hierarchy for AI operations:

```
AIError
 ├── NetworkError        — connectivity or server unreachable
 ├── AuthError           — invalid or expired API key/token
 ├── RateLimitError      — provider rate limit exceeded
 ├── ContextLengthError  — input exceeds model context window
 ├── ContentFilterError  — request or response blocked by safety filter
 ├── TimeoutError        — request or streaming exceeded timeout
 ├── ProviderError       — provider-specific error with mapped code
 └── UnknownError        — unmapped provider error
```

Every provider implementation maps its own error types to this hierarchy. Callers above the provider never see raw provider exceptions.

---

## Configuration

AI provider selection, model names, temperature, and other parameters are configurable:
- Developer defaults in a build-config or constants file
- User-overridable settings stored in DataStore (via Settings repository)
- Never hardcoded inline in orchestration or feature code

API keys are never stored in source code. See `05-security-standards.md`.
