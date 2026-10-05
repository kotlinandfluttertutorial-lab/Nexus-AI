# Nexus AI — Kiro Prompts for 20 Jira Tickets

## PURPOSE

This document is the **single source of truth** for all 20 Nexus AI Jira execution prompts.

Each section defines one Jira ticket's Kiro execution prompt. The corresponding individual prompt
file is generated from this document and stored under `docs/jira/kiro-prompts/`.

Do not edit individual prompt files directly — edit this source document and regenerate.

---

## DOCUMENT STRUCTURE

### Android Application Workstream (JIRA-01 – JIRA-20)

| Section | Jira Ticket | Prompt File |
|---|---|---|
| JIRA-01 | Core Foundation | `kiro-prompts/JIRA-01-core-foundation.md` |
| JIRA-02 | Design System | `kiro-prompts/JIRA-02-design-system.md` |
| JIRA-03 | Authentication & Security | `kiro-prompts/JIRA-03-authentication-security.md` |
| JIRA-04 | AI Provider Abstraction | `kiro-prompts/JIRA-04-ai-provider-abstraction.md` |
| JIRA-05 | Chat | `kiro-prompts/JIRA-05-chat.md` |
| JIRA-06 | Voice | `kiro-prompts/JIRA-06-voice.md` |
| JIRA-07 | Code Assistant | `kiro-prompts/JIRA-07-code-assistant.md` |
| JIRA-08 | Image Generation | `kiro-prompts/JIRA-08-image-generation.md` |
| JIRA-09 | Document Processing | `kiro-prompts/JIRA-09-document-processing.md` |
| JIRA-10 | RAG | `kiro-prompts/JIRA-10-rag.md` |
| JIRA-11 | MCP | `kiro-prompts/JIRA-11-mcp.md` |
| JIRA-12 | Agent Architecture | `kiro-prompts/JIRA-12-agent-architecture.md` |
| JIRA-13 | Tool Execution | `kiro-prompts/JIRA-13-tool-execution.md` |
| JIRA-14 | Memory & Conversation | `kiro-prompts/JIRA-14-memory-conversation.md` |
| JIRA-15 | AI Orchestration | `kiro-prompts/JIRA-15-ai-orchestration.md` |
| JIRA-16 | On-Device AI | `kiro-prompts/JIRA-16-on-device-ai.md` |
| JIRA-17 | Background Processing | `kiro-prompts/JIRA-17-background-processing.md` |
| JIRA-18 | Observability | `kiro-prompts/JIRA-18-observability.md` |
| JIRA-19 | Testing & CI/CD | `kiro-prompts/JIRA-19-testing-cicd.md` |
| JIRA-20 | Production & Deployment | `kiro-prompts/JIRA-20-production-deployment.md` |

### AI Roadmap Workstream (NAI-AI-01 – NAI-AI-20)

| Section | Ticket | Prompt File |
|---|---|---|
| NAI-AI-01 | GenAI Foundations | `kiro-prompts/NAI-AI-01-genai-foundations.md` |
| NAI-AI-02 | Transformer Architecture & Embeddings | `kiro-prompts/NAI-AI-02-transformers-embeddings.md` |
| NAI-AI-03 | Working with LLMs | `kiro-prompts/NAI-AI-03-working-with-llms.md` |
| NAI-AI-04 | Prompt Engineering | `kiro-prompts/NAI-AI-04-prompt-engineering.md` |
| NAI-AI-05 | Self-Hosted LLMs with Ollama | `kiro-prompts/NAI-AI-05-ollama.md` |
| NAI-AI-06 | LLM as a Service with Flask | `kiro-prompts/NAI-AI-06-flask-llm-service.md` |
| NAI-AI-07 | LLM Service Testing | `kiro-prompts/NAI-AI-07-llm-service-testing.md` |
| NAI-AI-08 | Building LLM Services with FastAPI | `kiro-prompts/NAI-AI-08-fastapi-llm-service.md` |
| NAI-AI-09 | Hugging Face & Open-Source LLMs | `kiro-prompts/NAI-AI-09-huggingface.md` |
| NAI-AI-10 | Advanced LLM Features | `kiro-prompts/NAI-AI-10-advanced-llm-features.md` |
| NAI-AI-11 | Agentic AI | `kiro-prompts/NAI-AI-11-agentic-ai.md` |
| NAI-AI-12 | LangChain | `kiro-prompts/NAI-AI-12-langchain.md` |
| NAI-AI-13 | Memory in AI Agents | `kiro-prompts/NAI-AI-13-agent-memory.md` |
| NAI-AI-14 | Retrieval-Augmented Generation | `kiro-prompts/NAI-AI-14-rag.md` |
| NAI-AI-15 | Vector Databases | `kiro-prompts/NAI-AI-15-vector-databases.md` |
| NAI-AI-16 | LangGraph | `kiro-prompts/NAI-AI-16-langgraph.md` |
| NAI-AI-17 | Model Context Protocol | `kiro-prompts/NAI-AI-17-mcp.md` |
| NAI-AI-18 | Multi-Agent Systems | `kiro-prompts/NAI-AI-18-multi-agent.md` |
| NAI-AI-19 | AI Evaluation | `kiro-prompts/NAI-AI-19-ai-evaluation.md` |
| NAI-AI-20 | Deployment & Production AI Applications | `kiro-prompts/NAI-AI-20-production-deployment.md` |

---

## COMMON EXECUTION RULES

Apply these rules to **every** Jira prompt below:

1. Do not start coding before inspecting the repository.
2. Read applicable `.kiro/steering/` files before implementation.
3. Read applicable `.kiro/skills/` files before implementation.
4. Read relevant `.kiro/specs/` before implementation.
5. Inspect existing reusable components before creating new ones.
6. Do not rewrite working code unnecessarily.
7. Preserve existing functionality unless the Jira ticket explicitly requires a change.
8. Maintain: `Presentation → Domain → Data`
9. Do not allow:
   - `Compose → Retrofit`
   - `Compose → Room`
   - `Compose → MCP transport`
   - `Compose → AI provider SDK`
10. Keep provider-specific implementations behind abstractions.
11. Keep business logic outside composables.
12. Use coroutines and Flow for asynchronous operations.
13. Use dependency injection consistently.
14. Add automated tests for the implemented behavior.
15. Never weaken or remove tests merely to obtain a passing build.
16. Run the appropriate Gradle tests/build/lint tasks.
17. Verify every Acceptance Criterion separately.
18. An AC may only be marked `PASS` when implementation/test/build evidence exists.
19. If an AC cannot be verified, mark it `FAIL` or `NOT VERIFIED` and explain why.
20. Do not implement unrelated Jira tickets unless required for compatibility.
21. If another ticket's infrastructure is required, reuse it rather than duplicating it.
22. Update documentation only when the implemented architecture actually changes.
23. Never hardcode secrets or credentials.
24. Never put sensitive information into logs.
25. Follow Steering rules over ad-hoc implementation decisions.

---

## JIRA-01 — Core Foundation

```
You are implementing JIRA-01 — Core Foundation for the Nexus AI Android project.

JIRA TITLE:
Core Foundation

DESCRIPTION:
Establish the application's core architecture and reusable infrastructure.

REQUIRED SKILLS:
android
clean-architecture
testing

Before making any changes:

1. Read and understand:
   - This Jira ticket
   - All Acceptance Criteria below
   - .kiro/steering/
   - Applicable .kiro/skills/
   - Relevant .kiro/specs/
   - Existing repository structure and implementation
2. Inspect existing:
   - Gradle configuration
   - Application/module structure
   - Presentation, Domain, and Data layers
   - Dependency injection
   - Networking
   - Database/storage
   - ViewModels
   - Repositories
   - Use cases
   - Existing error/result handling
   - Existing tests
3. Reuse existing implementations where appropriate.
4. Do not perform unnecessary rewrites.
5. Steering rules take precedence over ad-hoc implementation decisions.

ARCHITECTURE RULES:
- Maintain Presentation → Domain → Data dependency direction.
- Presentation must not directly access Retrofit, Room, MCP transport, or AI provider SDKs.
- Domain should remain framework-independent where practical.
- Repository interfaces belong in Domain.
- Repository implementations belong in Data.
- Use Hilt for dependency injection where already established.
- Use Kotlin coroutines and Flow for asynchronous operations.
- Do not introduce global mutable state.
- Keep business logic outside UI components.

IMPLEMENTATION TASK:
Establish or standardize the core foundation required by the rest of Nexus AI without breaking
existing functionality.

ACCEPTANCE CRITERIA:
AC1: Existing repository architecture is inspected and documented.
AC2: Presentation, Domain and Data responsibilities are clearly separated.
AC3: Existing DI solution is standardized/reused.
AC4: Common Result/Error handling is established.
AC5: Core infrastructure required by later modules is available.
AC6: Existing functionality continues to work.
AC7: Unit-test foundation is available.
AC8: Debug build succeeds.

WORKFLOW:
1. Analyze repository.
2. Identify reusable infrastructure.
3. Create an implementation plan before editing.
4. Implement only required changes.
5. Add/update tests.
6. Run relevant unit tests.
7. Run static analysis/lint where available.
8. Run the debug Gradle build.
9. Verify every AC individually with evidence.

FINAL RESPONSE:
Provide:
- Implementation Summary
- Files Created
- Files Modified
- Architecture Changes
- Tests Executed
- Build/Validation Results
- AC1–AC8 PASS/FAIL with evidence
- Known Limitations
- Recommended Next Jira Ticket

Never claim an AC is PASS without evidence.
```

---

## JIRA-02 — Design System

```
You are implementing JIRA-02 — Design System for Nexus AI.

JIRA TITLE:
Design System

REQUIRED SKILLS:
android
compose
testing

First inspect the repository, .kiro/steering/, applicable .kiro/skills/, relevant .kiro/specs/,
and existing UI components.

Do not create duplicate components if reusable UI already exists.

Follow these rules:
- Use Jetpack Compose and Material 3.
- Prefer stateless reusable composables.
- Hoist state where appropriate.
- Keep business logic out of composables.
- Centralize theme, typography, spacing, and shapes.
- Follow accessibility requirements.
- Preserve existing application behavior.

IMPLEMENT:
Establish the Nexus AI design system required by current and future features.

ACCEPTANCE CRITERIA:
AC1: Material 3 is established.
AC2: Light and dark themes are supported.
AC3: Typography is centralized.
AC4: Spacing and shapes are centralized.
AC5: Reusable UI components are created.
AC6: Components follow accessibility guidelines.
AC7: Compose tests cover important reusable components.

TEST:
- Compose UI tests
- Component rendering tests
- Interaction tests
- Theme-related tests where appropriate
- Existing unit tests
- Build validation

Verify every AC individually.

FINAL RESPONSE:
Implementation Summary
Files Created
Files Modified
UI/Architecture Changes
Tests Executed
AC1–AC7 PASS/FAIL with evidence
Known Limitations
Recommended Next Jira Ticket

Never claim PASS without evidence.
```

---

## JIRA-03 — Authentication & Security

```
You are implementing JIRA-03 — Authentication & Security for Nexus AI.

REQUIRED SKILLS:
android
security
testing

Read the Jira ticket, AC, .kiro/steering/, security skill, Android skill, testing skill,
relevant specs, and existing authentication/security implementation.

Inspect:
- Authentication state
- Token handling
- Secure storage
- DataStore
- Network authentication
- Logout flow
- Session expiration
- Existing error handling
- Logging
- Build configuration
- Existing tests

SECURITY RULES:
- Never hardcode API keys, passwords, JWT secrets, encryption keys, or production credentials.
- Do not log tokens or sensitive authentication data.
- Use secure storage appropriate to the architecture.
- Do not expose raw security exceptions to users.
- Preserve HTTPS requirements for stage/production.
- Follow existing security steering rules.

ACCEPTANCE CRITERIA:
AC1: Authentication state is represented.
AC2: Tokens are stored securely.
AC3: Secrets are not hardcoded.
AC4: Logout clears authenticated state.
AC5: Session expiration is handled.
AC6: Authentication errors are represented consistently.
AC7: Authentication logic has unit tests.

IMPLEMENTATION:
Create or improve the authentication foundation without duplicating existing infrastructure.

TEST:
- Authentication state tests
- Token storage tests
- Logout tests
- Expiration tests
- Error mapping tests
- Security-related unit tests
- Build validation

Verify every AC individually and provide evidence.

Do not weaken tests to make them pass.
```

---

## JIRA-04 — AI Provider Abstraction

```
You are implementing JIRA-04 — AI Provider Abstraction for Nexus AI.

REQUIRED SKILLS:
android
clean-architecture
ai-provider
testing

Read:
- Jira ticket
- Acceptance Criteria
- .kiro/steering/06-ai-architecture.md
- .kiro/skills/ai-provider/SKILL.md
- Clean Architecture skill
- Testing skill
- Relevant AI specs
- Existing repository/provider code

TARGET ARCHITECTURE:

UI
↓
AI Orchestration
↓
AI Provider Interface
↓
Provider Implementation

Provider-specific SDK models must not leak into application/domain APIs.

IMPLEMENT:
Establish provider-independent abstractions for:
- AI request
- AI response
- streaming events
- provider errors
- provider implementations
- fake/test provider

Use the common streaming event model where applicable:
- Started
- Token
- ToolCall
- ToolResult
- Completed
- Error

ACCEPTANCE CRITERIA:
AC1: Provider-independent AI interfaces exist.
AC2: AI request/response models are defined.
AC3: Streaming events are standardized.
AC4: Provider-specific models remain isolated.
AC5: At least one AI provider is integrated.
AC6: Provider errors are mapped consistently.
AC7: A fake provider can be used for tests.

TEST:
- Interface tests
- Mapping tests
- Streaming tests
- Error tests
- Fake provider tests
- Provider integration tests using controlled/fake data where possible
- Build validation

Do not place provider SDK dependencies directly in feature UI.

Verify AC1–AC7 with evidence.
```

---

## JIRA-05 — Chat

```
You are implementing JIRA-05 — Chat for Nexus AI.

REQUIRED SKILLS:
android
compose
ai-provider
testing

Read the repository, Jira ticket, AC, Chat specs, AI Provider skill, Compose skill,
architecture steering, and existing AI implementation.

TARGET FLOW:

Composable
→ Event
→ ViewModel
→ UseCase
→ Repository/AI Orchestration
→ AI Provider

The UI must not directly call network APIs or provider SDKs.

IMPLEMENT:
- User message submission
- AI response display
- Streaming response handling
- Loading state
- Error state
- Appropriate UI state/event handling
- Existing design-system components reuse

ACCEPTANCE CRITERIA:
AC1: User can send a message.
AC2: AI response is displayed.
AC3: Streaming responses are supported where available.
AC4: Loading state is displayed.
AC5: Error state is displayed.
AC6: UI does not directly call network/provider APIs.
AC7: Chat ViewModel/use cases are tested.

TEST:
- ViewModel tests
- Use case tests
- Streaming event tests
- Loading/error/success state tests
- Compose UI tests
- Fake AI provider tests

Verify every AC individually.
Do not claim successful streaming without test evidence.
```

---

## JIRA-06 — Voice

```
You are implementing JIRA-06 — Voice for Nexus AI.

REQUIRED SKILLS:
android
compose
testing

First inspect:
- Existing Chat implementation
- Android permission handling
- Lifecycle architecture
- Compose UI
- ViewModels
- Speech-to-text dependencies if already present
- Existing reusable components
- Voice specs

IMPLEMENT:
Create a lifecycle-safe voice input flow:

Voice UI
→ Permission
→ Speech-to-Text
→ Text Result
→ Chat submission

ACCEPTANCE CRITERIA:
AC1: Voice input can start and stop.
AC2: Microphone permission is handled.
AC3: Speech-to-text result is captured.
AC4: Voice input can be submitted to Chat.
AC5: Lifecycle changes are handled safely.
AC6: Permission and failure states are handled.

RULES:
- Do not leak microphone resources.
- Handle lifecycle cancellation.
- Do not place business logic inside composables.
- Reuse the Chat flow rather than creating a second AI submission path.
- Handle permission denial explicitly.

TEST:
- Permission state tests
- Start/stop tests
- Lifecycle tests
- Failure tests
- ViewModel/use-case tests
- Compose tests where appropriate

Verify AC1–AC6 individually.
```

---

## JIRA-07 — Code Assistant

```
You are implementing JIRA-07 — Code Assistant for Nexus AI.

REQUIRED SKILLS:
android
compose
ai-provider
testing

Inspect:
- Existing Chat/AI architecture
- AI provider abstraction
- Design system
- Code Assistant spec
- Existing reusable editor/text components
- ViewModels/use cases
- Tests

IMPLEMENT:
Support code-focused AI operations:
- Submit code
- Explain code
- Debug code
- Improve code
- Preserve formatting
- Handle long-running operations
- Handle errors

Use the existing AI provider abstraction.

ACCEPTANCE CRITERIA:
AC1: User can submit code.
AC2: Explain-code operation is supported.
AC3: Debug/improve operations are supported.
AC4: Code formatting is preserved.
AC5: AI provider remains abstracted.
AC6: Long-running/error states are handled.
AC7: Business logic is tested.

RULES:
- Do not put AI/provider logic in Compose.
- Reuse existing AI request/response abstractions.
- Avoid duplicating Chat infrastructure unnecessarily.
- Preserve code text exactly where appropriate.
- Do not log sensitive source code unnecessarily.

TEST:
- Use cases
- ViewModel
- Request construction
- Formatting preservation
- Loading/error
- Fake provider
- Compose interaction where appropriate

Verify all ACs with evidence.
```

---

## JIRA-08 — Image Generation

```
You are implementing JIRA-08 — Image Generation for Nexus AI.

REQUIRED SKILLS:
android
compose
ai-provider
testing

Inspect existing:
- AI provider abstraction
- Image Generation spec
- Design system
- UI state patterns
- Network/data abstractions
- Existing tests

IMPLEMENT:
Create an image-generation flow using the provider abstraction.

ACCEPTANCE CRITERIA:
AC1: User can submit an image prompt.
AC2: Image-generation provider is abstracted.
AC3: Loading state is supported.
AC4: Generated image is displayed.
AC5: Failure state is supported.
AC6: Provider implementation remains isolated.

RULES:
- Do not expose provider SDK models to UI/domain.
- Reuse existing AI abstractions where appropriate.
- Keep image-generation provider code isolated.
- Handle loading, success, and failure explicitly.
- Do not store secrets in source code.

TEST:
- Prompt/request tests
- Provider mapping tests
- Loading/success/error tests
- ViewModel/use-case tests
- Compose tests where applicable

Verify AC1–AC6 individually.
```

---

## JIRA-09 — Document Processing

```
You are implementing JIRA-09 — Document Processing for Nexus AI.

REQUIRED SKILLS:
android
clean-architecture
testing
security

Inspect:
- Document specification
- Existing storage/file handling
- Android file picker
- Background-processing architecture
- RAG requirements
- Security steering
- Existing tests

IMPLEMENT the document pipeline required for Nexus AI:

Document selection
→ Metadata
→ Extraction
→ Normalization
→ Observable processing status

ACCEPTANCE CRITERIA:
AC1: User can select supported documents.
AC2: Document metadata is captured.
AC3: Text can be extracted.
AC4: Extracted content is normalized.
AC5: Processing does not block the UI thread.
AC6: Processing status is observable.
AC7: Extraction failures are handled.
AC8: Tests cover document-processing logic.

RULES:
- Do not perform expensive extraction on the UI thread.
- Preserve document metadata required by downstream RAG.
- Validate supported inputs.
- Handle malformed/unsupported documents safely.
- Avoid sensitive document content in logs.

TEST:
- Metadata tests
- Extraction tests
- Normalization tests
- Failure tests
- Coroutine/background tests
- ViewModel/use-case tests

Verify every AC.
```

---

## JIRA-10 — RAG

```
You are implementing JIRA-10 — RAG for Nexus AI.

REQUIRED SKILLS:
rag
ai-provider
clean-architecture
testing

Read:
- RAG specification
- AI architecture steering
- RAG skill
- Clean Architecture skill
- Document Processing implementation
- Existing AI provider abstraction

Follow this pipeline:

Document
→ Extraction
→ Normalization
→ Chunking
→ Embedding
→ Vector Store
→ Similarity Search
→ Context Assembly
→ AI Provider

Preserve:
- document metadata
- chunk metadata
- source metadata
- page metadata where available

ACCEPTANCE CRITERIA:
AC1: Documents can be chunked.
AC2: Embeddings can be generated.
AC3: Embeddings can be stored.
AC4: Semantic retrieval is supported.
AC5: Top-K results can be returned.
AC6: Retrieved context can be passed to AI.
AC7: Empty retrieval is handled.
AC8: RAG components have unit tests.

RULES:
- Bound context size.
- Do not blindly inject unlimited retrieved content.
- Keep embedding/vector-store implementations replaceable.
- Use deterministic fake embeddings/vector stores for tests.
- Do not tightly couple RAG to Compose.

TEST:
- Chunking
- Embedding
- Storage
- Retrieval
- Top-K
- Context assembly
- Empty retrieval
- Error handling
- Fake vector store tests

Verify AC1–AC8 with evidence.
```

---

## JIRA-11 — MCP

```
You are implementing JIRA-11 — MCP for Nexus AI.

REQUIRED SKILLS:
mcp
clean-architecture
security
testing

Read:
- MCP steering/specification
- MCP skill
- Security skill
- Clean Architecture skill
- Tool architecture
- Existing network/transport abstractions

IMPLEMENT MCP support for:
- Server configuration
- Connection lifecycle
- Capability discovery
- Tool discovery
- Tool metadata
- Tool invocation
- Timeout
- Cancellation
- Error mapping

Transport must remain below the application/domain layer.

ACCEPTANCE CRITERIA:
AC1: MCP server configuration is supported.
AC2: MCP connection lifecycle is handled.
AC3: Server capabilities can be discovered.
AC4: MCP tools can be discovered.
AC5: MCP tools can be represented as domain/application models.
AC6: Tools can be invoked.
AC7: Timeout and cancellation are handled.
AC8: MCP errors are mapped to application errors.
AC9: UI does not directly depend on MCP transport.
AC10: MCP functionality is tested.

SECURITY:
- Validate tool names and inputs.
- Validate required parameters/schema.
- Apply timeouts.
- Never blindly execute arbitrary input.
- Do not expose raw transport exceptions to UI.
- Do not log secrets or sensitive tool arguments.

TEST:
- Configuration
- Lifecycle
- Capability discovery
- Tool discovery
- Invocation
- Timeout
- Cancellation
- Error mapping
- Fake MCP transport/client

Verify all ACs individually.
```

---

## JIRA-12 — Agent Architecture

```
You are implementing JIRA-12 — Agent Architecture for Nexus AI.

REQUIRED SKILLS:
agent
mcp
ai-provider
clean-architecture
testing

Read:
- Agent specification
- Agent skill
- MCP skill
- AI Provider skill
- Architecture steering
- Existing Tool abstractions
- Existing orchestration code

Follow the agent pipeline:

User Task
→ Task Understanding
→ Context Collection
→ Planning
→ Tool Selection/Execution
→ Observation
→ Next Step
→ Completion

IMPLEMENT:
- Agent abstraction
- Structured task model
- Agent context
- Planning
- Tool selection
- Multi-step execution
- Cancellation
- Maximum execution limits
- Tool failure recovery

ACCEPTANCE CRITERIA:
AC1: Agent abstraction is established.
AC2: Structured tasks can be represented.
AC3: Agent context can be maintained.
AC4: Agent can generate a plan.
AC5: Agent can select application tools.
AC6: Multi-step execution is supported.
AC7: Cancellation is supported.
AC8: Maximum execution limits exist.
AC9: Tool failures are handled.
AC10: Agent is independent from Compose UI.
AC11: Agent logic is testable.

RULES:
- No uncontrolled recursion.
- Enforce step/time limits.
- Validate tools before execution.
- Support cancellation.
- Keep agent logic independent from Compose.
- Use application-level Tool interfaces.

TEST:
- Task understanding
- Planning
- Tool selection
- Multi-step execution
- Cancellation
- Limits
- Tool failure
- Fake tools
- Fake AI provider

Verify AC1–AC11 with evidence.
```

---

## JIRA-13 — Tool Execution

```
You are implementing JIRA-13 — Tool Execution for Nexus AI.

REQUIRED SKILLS:
mcp
agent
security
testing

Inspect:
- MCP implementation
- Agent architecture
- Existing provider abstraction
- Tool specifications
- Security steering
- Existing tests

IMPLEMENT a common application-level Tool architecture.

Required concepts:
- Tool
- ToolInput
- ToolOutput
- Tool metadata
- Tool validation
- Tool execution
- Tool Registry
- Standardized Tool errors

MCP tools must be converted into application-level tools.

ACCEPTANCE CRITERIA:
AC1: Common Tool interface exists.
AC2: Tool input/output models are standardized.
AC3: MCP tools can be represented as application tools.
AC4: Tool input is validated.
AC5: Tool execution supports timeout.
AC6: Tool errors are standardized.
AC7: Tool Registry exists.
AC8: Agent can execute registered tools.
AC9: Tool execution is tested.

SECURITY:
- Validate names and schemas.
- Validate required parameters.
- Enforce timeout.
- Never blindly execute arbitrary tool input.
- Do not expose secrets.
- Standardize failures.

TEST:
- Tool models
- Validation
- Registry
- MCP conversion
- Timeout
- Error mapping
- Agent execution
- Fake tools

Verify every AC individually.
```

---

## JIRA-14 — Memory & Conversation

```
You are implementing JIRA-14 — Memory & Conversation for Nexus AI.

REQUIRED SKILLS:
android
clean-architecture
testing

Inspect:
- Existing Room/database setup
- DataStore
- Chat models
- Conversation UI
- Repository interfaces
- Existing migrations
- Memory specification

IMPLEMENT:
- Conversation persistence
- Message persistence
- Conversation history
- Rename
- Delete
- Asynchronous Room operations
- Database migration support

ACCEPTANCE CRITERIA:
AC1: Conversations can be persisted.
AC2: Messages can be persisted.
AC3: Conversation history can be loaded.
AC4: Conversations can be renamed.
AC5: Conversations can be deleted.
AC6: Room operations are asynchronous.
AC7: Database migrations are handled.
AC8: Memory functionality is tested.

RULES:
- Keep Room implementation in Data.
- Expose repository interfaces through Domain.
- Do not perform blocking database operations.
- Preserve existing data during migrations.
- Do not log full conversation content unnecessarily.

TEST:
- DAO tests
- Repository tests
- Use-case tests
- Migration tests
- Rename/delete tests
- History loading tests
- Coroutine/Flow tests

Verify AC1–AC8.
```

---

## JIRA-15 — AI Orchestration

```
You are implementing JIRA-15 — AI Orchestration for Nexus AI.

REQUIRED SKILLS:
ai-provider
agent
mcp
rag
clean-architecture
testing

Read:
- AI architecture steering
- AI Provider skill
- Agent skill
- MCP skill
- RAG skill
- Relevant specifications
- Existing Chat, Tool, Memory, and Provider implementations

TARGET:

UI
↓
AI Orchestration
├── AI Provider
├── RAG
├── Agent
├── MCP
├── Tools
└── Conversation Context

IMPLEMENT:
Create an orchestration layer that coordinates AI capabilities without Compose dependencies.

ACCEPTANCE CRITERIA:
AC1: AI orchestration layer exists.
AC2: Chat requests can be routed.
AC3: RAG requests can be routed.
AC4: Agent requests can be routed.
AC5: MCP/tool execution can be coordinated.
AC6: Conversation context can be supplied.
AC7: AI errors are standardized.
AC8: Orchestration has no Compose dependency.
AC9: End-to-end orchestration scenarios are tested.

RULES:
- Keep orchestration independent of UI.
- Reuse existing abstractions.
- Do not duplicate provider logic.
- Standardize failures.
- Preserve streaming behavior where required.

TEST:
- Chat routing
- RAG routing
- Agent routing
- Tool/MCP coordination
- Context handling
- Error handling
- End-to-end orchestration scenarios

Verify AC1–AC9 individually.
```

---

## JIRA-16 — On-Device AI

```
You are implementing JIRA-16 — On-Device AI for Nexus AI.

REQUIRED SKILLS:
android
ai-provider
security
testing

Inspect:
- AI Provider abstraction
- AI architecture steering
- Android capabilities
- Existing lifecycle architecture
- Existing provider implementations
- Security rules
- On-device AI specification

IMPLEMENT:
Integrate on-device AI through the common AI abstraction where practical.

Required areas:
- Capability detection
- Model initialization
- Lifecycle-safe model management
- Offline inference
- Streaming where available
- Configurable cloud fallback
- Failure handling

ACCEPTANCE CRITERIA:
AC1: On-device AI follows the common AI abstraction.
AC2: Device capability detection exists.
AC3: Model initialization is lifecycle-safe.
AC4: On-device inference does not require network access.
AC5: Streaming is supported where available.
AC6: Cloud fallback can be triggered when configured.
AC7: On-device failures are handled.
AC8: Capability/inference logic is tested.

RULES:
- Do not assume every device supports the model.
- Avoid blocking UI operations.
- Handle model initialization failures.
- Keep fallback explicit/configurable.
- Do not expose sensitive local data unnecessarily.

TEST:
- Capability detection
- Initialization
- Inference
- Failure
- Cancellation/lifecycle
- Fallback
- Fake on-device provider

Verify all ACs.
```

---

## JIRA-17 — Background Processing

```
You are implementing JIRA-17 — Background Processing for Nexus AI.

REQUIRED SKILLS:
android
testing

Inspect:
- Existing WorkManager configuration
- Document Processing
- AI jobs
- Application lifecycle
- Repository/data operations
- Existing background tasks
- Testing infrastructure

IMPLEMENT appropriate WorkManager-based background processing.

Use it for work that should survive UI recreation or continue beyond the immediate UI lifecycle.

ACCEPTANCE CRITERIA:
AC1: WorkManager is used for appropriate long-running work.
AC2: Document processing can continue outside UI lifecycle.
AC3: Appropriate AI/background jobs can survive UI recreation.
AC4: Retry policy is defined.
AC5: Job status is observable.
AC6: Failures are represented.
AC7: UI remains responsive.

RULES:
- Do not use background workers unnecessarily.
- Define appropriate retry behavior.
- Make work idempotent where practical.
- Expose observable job state.
- Do not block UI threads.

TEST:
- Worker tests
- Retry behavior
- Failure behavior
- Status updates
- Lifecycle/recreation behavior
- Existing application tests

Verify AC1–AC7.
```

---

## JIRA-18 — Observability

```
You are implementing JIRA-18 — Observability for Nexus AI.

REQUIRED SKILLS:
android
security
testing

Inspect:
- Existing logging
- Error handling
- AI Provider
- MCP
- Tool execution
- Agent execution
- Performance-sensitive operations
- Monitoring configuration
- Security steering

IMPLEMENT standardized observability for Nexus AI.

Required areas:
- Logging
- Error diagnostics
- AI operation diagnostics
- MCP/tool diagnostics
- Performance metrics
- Crash/error monitoring integration where selected
- Production restrictions

ACCEPTANCE CRITERIA:
AC1: Application logging is standardized.
AC2: Sensitive information is excluded from logs.
AC3: AI operations can be diagnosed.
AC4: MCP/tool operations can be diagnosed.
AC5: Important performance metrics can be captured.
AC6: Errors/crashes can be integrated with the selected monitoring solution.
AC7: Production logging is appropriately restricted.

SECURITY:
Never log:
- API keys
- passwords
- access tokens
- JWTs
- sensitive document content
- unnecessary conversation content
- sensitive tool inputs

TEST:
- Logging behavior
- Sensitive-data filtering
- Error reporting
- Metric generation
- Production restrictions

Verify every AC.
```

---

## JIRA-19 — Testing & CI/CD

```
You are implementing JIRA-19 — Testing & CI/CD for Nexus AI.

REQUIRED SKILLS:
android
testing
security

Inspect:
- Existing tests
- Gradle configuration
- GitHub Actions
- Lint/static analysis
- Build variants
- Secret/configuration handling
- Compose tests
- Current CI workflows

IMPLEMENT a CI/CD validation pipeline appropriate for Nexus AI.

ACCEPTANCE CRITERIA:
AC1: Unit tests execute in CI.
AC2: Android build executes in CI.
AC3: Static analysis/lint executes in CI.
AC4: Critical test failures fail the pipeline.
AC5: Secrets are externalized.
AC6: Debug build validation exists.
AC7: Relevant Compose tests are included.
AC8: CI results are visible.

RULES:
- Never commit production secrets.
- Do not weaken tests to make CI pass.
- Keep CI deterministic where possible.
- Reuse existing Gradle tasks.
- Ensure failures are visible and actionable.

IMPLEMENT:
- GitHub Actions workflow(s)
- Test execution
- Build validation
- Lint/static analysis
- Secret handling
- Compose test execution where appropriate

TEST:
Run the same critical commands locally where possible.

Verify AC1–AC8 with concrete CI/build evidence.
```

---

## JIRA-20 — Production & Deployment

```
You are implementing JIRA-20 — Production & Deployment for Nexus AI.

REQUIRED SKILLS:
android
security
testing

Inspect:
- Existing Gradle configuration
- Build variants
- Debug/release configuration
- R8/ProGuard
- Environment configuration
- API endpoints
- Secret handling
- CI/CD
- Existing deployment documentation
- Architecture documentation

IMPLEMENT production-readiness configuration without changing unrelated application behavior.

ENVIRONMENTS:
- Local
- Stage
- Production

ACCEPTANCE CRITERIA:
AC1: Local configuration is supported.
AC2: Stage configuration is supported.
AC3: Production configuration is supported.
AC4: Environment-specific endpoints are isolated.
AC5: Secrets are externalized.
AC6: Release configuration is reviewed.
AC7: R8/ProGuard configuration is reviewed.
AC8: Release build succeeds.
AC9: Critical application flows are validated.
AC10: Production deployment documentation exists.
AC11: Architecture documentation is updated.

SECURITY:
- No production credentials in source control.
- No API keys hardcoded.
- Use appropriate environment/configuration mechanisms.
- Review exported Android components.
- Review backup/security configuration where applicable.
- Review network security.
- Ensure production logging restrictions.

BUILD VALIDATION:
- Debug build
- Release build
- Unit tests
- Compose tests where applicable
- Lint/static analysis
- R8/ProGuard validation

DOCUMENTATION:
Update only documentation that reflects the actual implemented architecture and deployment process.

Verify AC1–AC11 individually.

FINAL RESPONSE:
- Implementation Summary
- Files Created
- Files Modified
- Environment Configuration
- Security Changes
- Build Results
- Tests Executed
- AC1–AC11 PASS/FAIL with evidence
- Known Limitations
- Recommended Next Step

Never claim release readiness without actual build/test evidence.
```

---

## AI ROADMAP MODULE INDEX

The following 20 sections cover the AI Roadmap workstream (NAI-AI-01 – NAI-AI-20).
Each prompt targets the Python backend, model-serving infrastructure, or learning experiments.
Android architecture steering rules do not govern these workstreams.
Security standards (`05-security-standards.md`) apply to all workstreams.

| Section | Ticket | Prompt File |
|---|---|---|
| NAI-AI-01 | GenAI Foundations | `kiro-prompts/NAI-AI-01-genai-foundations.md` |
| NAI-AI-02 | Transformer Architecture & Embeddings | `kiro-prompts/NAI-AI-02-transformers-embeddings.md` |
| NAI-AI-03 | Working with LLMs | `kiro-prompts/NAI-AI-03-working-with-llms.md` |
| NAI-AI-04 | Prompt Engineering | `kiro-prompts/NAI-AI-04-prompt-engineering.md` |
| NAI-AI-05 | Self-Hosted LLMs with Ollama | `kiro-prompts/NAI-AI-05-ollama.md` |
| NAI-AI-06 | LLM as a Service with Flask | `kiro-prompts/NAI-AI-06-flask-llm-service.md` |
| NAI-AI-07 | LLM Service Testing | `kiro-prompts/NAI-AI-07-llm-service-testing.md` |
| NAI-AI-08 | Building LLM Services with FastAPI | `kiro-prompts/NAI-AI-08-fastapi-llm-service.md` |
| NAI-AI-09 | Hugging Face & Open-Source LLMs | `kiro-prompts/NAI-AI-09-huggingface.md` |
| NAI-AI-10 | Advanced LLM Features | `kiro-prompts/NAI-AI-10-advanced-llm-features.md` |
| NAI-AI-11 | Agentic AI | `kiro-prompts/NAI-AI-11-agentic-ai.md` |
| NAI-AI-12 | LangChain | `kiro-prompts/NAI-AI-12-langchain.md` |
| NAI-AI-13 | Memory in AI Agents | `kiro-prompts/NAI-AI-13-agent-memory.md` |
| NAI-AI-14 | Retrieval-Augmented Generation | `kiro-prompts/NAI-AI-14-rag.md` |
| NAI-AI-15 | Vector Databases | `kiro-prompts/NAI-AI-15-vector-databases.md` |
| NAI-AI-16 | LangGraph | `kiro-prompts/NAI-AI-16-langgraph.md` |
| NAI-AI-17 | Model Context Protocol | `kiro-prompts/NAI-AI-17-mcp.md` |
| NAI-AI-18 | Multi-Agent Systems | `kiro-prompts/NAI-AI-18-multi-agent.md` |
| NAI-AI-19 | AI Evaluation | `kiro-prompts/NAI-AI-19-ai-evaluation.md` |
| NAI-AI-20 | Deployment & Production AI Applications | `kiro-prompts/NAI-AI-20-production-deployment.md` |

---

## AI ROADMAP COMMON EXECUTION RULES

Apply these rules to **every** NAI-AI-XX prompt in addition to the Android common rules above:

1. This is a Python backend / infrastructure / learning workstream — Kotlin/Android rules do not apply.
2. Read `.kiro/skills/<relevant-skill>/SKILL.md` before implementation.
3. Never hardcode API keys, tokens, or credentials — load from environment or Secret Manager.
4. Provider SDK types stay inside adapter/infrastructure files — never in domain or API layers.
5. All automated tests use fake/stub providers — no real LLM API calls in the test suite.
6. Model weight files (`*.bin`, `*.safetensors`, `*.pt`, `*.gguf`) are never committed to Git.
7. Add or verify model weight patterns in `.gitignore` before loading any model.
8. Document model licenses before integrating any Hugging Face or open-source model.
9. Context size passed to an LLM is always bounded — never pass unbounded text.
10. Validate all tool inputs against their schema before execution.
11. Enforce step/round limits on all agent and graph executions — no unbounded loops.
12. Structured errors: `{"error": {"code": "...", "message": "..."}}` for Flask; `{"detail": {"code": "...", "message": "..."}}` for FastAPI.
13. Evaluation datasets are versioned — never overwrite existing dataset files.
14. Judge LLMs use `temperature=0` for reproducible evaluation results.
15. Security standards (`05-security-standards.md`) apply to all workstreams.
16. Verify every Acceptance Criterion separately with evidence before marking PASS.
17. An AC may only be marked `PASS` when implementation/test/build evidence exists.

---

## NAI-AI-01 — GenAI Foundations

```
You are implementing NAI-AI-01 — GenAI Foundations for the Nexus AI AI Roadmap workstream.

TICKET KEY: NAI-AI-01
EPIC: AI Roadmap — Full Stack Agentic AI

JIRA TITLE:
GenAI Foundations

DESCRIPTION:
Establish GenAI foundations: integrate a hosted LLM inference API with configurable generation
parameters (temperature and max output tokens). Build a working text-generation service that
demonstrates the core LLM request-response cycle.

REQUIRED SKILLS:
fastapi
evaluation

Before starting:
1. Read this ticket and all ACs below.
2. Read .kiro/steering/ — all files.
3. Read .kiro/skills/fastapi/SKILL.md and .kiro/skills/evaluation/SKILL.md.
4. Inspect backend/fastapi-service/ and learning/01-genai/ for existing work.
5. Reuse existing infrastructure — do not duplicate.

ARCHITECTURE RULES:
- API keys from environment via pydantic-settings — never hardcoded.
- Provider SDK calls isolated behind a service layer.
- Structured errors: {"error": {"code": "...", "message": "..."}}.
- This is a Python backend workstream — Android architecture rules do not apply.

IMPLEMENTATION:
Create a text-generation service with at least one hosted provider behind a service layer.
Expose a generation endpoint accepting prompt + temperature + max_tokens.
Validate that out-of-range temperature or negative max_tokens are rejected.
Provide .env.example with all required keys documented.

ACCEPTANCE CRITERIA:
AC1: Hosted LLM API is called successfully.
AC2: Temperature is configurable and applied to the request.
AC3: max_tokens is configurable and applied to the request.
AC4: Invalid parameter values are rejected with a structured error.
AC5: API key is loaded from environment — never hardcoded.
AC6: A working text-generation endpoint or script produces non-empty output.
AC7: Unit tests cover generation service logic using a fake provider.

WORKFLOW:
1. Inspect backend/ and learning/01-genai/ for existing work.
2. Read fastapi skill and 05-security-standards.md.
3. Create implementation plan.
4. Implement generation service with parameter validation.
5. Write unit tests with fake provider.
6. Run: pytest learning/01-genai/ or pytest backend/fastapi-service/
7. Verify every AC individually with evidence.

FINAL RESPONSE:
- Implementation Summary
- Files Created / Modified
- Tests Executed and Results
- AC1–AC7 PASS/FAIL with evidence
- Known Limitations
- Recommended Next Ticket: NAI-AI-02

Never claim an AC is PASS without evidence.
```

---

## NAI-AI-02 — Transformer Architecture & Embeddings

```
You are implementing NAI-AI-02 — Transformer Architecture & Embeddings for the Nexus AI roadmap.

TICKET KEY: NAI-AI-02
EPIC: AI Roadmap — Full Stack Agentic AI

JIRA TITLE:
Transformer Architecture & Embeddings

REQUIRED SKILLS:
huggingface
evaluation

Before starting:
1. Read this ticket, all ACs, .kiro/steering/, .kiro/skills/huggingface/SKILL.md,
   .kiro/skills/evaluation/SKILL.md.
2. Inspect learning/02-transformers-embeddings/ for existing work.
3. Review model card license before selecting any model.
4. Confirm *.bin, *.safetensors, *.pt are in .gitignore.

IMPLEMENT:
1. Tokenization experiment: load tokenizer, encode sentences, report token IDs,
   count, and vocabulary size.
2. Embedding generation: use sentence-transformers/all-MiniLM-L6-v2; report dimension.
3. Semantic similarity: compute cosine similarity for at least 3 sentence pairs;
   similar pairs must score measurably higher.
4. README: document models, licenses, embedding dimensions, and key findings.

ACCEPTANCE CRITERIA:
AC1: Tokenizer loads and encodes sample text without error.
AC2: Token counts and vocabulary size are reported.
AC3: Text embeddings are generated with a consistent documented dimension.
AC4: Semantically similar pairs score measurably higher than unrelated pairs.
AC5: Embedding dimension is documented for each model used.
AC6: Model licenses are documented.
AC7: Unit tests cover cosine similarity with known vector inputs.

WORKFLOW:
1. Check learning/02-transformers-embeddings/ for existing work.
2. Implement tokenization experiment.
3. Implement embedding generation.
4. Implement cosine similarity comparisons.
5. Write README with results.
6. Write unit tests.
7. Run: pytest learning/02-transformers-embeddings/tests/
8. Verify every AC individually.

FINAL RESPONSE:
- Implementation Summary
- Files Created / Modified
- Similarity Comparison Results table
- Tests Executed and Results
- AC1–AC7 PASS/FAIL with evidence
- Known Limitations
- Recommended Next Ticket: NAI-AI-03

Never claim an AC is PASS without evidence.
```

---

## NAI-AI-03 — Working with LLMs

```
You are implementing NAI-AI-03 — Working with LLMs for the Nexus AI roadmap.

TICKET KEY: NAI-AI-03
EPIC: AI Roadmap — Full Stack Agentic AI

JIRA TITLE:
Working with LLMs

REQUIRED SKILLS:
fastapi
evaluation

Before starting:
1. Read this ticket, all ACs, .kiro/steering/, .kiro/skills/fastapi/SKILL.md.
2. Inspect NAI-AI-01 implementation for existing provider call patterns.
3. Inspect backend/fastapi-service/ structure.
4. Do not re-implement infrastructure already complete in NAI-AI-01.

ARCHITECTURE RULES:
- LLMProvider is a Protocol in domain — no SDK imports.
- Provider adapters live in infrastructure/providers/.
- Callers inject provider via FastAPI Depends.
- Streaming uses AsyncIterator[StreamChunk].
- Provider errors caught inside adapter and mapped to domain error types.
- Active provider selected from environment config.

IMPLEMENT:
1. LLMProvider Protocol (domain/interfaces/llm_provider.py):
   complete(request) -> ChatResponse
   stream(request) -> AsyncIterator[StreamChunk]
2. OpenAIProvider and GeminiProvider adapters in infrastructure/providers/.
3. Provider factory wired from settings.default_provider.
4. POST /v1/chat/completions: non-streaming and streaming.
5. FakeLLMProvider in tests/fakes/ with deterministic responses.

ACCEPTANCE CRITERIA:
AC1: LLMProvider protocol defined in domain with no SDK imports.
AC2: At least two provider adapters implemented.
AC3: Active provider switchable via environment variable.
AC4: Streaming works end-to-end, tokens emitted incrementally.
AC5: Provider SDK types do not appear outside adapter files.
AC6: Provider errors mapped to domain error types inside adapters.
AC7: FakeLLMProvider returns deterministic responses.
AC8: Unit tests cover provider switching, streaming, and error mapping.

WORKFLOW:
1. Inspect existing provider code from NAI-AI-01.
2. Define LLMProvider protocol and domain models.
3. Implement OpenAI and Gemini adapters.
4. Wire provider factory.
5. Update FastAPI endpoint.
6. Create FakeLLMProvider.
7. Write tests.
8. Run: pytest backend/fastapi-service/tests/
9. Verify every AC individually.

FINAL RESPONSE:
- Implementation Summary
- Files Created / Modified
- Provider Switching Evidence
- Tests Executed and Results
- AC1–AC8 PASS/FAIL with evidence
- Known Limitations
- Recommended Next Ticket: NAI-AI-04

Never claim an AC is PASS without evidence.
```

---

## NAI-AI-04 — Prompt Engineering

```
You are implementing NAI-AI-04 — Prompt Engineering for the Nexus AI roadmap.

TICKET KEY: NAI-AI-04
EPIC: AI Roadmap — Full Stack Agentic AI

JIRA TITLE:
Prompt Engineering

REQUIRED SKILLS:
fastapi
evaluation

Before starting:
1. Read this ticket, all ACs, .kiro/steering/, .kiro/skills/fastapi/SKILL.md,
   05-security-standards.md AI section on prompt injection.
2. Inspect NAI-AI-03 LLMProvider interface.
3. Inspect learning/04-prompt-engineering/ for existing work.

ARCHITECTURE RULES:
- Prompt templates in application/prompts/ as versioned files/constants — not inline.
- Template variables explicitly named and documented.
- Output validation uses JSON schema or Pydantic — not regex on freeform output.
- Input size limits enforced before LLM is called.
- Sanitize user-supplied content in templates.

IMPLEMENT:
1. PromptTemplate class: template ID, version, render(variables) -> str, size validation.
2. Three strategies in learning/04-prompt-engineering/:
   zero-shot, few-shot (2–3 examples), role-based.
3. Structured output: prompt instructs model to return JSON; validate against Pydantic model.
4. Workbench script: iterate test cases, record rendered prompt/response/validation result.

ACCEPTANCE CRITERIA:
AC1: Reusable templates defined with named input variables.
AC2: Three strategies implemented: zero-shot, few-shot, role-based.
AC3: Structured output prompts produce JSON validated against a Pydantic model.
AC4: Oversized inputs rejected with structured error before LLM is called.
AC5: Invalid model responses rejected by output validation.
AC6: Templates are versioned.
AC7: Tests cover rendering, size rejection, and output validation (valid + invalid cases).

WORKFLOW:
1. Read 05-security-standards.md AI section.
2. Implement PromptTemplate.
3. Implement three prompt strategies.
4. Implement structured output validation.
5. Build workbench script.
6. Write tests.
7. Run: pytest learning/04-prompt-engineering/tests/
8. Verify every AC individually.

FINAL RESPONSE:
- Implementation Summary
- Files Created / Modified
- One example output per prompt strategy
- Validation evidence (pass case + rejection case)
- Tests Executed and Results
- AC1–AC7 PASS/FAIL with evidence
- Known Limitations
- Recommended Next Ticket: NAI-AI-05

Never claim an AC is PASS without evidence.
```

---

## NAI-AI-05 — Self-Hosted LLMs with Ollama

```
You are implementing NAI-AI-05 — Self-Hosted LLMs with Ollama for the Nexus AI roadmap.

TICKET KEY: NAI-AI-05
EPIC: AI Roadmap — Full Stack Agentic AI

JIRA TITLE:
Self-Hosted LLMs with Ollama

REQUIRED SKILLS:
fastapi
evaluation

Before starting:
1. Read this ticket, all ACs, .kiro/steering/, .kiro/skills/fastapi/SKILL.md.
2. Inspect NAI-AI-03 LLMProvider protocol and existing adapters.
3. Verify Ollama is installed: ollama --version.
4. Confirm OLLAMA_BASE_URL is in settings.

ARCHITECTURE RULES:
- OllamaProvider in infrastructure/providers/ollama_provider.py.
- Implements LLMProvider — no Ollama-specific types outside the file.
- Uses httpx.AsyncClient — not Ollama Python SDK if it conflicts with async pattern.
- OLLAMA_BASE_URL defaults to http://localhost:11434; overridable from environment.
- Health: GET /api/tags; success = Ollama ready.
- Latency logged at DEBUG level per request.

IMPLEMENT:
1. Document Ollama setup in learning/05-ollama/README.md.
2. OllamaProvider using httpx.AsyncClient:
   POST /api/chat for non-streaming.
   POST /api/chat with "stream": true for streaming.
3. Model selection via ChatRequest.model; default from OLLAMA_DEFAULT_MODEL.
4. Latency measurement per complete() and stream() call.
5. ping() -> bool method wired into /readiness.
6. Fake Ollama HTTP client for tests.

ACCEPTANCE CRITERIA:
AC1: Ollama health check returns 200.
AC2: At least one model pulled and selectable by name.
AC3: OllamaProvider implements LLMProvider protocol.
AC4: Non-streaming completion returns valid ChatResponse.
AC5: Streaming emits StreamChunk tokens incrementally.
AC6: Ollama types do not appear outside ollama_provider.py.
AC7: Latency measured and logged per request.
AC8: Tests cover adapter using fake Ollama HTTP client.

WORKFLOW:
1. Verify Ollama installed and pull a model.
2. Read LLMProvider protocol.
3. Implement OllamaProvider.
4. Add latency logging.
5. Wire into provider factory.
6. Create fake HTTP client.
7. Write tests.
8. Run: pytest backend/fastapi-service/tests/
9. Verify every AC individually.

FINAL RESPONSE:
- Implementation Summary
- Files Created / Modified
- Ollama Setup Steps
- Latency Log Evidence
- Tests Executed and Results
- AC1–AC8 PASS/FAIL with evidence
- Known Limitations
- Recommended Next Ticket: NAI-AI-06

Never claim an AC is PASS without evidence.
```

---

## NAI-AI-06 — LLM as a Service with Flask

```
You are implementing NAI-AI-06 — LLM as a Service with Flask for the Nexus AI roadmap.

TICKET KEY: NAI-AI-06
EPIC: AI Roadmap — Full Stack Agentic AI

JIRA TITLE:
LLM as a Service with Flask

REQUIRED SKILLS:
flask
evaluation

Before starting:
1. Read this ticket, all ACs, .kiro/steering/, .kiro/skills/flask/SKILL.md.
2. Inspect backend/flask-service/ for existing work.
3. This service is independent of FastAPI — shares the provider abstraction concept only.

ARCHITECTURE RULES:
- Application factory: create_app(config_name) -> Flask.
- Routes in Blueprints: chat_bp, models_bp, health_bp.
- Route handlers delegate to service layer — no SDK calls in Blueprint functions.
- Config from os.environ via Config class.
- Errors: {"error": {"code": "...", "message": "..."}}.
- SSE: flask.Response(generator, mimetype="text/event-stream").
- API key auth on all inference endpoints.

IMPLEMENT:
1. backend/flask-service/ structure per flask skill.
2. POST /v1/chat/completions (streaming and non-streaming).
3. GET /v1/models — list model names.
4. GET /health — {"status": "ok"}, no auth.
5. Request validation: missing messages -> 400; temperature range check.
6. FakeProvider in tests/fakes/ for all tests.

ACCEPTANCE CRITERIA:
AC1: App uses create_app() factory.
AC2: POST /v1/chat/completions returns valid JSON.
AC3: GET /v1/models returns list of model names.
AC4: GET /health returns 200 — no auth required.
AC5: Missing messages returns 400 with structured error.
AC6: Provider wired through service layer.
AC7: API key auth enforced on inference endpoints.
AC8: SSE streaming supported when "stream": true.
AC9: All tests use FakeProvider.

WORKFLOW:
1. Read flask skill — factory pattern and SSE rules.
2. Create application structure.
3. Implement Blueprints, service, errors.
4. Implement validation and auth.
5. Implement SSE streaming.
6. Create FakeProvider.
7. Write tests.
8. Run: pytest backend/flask-service/tests/
9. Verify every AC individually.

FINAL RESPONSE:
- Implementation Summary
- Files Created / Modified
- Endpoint Smoke Test Evidence per endpoint
- Tests Executed and Results
- AC1–AC9 PASS/FAIL with evidence
- Known Limitations
- Recommended Next Ticket: NAI-AI-07

Never claim an AC is PASS without evidence.
```

---

## NAI-AI-07 — LLM Service Testing

```
You are implementing NAI-AI-07 — LLM Service Testing for the Nexus AI roadmap.

TICKET KEY: NAI-AI-07
EPIC: AI Roadmap — Full Stack Agentic AI

JIRA TITLE:
LLM Service Testing

REQUIRED SKILLS:
flask
fastapi
evaluation

Before starting:
1. Read this ticket, all ACs, .kiro/steering/.
2. Read .kiro/skills/flask/SKILL.md and .kiro/skills/fastapi/SKILL.md testing sections.
3. Confirm NAI-AI-06 (Flask) and NAI-AI-08 (FastAPI) are implemented.
4. Audit existing tests for gaps before writing new ones.

ARCHITECTURE RULES:
- All automated tests use fake/stub providers — zero real API calls.
- Tests are deterministic.
- Streaming tests collect full SSE event sequence and assert ordering.
- Test report generated via: pytest --tb=short -v > learning/07-service-testing/test-report.txt

IMPLEMENT:
1. For Flask and FastAPI: tests covering happy-path, streaming, missing messages (400/422),
   invalid temperature (400/422), missing API key (401), wrong API key (401),
   provider error propagation, health (200 no auth), models list.
2. SSE parser helper: backend/shared/test_helpers/sse_parser.py
   parse_sse_events(raw: bytes) -> list[dict]
3. Bruno/Postman collection: learning/07-service-testing/nexus-ai-api-collection.json
   with BASE_URL env var; one request per endpoint per service.
4. Run full suite and save report.

ACCEPTANCE CRITERIA:
AC1: Bruno/Postman collection covers all major endpoints for both services.
AC2: Happy-path chat completion tests pass for Flask and FastAPI.
AC3: Missing/malformed request body tests verify 400/422.
AC4: Auth failure tests verify 401 for missing and wrong key.
AC5: Streaming tests collect full SSE sequence and assert token order and [DONE].
AC6: Provider error propagation tested via fake provider returning error.
AC7: All tests use fake/stub providers.
AC8: Test report generated and shows all tests passing.

WORKFLOW:
1. Confirm both services are implemented.
2. Audit existing coverage.
3. Implement SSE parser.
4. Add missing Flask tests.
5. Add missing FastAPI tests.
6. Create Bruno/Postman collection.
7. Run full suite; save report.
8. Verify every AC.

FINAL RESPONSE:
- Implementation Summary
- Files Created / Modified
- Test Suite Summary (pass/fail counts)
- Collection Location
- AC1–AC8 PASS/FAIL with evidence
- Known Limitations
- Recommended Next Ticket: NAI-AI-08 (if not done) or NAI-AI-09

Never claim an AC is PASS without evidence.
```

---

## NAI-AI-08 — Building LLM Services with FastAPI

```
You are implementing NAI-AI-08 — Building LLM Services with FastAPI for the Nexus AI roadmap.

TICKET KEY: NAI-AI-08
EPIC: AI Roadmap — Full Stack Agentic AI

JIRA TITLE:
Building LLM Services with FastAPI

REQUIRED SKILLS:
fastapi
evaluation

Before starting:
1. Read this ticket, all ACs, .kiro/steering/, .kiro/skills/fastapi/SKILL.md.
2. Inspect existing FastAPI service from NAI-AI-01 and NAI-AI-03.
3. Audit which endpoints and models are already implemented.
4. Extend existing code — do not rewrite working modules.

ARCHITECTURE RULES:
- All routes have typed response_model and Pydantic request bodies.
- Services injected via FastAPI Depends.
- Settings from pydantic-settings; no hardcoded values.
- Provider SDK types stay in infrastructure/providers/.
- Streaming: StreamingResponse with media_type="text/event-stream".
- Auth: API key dependency on all inference routes.
- Errors: {"detail": {"code": "...", "message": "..."}}.

IMPLEMENT:
Endpoints:
  POST /v1/chat/completions — non-streaming and streaming
  POST /v1/embeddings — generate embeddings
  GET /v1/models — list configured models
  GET /health — liveness (no auth)
  GET /readiness — readiness with provider health check (no auth)

Components:
  Pydantic models: Message, ChatRequest, ChatResponse, StreamChunk,
                   EmbeddingRequest, EmbeddingResponse, HealthResponse, ReadinessResponse
  Routers: chat.py, embeddings.py, models.py, health.py
  Auth middleware: verify_api_key dependency
  Services: ChatService, EmbeddingService
  Settings: pydantic-settings; .env.example complete

ACCEPTANCE CRITERIA:
AC1: All services injected via Depends.
AC2: All request/response bodies have Pydantic v2 models.
AC3: POST /v1/chat/completions works non-streaming.
AC4: POST /v1/chat/completions works streaming (SSE).
AC5: POST /v1/embeddings returns consistent-dimension float vectors.
AC6: GET /health returns {"status": "ok"} — no auth.
AC7: GET /readiness returns provider health — no auth.
AC8: Missing/invalid API key returns 401.
AC9: OpenAPI docs auto-generated and accurate.
AC10: All settings from pydantic-settings; .env.example complete.
AC11: Integration tests cover all endpoints using FakeLLMProvider.

WORKFLOW:
1. Inspect existing service from NAI-AI-01 and NAI-AI-03.
2. Add missing Pydantic models.
3. Implement missing routers.
4. Implement embeddings endpoint and service.
5. Finalize auth middleware.
6. Write/extend integration tests.
7. Run: pytest backend/fastapi-service/tests/ -v
8. Start service locally; verify /docs.
9. Verify every AC.

FINAL RESPONSE:
- Implementation Summary
- Files Created / Modified
- OpenAPI Endpoint List
- Tests Executed and Results
- AC1–AC11 PASS/FAIL with evidence
- Known Limitations
- Recommended Next Ticket: NAI-AI-09

Never claim an AC is PASS without evidence.
```

---

## NAI-AI-09 — Hugging Face & Open-Source LLMs

```
You are implementing NAI-AI-09 — Hugging Face & Open-Source LLMs for the Nexus AI roadmap.

TICKET KEY: NAI-AI-09
EPIC: AI Roadmap — Full Stack Agentic AI

JIRA TITLE:
Hugging Face & Open-Source LLMs

REQUIRED SKILLS:
huggingface
evaluation

Before starting:
1. Read this ticket, all ACs, .kiro/steering/, .kiro/skills/huggingface/SKILL.md.
2. Inspect NAI-AI-03 LLMProvider protocol.
3. Confirm *.bin, *.safetensors, *.pt in .gitignore.
4. Review model card license on Hugging Face before selecting models.

ARCHITECTURE RULES:
- Model weights never committed to Git.
- HFTextGenerationProvider and HFEmbeddingProvider implement domain protocols.
- transformers.* types do not appear outside backend/model-serving/huggingface/.
- Device placement explicit — specify device or device_map="auto".
- CI tests use FakeHFProvider or hf-internal-testing/tiny-random-gpt2.
- Document licenses and RAM/VRAM requirements.

IMPLEMENT:
1. Select models: one text generation (MIT or apache-2.0 license) + all-MiniLM-L6-v2 embeddings.
2. HFTextGenerationProvider: complete() + stream() via TextIteratorStreamer in thread.
3. HFEmbeddingProvider: embed(texts) -> list[list[float]] with normalize_embeddings=True.
4. Performance benchmark: learning/09-huggingface/benchmark.py
   Compare 5 prompts HF vs OpenAI/Gemini; record latency.
5. FakeHFProvider for unit tests.
6. README: models, licenses, min RAM/VRAM, benchmark results.

ACCEPTANCE CRITERIA:
AC1: Text generation model loads and produces non-empty output.
AC2: Model license documented.
AC3: HFTextGenerationProvider implements LLMProvider protocol.
AC4: HFEmbeddingProvider generates normalized fixed-dimension vectors.
AC5: HFEmbeddingProvider integrates with existing embedding pipeline.
AC6: Model weights not committed (git status evidence).
AC7: RAM/VRAM requirements documented.
AC8: Performance comparison documented.
AC9: Unit tests use FakeHFProvider.

WORKFLOW:
1. Check .gitignore for model weight patterns.
2. Select models; review licenses.
3. Implement HFTextGenerationProvider.
4. Implement HFEmbeddingProvider.
5. Run benchmark.
6. Create FakeHFProvider.
7. Write tests.
8. Run: pytest backend/model-serving/huggingface/tests/ -v
9. Verify every AC.

FINAL RESPONSE:
- Implementation Summary
- Files Created / Modified
- Model Licenses Documented
- Performance Comparison Table
- Git Status Evidence (weights not tracked)
- Tests Executed and Results
- AC1–AC9 PASS/FAIL with evidence
- Known Limitations
- Recommended Next Ticket: NAI-AI-10

Never claim an AC is PASS without evidence.
```

---

## NAI-AI-10 — Advanced LLM Features

```
You are implementing NAI-AI-10 — Advanced LLM Features for the Nexus AI roadmap.

TICKET KEY: NAI-AI-10
EPIC: AI Roadmap — Full Stack Agentic AI

JIRA TITLE:
Advanced LLM Features

REQUIRED SKILLS:
fastapi
evaluation

Before starting:
1. Read this ticket, all ACs, .kiro/steering/, .kiro/skills/fastapi/SKILL.md.
2. Inspect NAI-AI-03 LLMProvider and NAI-AI-08 FastAPI service.
3. Review ChatRequest/ChatResponse models for fields needed.
4. Read 05-security-standards.md — validate AI output before tool execution.

ARCHITECTURE RULES:
- Structured output responses validated against Pydantic model before returning.
- Tool calling: provider adapter handles ToolCall event; caller feeds result back.
- Vision input: content is list[ContentPart] — not a top-level field.
- Unsupported feature returns {"error": {"code": "feature_not_supported"}} — never 500.
- New domain model fields use Optional with None default — no breaking changes.

IMPLEMENT:
1. Structured JSON output: ChatRequest.response_format + json_schema field;
   validate response against json_schema via jsonschema.validate().
2. Streaming: end-to-end integration test collecting all SSE chunks.
3. Tool calling: ChatRequest.tools: list[ToolDefinition];
   return ChatResponse.tool_calls; demo in learning/10-advanced-llm/tool_calling_demo.py.
4. Vision: Message.content: str | list[ContentPart]; ImageUrlPart model;
   demo in learning/10-advanced-llm/vision_demo.py.

ACCEPTANCE CRITERIA:
AC1: Structured JSON output requested and response validated against json_schema.
AC2: Streaming integration test verifies: first token → incremental → [DONE].
AC3: Tool calling round-trip demonstrated: model requests tool → result fed back → final answer.
AC4: Vision input (image URL or base64) processed by capable model.
AC5: Advanced features use LLMProvider interface — no SDK types in route handlers.
AC6: Unsupported feature returns feature_not_supported error — not 500.
AC7: Tests cover structured output, streaming, tool calling, vision.

WORKFLOW:
1. Review existing models and adapters.
2. Extend domain models with optional fields.
3. Implement structured output + validation.
4. Implement streaming integration test.
5. Implement tool calling in domain models and adapters.
6. Implement vision input.
7. Add demos.
8. Write tests.
9. Run: pytest backend/fastapi-service/tests/ -v
10. Verify every AC.

FINAL RESPONSE:
- Implementation Summary
- Files Created / Modified
- Tool Calling Demo Output
- Tests Executed and Results
- AC1–AC7 PASS/FAIL with evidence
- Known Limitations
- Recommended Next Ticket: NAI-AI-11

Never claim an AC is PASS without evidence.
```

---

## NAI-AI-11 — Agentic AI

```
You are implementing NAI-AI-11 — Agentic AI for the Nexus AI roadmap.

TICKET KEY: NAI-AI-11
EPIC: AI Roadmap — Full Stack Agentic AI

JIRA TITLE:
Agentic AI

REQUIRED SKILLS:
fastapi
multi-agent
evaluation

Before starting:
1. Read this ticket, all ACs, .kiro/steering/ (agent architecture section),
   .kiro/skills/fastapi/SKILL.md, .kiro/skills/multi-agent/SKILL.md.
2. Inspect NAI-AI-08 FastAPI service and NAI-AI-10 tool calling.
3. Read 05-security-standards.md — validate all tool inputs.

ARCHITECTURE RULES:
- Agent logic in backend/agent-runtime/agents/ — not inside FastAPI route handlers.
- Agent uses LLMProvider interface — never calls provider SDK directly.
- Tools looked up from ToolRegistry by name.
- All tool inputs validated against JSON schema before execution.
- Max step count enforced unconditionally at construction time.
- asyncio.CancelledError propagated cleanly.
- Execution trace: list[TraceStep] attached to AgentResult.

IMPLEMENT:
1. Domain models: AgentTask, TraceStep, AgentResult, AgentStatus.
2. ToolRegistry: register, get, list_tools.
3. AgentExecutor execution loop:
   Build prompt → Call LLM → Parse thought + tool_call → Validate input → Execute with timeout
   → Record trace → Check step limit → Loop until FINISH.
4. Demo: learning/11-agentic-ai/demo.py with 2–3 simple tools.
5. POST /v1/agents/run endpoint.

ACCEPTANCE CRITERIA:
AC1: Agent produces execution plan before first tool call.
AC2: Agent selects tools from ToolRegistry by name.
AC3: Tool input validated against schema; invalid input rejected.
AC4: Each tool call has asyncio.wait_for timeout.
AC5: Max step count enforced; returns STEP_LIMIT_REACHED.
AC6: CancelledError stops execution cleanly; returns CANCELLED.
AC7: Trace records thought, tool name, input, output per step.
AC8: Demo completes a bounded multi-step goal (≥2 tool calls).
AC9: Agent logic in backend/agent-runtime/ — not in FastAPI handlers.
AC10: Tests cover planning, tool selection, step limit, cancellation, validation failure.

WORKFLOW:
1. Read multi-agent skill safeguards.
2. Define domain models.
3. Implement ToolRegistry.
4. Implement AgentExecutor.
5. Write demo with real or fake tools.
6. Add FastAPI endpoint.
7. Write tests with FakeLLMProvider and FakeTool.
8. Run: pytest backend/agent-runtime/tests/ -v
9. Verify every AC.

FINAL RESPONSE:
- Implementation Summary
- Files Created / Modified
- Demo Execution Trace (condensed)
- Step Limit Test Evidence
- Cancellation Test Evidence
- Tests Executed and Results
- AC1–AC10 PASS/FAIL with evidence
- Known Limitations
- Recommended Next Ticket: NAI-AI-12

Never claim an AC is PASS without evidence.
```

---

## NAI-AI-12 — LangChain

```
You are implementing NAI-AI-12 — LangChain for the Nexus AI roadmap.

TICKET KEY: NAI-AI-12
EPIC: AI Roadmap — Full Stack Agentic AI

JIRA TITLE:
LangChain

REQUIRED SKILLS:
langchain
evaluation

Before starting:
1. Read this ticket, all ACs, .kiro/steering/, .kiro/skills/langchain/SKILL.md.
2. Inspect NAI-AI-08 LLMProvider and NAI-AI-09 embedding provider.
3. Inspect backend/agent-runtime/langchain/ for existing work.

ARCHITECTURE RULES:
- langchain.* types must not appear in domain/, api/, or application/.
- LangChain components wrapped in adapters in agent-runtime/langchain/adapters/.
- Prompt templates in prompts/ files — not inline strings.
- RAG chain uses LCEL: retriever | prompt | llm | output_parser.
- Streaming via astream().
- Document metadata (source, page) preserved through the chain.
- Tests use FakeListChatModel — zero real API calls.

IMPLEMENT:
1. LCEL RAG chain (chains/rag_chain.py): injected llm + retriever.
2. Document QA chain returning answer + source references.
3. @tool search tool.
4. LangChainLLMAdapter: wraps BaseChatModel → LLMProvider protocol.
5. Prompt templates in prompts/: rag_template.txt, qa_template.txt, prompts.py with versions.
6. Demo: learning/12-langchain/rag_demo.py — 3 questions, answers, and sources.

ACCEPTANCE CRITERIA:
AC1: RAG chain built with LCEL.
AC2: langchain.* not in domain/, api/, or application/.
AC3: Prompt templates in files with version constants.
AC4: At least one @tool defined and invocable.
AC5: Document loader preserves source and page metadata.
AC6: Streaming works via astream().
AC7: LangChainLLMAdapter implements LLMProvider protocol.
AC8: Tests use FakeListChatModel.

WORKFLOW:
1. Set up backend/agent-runtime/langchain/ structure.
2. Create prompt template files and loader.
3. Implement LCEL RAG chain with streaming.
4. Implement document QA chain with sources.
5. Implement @tool.
6. Implement LangChainLLMAdapter.
7. Build demo.
8. Write tests.
9. Run: pytest backend/agent-runtime/langchain/tests/ -v
10. Verify every AC.

FINAL RESPONSE:
- Implementation Summary
- Files Created / Modified
- RAG Demo Output (3 Q&A + sources)
- Streaming Evidence
- Tests Executed and Results
- AC1–AC8 PASS/FAIL with evidence
- Known Limitations
- Recommended Next Ticket: NAI-AI-13

Never claim an AC is PASS without evidence.
```

---

## NAI-AI-13 — Memory in AI Agents

```
You are implementing NAI-AI-13 — Memory in AI Agents for the Nexus AI roadmap.

TICKET KEY: NAI-AI-13
EPIC: AI Roadmap — Full Stack Agentic AI

JIRA TITLE:
Memory in AI Agents

REQUIRED SKILLS:
langchain
evaluation

Before starting:
1. Read this ticket, all ACs, .kiro/steering/, .kiro/skills/langchain/SKILL.md.
2. Inspect NAI-AI-11 AgentExecutor and NAI-AI-08 FastAPI service.
3. Read 05-security-standards.md — do not log conversation content.

ARCHITECTURE RULES:
- MemoryRepository interface in domain layer.
- Implementation (SQLite / in-memory) in infrastructure/ — not domain.
- MemoryRepository injected into agent via Depends.
- Session memory keyed by session_id — no cross-session leakage.
- Conversation content never logged at INFO level or above.
- Retention policy enforced on every add_message().

IMPLEMENT:
1. MemoryRepository protocol: add_message, get_history, search_relevant, clear, delete_session.
2. InMemoryStore: dict[session_id, list[Message]]; max_turns retention.
3. SQLiteStore: persistent with migrations; search_relevant via vector store if NAI-AI-15 done.
4. Retention: max_turns configurable from MEMORY_MAX_TURNS env var.
5. FastAPI: GET /v1/memory/{session_id}, DELETE /v1/memory/{session_id}.
6. Demo: learning/13-agent-memory/memory_demo.py

ACCEPTANCE CRITERIA:
AC1: Session memory per session_id — no cross-session leakage.
AC2: Persistence survives process restart (SQLite).
AC3: get_history() returns chronological order.
AC4: search_relevant() returns semantically related messages.
AC5: clear() removes all messages; subsequent get_history() returns empty.
AC6: Retention deletes oldest when max_turns exceeded.
AC7: MemoryRepository is domain interface; implementation in infrastructure/.
AC8: Tests cover add, get_history, search_relevant, clear, retention boundary.

WORKFLOW:
1. Define MemoryRepository protocol.
2. Implement InMemoryStore with retention.
3. Implement SQLiteStore with persistence.
4. Add FastAPI endpoints.
5. Build demo.
6. Write tests.
7. Run: pytest backend/agent-runtime/memory/tests/ -v
8. Verify every AC.

FINAL RESPONSE:
- Implementation Summary
- Files Created / Modified
- Retention Policy Evidence
- Persistence Evidence (write → restart → read)
- Tests Executed and Results
- AC1–AC8 PASS/FAIL with evidence
- Known Limitations
- Recommended Next Ticket: NAI-AI-14

Never claim an AC is PASS without evidence.
```

---

## NAI-AI-14 — Retrieval-Augmented Generation

```
You are implementing NAI-AI-14 — Retrieval-Augmented Generation for the Nexus AI roadmap.

TICKET KEY: NAI-AI-14
EPIC: AI Roadmap — Full Stack Agentic AI

JIRA TITLE:
Retrieval-Augmented Generation

REQUIRED SKILLS:
langchain
vector-db
evaluation

Before starting:
1. Read this ticket, all ACs, .kiro/steering/ (RAG Pipeline section),
   .kiro/skills/langchain/SKILL.md, .kiro/skills/vector-db/SKILL.md.
2. Confirm NAI-AI-15 VectorStore interface is implemented.
3. Inspect backend/rag/ for existing pipeline stages.

ARCHITECTURE RULES:
- Each pipeline stage in a separate module with its own interface.
- Document ID, chunk ID, source, page preserved through every stage.
- Context size bounded by MAX_CONTEXT_TOKENS setting.
- Empty retrieval returns structured empty result — LLM not called.
- Stage failures return typed errors — no silent skips.
- FakeVectorStore and FakeEmbeddingProvider in all unit tests.

IMPLEMENT:
Pipeline: Load → Chunk → Embed → Store → Retrieve → (optional rewrite) → Assemble → Generate

1. Loader: .txt, .pdf, .md; raises ExtractionError on unsupported/parse failure.
2. Splitter: RecursiveCharacterTextSplitter; configurable chunk_size, overlap.
3. EmbeddingService: EmbeddingProvider protocol + OpenAIEmbeddingProvider + FakeEmbeddingProvider.
4. Ingestion pipeline: Load → Chunk → Embed → Store.
5. RetrievalService: retrieve(query, top_k, filters); optional query rewriting on low relevance.
6. ContextAssembler: truncate at MAX_CONTEXT_TOKENS; return text + sources + truncated flag.
7. RAGPipeline: Retrieve → Assemble → Generate; returns RAGResponse(answer, sources, empty_retrieval).
8. POST /v1/rag/query endpoint.

ACCEPTANCE CRITERIA:
AC1: Documents loaded and split with configurable chunk_size and overlap.
AC2: Embeddings generated and stored.
AC3: Semantic search retrieves top-K chunks.
AC4: Query rewriting attempted on low relevance.
AC5: Context includes source, path, page references.
AC6: Context bounded by MAX_CONTEXT_TOKENS; truncated=True when truncated.
AC7: Empty retrieval returns RAGResponse(empty_retrieval=True) — no generation.
AC8: Unsupported format raises ExtractionError.
AC9: Each stage tested independently using fakes.

WORKFLOW:
1. Implement loader.
2. Implement splitter.
3. Implement embedding service + fake.
4. Implement ingestion pipeline.
5. Implement retrieval service with query rewriting.
6. Implement context assembler.
7. Implement end-to-end RAG pipeline.
8. Add FastAPI endpoint.
9. Write stage-level tests.
10. Run: pytest backend/rag/tests/ -v
11. Verify every AC.

FINAL RESPONSE:
- Implementation Summary
- Files Created / Modified
- End-to-End Pipeline Test (document → question → answer + sources)
- Truncation Evidence
- Empty Retrieval Evidence
- Tests Executed and Results
- AC1–AC9 PASS/FAIL with evidence
- Known Limitations
- Recommended Next Ticket: NAI-AI-15

Never claim an AC is PASS without evidence.
```

---

## NAI-AI-15 — Vector Databases

```
You are implementing NAI-AI-15 — Vector Databases for the Nexus AI roadmap.

TICKET KEY: NAI-AI-15
EPIC: AI Roadmap — Full Stack Agentic AI

JIRA TITLE:
Vector Databases

REQUIRED SKILLS:
vector-db
evaluation

Before starting:
1. Read this ticket, all ACs, .kiro/steering/, .kiro/skills/vector-db/SKILL.md.
2. Inspect NAI-AI-02 embedding generation and NAI-AI-14 RAG pipeline.
3. Inspect backend/rag/vector_store/ for existing work.

ARCHITECTURE RULES:
- VectorStore protocol in backend/rag/vector_store/interface.py — no ChromaDB imports.
- ChromaVectorStoreAdapter in chroma_adapter.py.
- FakeVectorStore in fake_vector_store.py.
- Production: always PersistentClient — never in-memory in production.
- anonymized_telemetry=False in all clients.
- Normalize cosine distance: relevance = max(0.0, 1 - distance/2).
- ChromaDB types never appear outside chroma_adapter.py.

IMPLEMENT:
1. DocumentChunk and SearchResult models.
2. VectorStore protocol: insert, search, delete, count, collection_name.
3. ChromaVectorStoreAdapter: upsert on insert; where filter on search; delete by document_id.
4. FakeVectorStore: in-memory with fixed_results override.
5. CollectionConfig: DEFAULT_COLLECTION + CHROMA_PERSIST_DIR from env.
6. Demo: learning/15-vector-databases/chroma_demo.py
   Insert 10 chunks; 3 searches; metadata filter; delete; count.

ACCEPTANCE CRITERIA:
AC1: VectorStore protocol with no ChromaDB imports.
AC2: ChromaVectorStoreAdapter implements protocol.
AC3: Chunks with metadata inserted and retrievable.
AC4: Search returns relevance scores in [0, 1].
AC5: Metadata filter restricts results to specified document_id.
AC6: delete() removes all chunks; count() decreases.
AC7: FakeVectorStore returns deterministic results.
AC8: Tests cover insert, search, filter, delete, count, out-of-range top_k.
AC9: anonymized_telemetry=False in all clients.

WORKFLOW:
1. Read vector-db skill — relevance formula.
2. Create domain models and interface.
3. Implement ChromaVectorStoreAdapter.
4. Implement FakeVectorStore.
5. Build and run demo.
6. Write tests using chromadb.EphemeralClient() for integration.
7. Run: pytest backend/rag/vector_store/tests/ -v
8. Verify every AC.

FINAL RESPONSE:
- Implementation Summary
- Files Created / Modified
- Demo Output (search results table with scores; filter demo)
- Delete + Count Evidence
- Tests Executed and Results
- AC1–AC9 PASS/FAIL with evidence
- Known Limitations
- Recommended Next Ticket: NAI-AI-16

Never claim an AC is PASS without evidence.
```

---

## NAI-AI-16 — LangGraph

```
You are implementing NAI-AI-16 — LangGraph for the Nexus AI roadmap.

TICKET KEY: NAI-AI-16
EPIC: AI Roadmap — Full Stack Agentic AI

JIRA TITLE:
LangGraph

REQUIRED SKILLS:
langgraph
evaluation

Before starting:
1. Read this ticket, all ACs, .kiro/steering/, .kiro/skills/langgraph/SKILL.md — all rules.
2. Inspect NAI-AI-12 LangChain and NAI-AI-11 Agent.
3. Inspect backend/agent-runtime/langgraph/ for existing work.

ARCHITECTURE RULES:
- Every graph has a TypedDict state — no ad-hoc dicts.
- Node functions are pure: accept state, return partial state update dict only.
- Routing logic in standalone functions passed to add_conditional_edges.
- Every looping graph has step_count: int and a guard edge.
- Human approval via interrupt_before — not a blocking call inside a node.
- Checkpointing: MemorySaver in dev; SqliteSaver for persistence.
- LangGraph types do not appear outside backend/agent-runtime/langgraph/.

IMPLEMENT:
1. ResearchState TypedDict with Annotated list accumulation and step_count.
2. Node functions (pure async): planner, researcher, summarizer, human_review.
3. Routing function: should_continue(state) -> str
   (step limit, approval gate, summary done, continue).
4. Graph assembly: conditional edges, MemorySaver checkpoint,
   interrupt_before=["human_review"].
5. Demo: learning/16-langgraph/research_demo.py — normal, step-limit, approval.

ACCEPTANCE CRITERIA:
AC1: ResearchState is typed TypedDict with Annotated accumulation.
AC2: At least three pure async node functions.
AC3: Routing via add_conditional_edges with standalone function.
AC4: step_count prevents unbounded loops.
AC5: MemorySaver checkpoint enables pause and resume by thread_id.
AC6: interrupt_before pauses; graph resumes after approved=True injected.
AC7: Workflow completes end-to-end with FakeListChatModel.
AC8: Tests cover routing logic, step limit, and checkpoint resume.

WORKFLOW:
1. Define ResearchState TypedDict.
2. Implement node functions.
3. Implement routing function.
4. Assemble graph with checkpoint and interrupt.
5. Build and run demo.
6. Write tests (routing tested in isolation; graph with fake LLM).
7. Run: pytest backend/agent-runtime/langgraph/tests/ -v
8. Verify every AC.

FINAL RESPONSE:
- Implementation Summary
- Files Created / Modified
- Graph Execution Trace (normal + step-limit paths)
- Checkpoint Resume Evidence
- Human Approval Gate Evidence
- Tests Executed and Results
- AC1–AC8 PASS/FAIL with evidence
- Known Limitations
- Recommended Next Ticket: NAI-AI-17

Never claim an AC is PASS without evidence.
```

---

## NAI-AI-17 — Model Context Protocol

```
You are implementing NAI-AI-17 — Model Context Protocol for the Nexus AI roadmap.

TICKET KEY: NAI-AI-17
EPIC: AI Roadmap — Full Stack Agentic AI

JIRA TITLE:
Model Context Protocol

REQUIRED SKILLS:
fastapi
multi-agent
evaluation

Before starting:
1. Read this ticket, all ACs, .kiro/steering/ (MCP Architecture section),
   .kiro/skills/fastapi/SKILL.md, .kiro/skills/multi-agent/SKILL.md.
2. Inspect NAI-AI-11 ToolRegistry.
3. Inspect backend/mcp/ for existing work.
4. Read 05-security-standards.md — validate tool inputs; never blindly execute.

ARCHITECTURE RULES:
- MCP transport types never in domain, agent, or API layers.
- MCP tools converted to ToolDefinition and registered in ToolRegistry.
- Agent calls tools by name through ToolRegistry — no MCP transport awareness.
- All tool inputs validated against discovered JSON schema before call.
- asyncio.wait_for timeout enforced per call.
- MCPClientManager handles connect/disconnect/reconnect with backoff.

IMPLEMENT:
1. MCPClientManager: connect, disconnect, is_connected, list_tools, call_tool(timeout).
   Reconnect on ConnectionError with exponential backoff, max 3 retries.
2. MCPToolAdapter: to_tool_definition(mcp_tool) -> ToolDefinition.
   execute_fn calls client.call_tool(); errors mapped to ToolError.
3. Demo MCP server: backend/mcp/servers/demo_server.py with get_weather tool; runnable standalone.
4. MCPToolLoader: load_tools(client, registry) -> int; registers discovered tools.
5. FastAPI: GET /v1/mcp/tools, POST /v1/mcp/tools/{tool_name}/invoke.
6. FakeMCPClient for tests.

ACCEPTANCE CRITERIA:
AC1: MCPClientManager connects and is_connected() returns True.
AC2: list_tools() returns server's tool catalogue.
AC3: call_tool() returns valid result.
AC4: Tool input validated against schema before call.
AC5: Connection lifecycle (connect, disconnect, reconnect) handled.
AC6: MCP errors mapped to domain ToolError types.
AC7: asyncio.wait_for timeout enforced per call.
AC8: Demo server runnable standalone with at least one tool.
AC9: MCP transport types not in domain/, api/, or agent-runtime/agents/.
AC10: Tests use FakeMCPClient.

WORKFLOW:
1. Read multi-agent skill ToolRegistry rules.
2. Read 05-security-standards.md input validation.
3. Implement MCPClientManager.
4. Implement MCPToolAdapter.
5. Build demo server.
6. Implement MCPToolLoader.
7. Add FastAPI endpoints.
8. Create FakeMCPClient.
9. Write tests.
10. Run: pytest backend/mcp/tests/ -v
11. Verify every AC.

FINAL RESPONSE:
- Implementation Summary
- Files Created / Modified
- Tool Discovery Evidence
- Tool Invocation Evidence
- Error Mapping Evidence
- Tests Executed and Results
- AC1–AC10 PASS/FAIL with evidence
- Known Limitations
- Recommended Next Ticket: NAI-AI-18

Never claim an AC is PASS without evidence.
```

---

## NAI-AI-18 — Multi-Agent Systems

```
You are implementing NAI-AI-18 — Multi-Agent Systems for the Nexus AI roadmap.

TICKET KEY: NAI-AI-18
EPIC: AI Roadmap — Full Stack Agentic AI

JIRA TITLE:
Multi-Agent Systems

REQUIRED SKILLS:
multi-agent
evaluation

Before starting:
1. Read this ticket, all ACs, .kiro/steering/, .kiro/skills/multi-agent/SKILL.md — all rules.
2. Inspect NAI-AI-11 (ToolRegistry, AgentResult, AgentExecutor).
3. Inspect NAI-AI-16 (LangGraph — consider for supervisor orchestration).
4. Inspect backend/agent-runtime/multi-agent/ for existing work.

ARCHITECTURE RULES:
- Supervisor delegates — never executes tools directly.
- Specialists communicate only through supervisor.
- Shared ToolRegistry constructed once; passed to all agents.
- Each specialist has max_steps enforced independently.
- Supervisor has max_rounds enforced globally.
- All agents return typed AgentResult.
- CancelledError on supervisor propagates to all running specialists.

IMPLEMENT:
1. Agent protocol: name + async run(task, context) -> AgentResult.
2. AgentContext: task_id, shared_memory, add_result, aggregate_results.
3. ResearcherAgent (max_steps=5) and WriterAgent (max_steps=3) using AgentExecutor.
4. TaskRouter: route(task, context) -> str | None using LLMProvider.
5. SupervisorAgent: delegates to specialists, enforces max_rounds, aggregates results.
6. POST /v1/agents/multi/run endpoint.
7. Demo: learning/18-multi-agent/research_demo.py

ACCEPTANCE CRITERIA:
AC1: Supervisor delegates to at least two specialists.
AC2: Specialists communicate only through supervisor.
AC3: Shared ToolRegistry used by all agents.
AC4: Each specialist has independent max_steps.
AC5: Supervisor enforces max_rounds; returns STEP_LIMIT_REACHED at limit.
AC6: asyncio.cancel() on supervisor propagates to running specialists.
AC7: Every agent returns typed AgentResult with status, output, steps_taken.
AC8: Bounded multi-agent task completes end-to-end with SUCCESS.
AC9: Tests use FakeAgent specialists; verify delegation sequence and aggregation.

WORKFLOW:
1. Read multi-agent skill safeguards.
2. Define Agent protocol and AgentContext.
3. Implement ResearcherAgent and WriterAgent.
4. Implement TaskRouter.
5. Implement SupervisorAgent.
6. Add FastAPI endpoint.
7. Build demo.
8. Write tests with FakeAgent.
9. Run: pytest backend/agent-runtime/multi-agent/tests/ -v
10. Verify every AC.

FINAL RESPONSE:
- Implementation Summary
- Files Created / Modified
- Delegation Trace from Demo
- Step Limit Enforcement Evidence
- Cancellation Test Evidence
- Tests Executed and Results
- AC1–AC9 PASS/FAIL with evidence
- Known Limitations
- Recommended Next Ticket: NAI-AI-19

Never claim an AC is PASS without evidence.
```

---

## NAI-AI-19 — AI Evaluation

```
You are implementing NAI-AI-19 — AI Evaluation for the Nexus AI roadmap.

TICKET KEY: NAI-AI-19
EPIC: AI Roadmap — Full Stack Agentic AI

JIRA TITLE:
AI Evaluation

REQUIRED SKILLS:
evaluation

Before starting:
1. Read this ticket, all ACs, .kiro/steering/, .kiro/skills/evaluation/SKILL.md — all rules.
2. Inspect NAI-AI-14 RAG pipeline and NAI-AI-18 multi-agent (systems under evaluation).
3. Read 05-security-standards.md — no production user data in evaluation sets.

ARCHITECTURE RULES:
- Datasets versioned JSON files — never overwrite; append new versions.
- Judge LLM temperature=0 for reproducibility.
- EvaluationReport records run_id, timestamp, model_version, dataset_version, all scores.
- Unit tests use FakeLLMProvider as judge — zero real API calls.
- No production PII in evaluation datasets.

IMPLEMENT:
1. RAG eval dataset: backend/evaluation/datasets/rag_eval_v1.json — ≥10 items with reference answers.
2. Agent eval dataset: backend/evaluation/datasets/agent_eval_v1.json — ≥5 tasks with criteria.
3. RAG metrics: compute_faithfulness, compute_answer_relevance, compute_context_precision,
   compute_context_recall (all return float [0,1]).
4. Agent metrics: task_completion_rate, avg_steps, step_limit_hit_rate.
5. Operational metrics: OperationalMetrics dataclass; compute_percentiles(times) -> p50/p95/p99.
6. RAG evaluator runner: iterates dataset; computes metrics; records operational data.
7. Agent evaluator runner: iterates tasks; records AgentResult + operational data.
8. Report generator: EvaluationReport saved to backend/evaluation/reports/{run_id}.json.
9. FastAPI: GET /v1/evaluation/reports, GET /v1/evaluation/reports/{run_id}.

ACCEPTANCE CRITERIA:
AC1: RAG dataset with ≥10 items including reference answers.
AC2: compute_faithfulness() returns [0,1]; unit tested with known inputs.
AC3: compute_answer_relevance() returns [0,1].
AC4: Agent task completion rate computed correctly.
AC5: Latency tracked as p50 and p95.
AC6: Token usage and estimated cost recorded per item.
AC7: EvaluationReport generated with all required fields.
AC8: Two reports from different model configs comparable (same schema).
AC9: Judge LLM uses temperature=0; judge model recorded in report.
AC10: All metric tests use FakeLLMProvider.

WORKFLOW:
1. Create evaluation datasets.
2. Implement RAG metrics.
3. Implement agent metrics.
4. Implement operational metrics.
5. Implement evaluation runners.
6. Implement report generator.
7. Add FastAPI endpoints.
8. Write metric unit tests.
9. Run full eval against FakeLLMProvider; verify report.
10. Run: pytest backend/evaluation/tests/ -v
11. Verify every AC.

FINAL RESPONSE:
- Implementation Summary
- Files Created / Modified
- Sample EvaluationReport JSON (condensed)
- Two-Report Comparison Evidence
- Tests Executed and Results
- AC1–AC10 PASS/FAIL with evidence
- Known Limitations
- Recommended Next Ticket: NAI-AI-20

Never claim an AC is PASS without evidence.
```

---

## NAI-AI-20 — Deployment & Production AI Applications

```
You are implementing NAI-AI-20 — Deployment & Production AI Applications for the Nexus AI roadmap.

TICKET KEY: NAI-AI-20
EPIC: AI Roadmap — Full Stack Agentic AI

JIRA TITLE:
Deployment & Production AI Applications

REQUIRED SKILLS:
deployment
evaluation

Before starting:
1. Read this ticket, all ACs, .kiro/steering/, .kiro/skills/deployment/SKILL.md — all rules.
2. Confirm NAI-AI-08 health/readiness endpoints are implemented before writing Dockerfiles.
3. Inspect infrastructure/ for existing Docker or CI/CD config.
4. Read 05-security-standards.md — secrets, minimum privilege, HTTPS.

ARCHITECTURE RULES:
- Secrets from Secret Manager at runtime — never baked into image layers.
- Container runs as non-root user.
- Production image tags: exact SHA or versioned tag — never "latest".
- All logs: newline-delimited JSON with severity, message, timestamp, service.
- Service account: minimum required IAM roles only.
- Rollback: single gcloud run services update-traffic command.
- .env files never committed; .env.example committed with placeholders.

IMPLEMENT:
1. Dockerfile: infrastructure/docker/fastapi.Dockerfile
   python:3.11-slim; layer order; non-root user; EXPOSE 8080.
2. Docker Compose: infrastructure/docker/docker-compose.yml
   fastapi + chroma services; .env mount; health check.
3. Structured JSON logging: CloudLoggingFormatter in
   backend/fastapi-service/infrastructure/logging.py; called from main.py startup.
4. Cloud Run YAML: infrastructure/cloud-run/fastapi-service.yaml
   minScale/maxScale; resource limits; Secret Manager env var.
5. GitHub Actions:
   infrastructure/ci-cd/build-push.yml — build + push with SHA tag.
   infrastructure/ci-cd/deploy.yml — deploy to staging; health check; shift traffic.
6. Environment examples: infrastructure/deployment/environments/
   local.env.example, staging.env.example, production.env.example.
7. Runbook: infrastructure/deployment/runbook.md
   Prerequisites; deploy staging; promote production; rollback command; health verification;
   log location; IAM permissions documented.

ACCEPTANCE CRITERIA:
AC1: Docker image builds successfully.
AC2: docker-compose up starts full local stack; GET /health returns 200.
AC3: Service deploys to Cloud Run staging.
AC4: All secrets from Secret Manager — none hardcoded.
AC5: GET /health returns {"status": "ok"} on deployed service.
AC6: GET /readiness returns provider health on deployed service.
AC7: All container logs are valid structured JSON in Cloud Logging.
AC8: Rollback to previous revision demonstrated.
AC9: GitHub Actions workflow builds, pushes, and deploys.
AC10: runbook.md documents full deployment and rollback procedure.
AC11: Service account uses minimum required IAM permissions (documented).

WORKFLOW:
1. Read deployment skill — all rules and patterns.
2. Write Dockerfile; verify local build.
3. Write docker-compose.yml; verify local stack.
4. Implement structured JSON logging; verify format.
5. Write Cloud Run YAML.
6. Configure Secret Manager.
7. Write GitHub Actions workflows.
8. Deploy to staging; run health/readiness checks.
9. Demonstrate rollback.
10. Write runbook.
11. Verify every AC with concrete evidence.

FINAL RESPONSE:
- Implementation Summary
- Files Created / Modified
- Docker Build Evidence
- Local Stack Evidence (docker-compose + curl /health)
- Cloud Run Deployment Evidence (URL + /health response)
- Rollback Evidence (gcloud command + traffic output)
- Structured Log Evidence (sample JSON line)
- Tests Executed and Results
- AC1–AC11 PASS/FAIL with evidence
- Known Limitations
- IAM Permissions Documented

Never claim an AC is PASS without evidence.
Never claim production readiness without actual build, deployment, and rollback evidence.
```
