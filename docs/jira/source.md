# Nexus AI — Kiro Prompts for 20 Jira Tickets

## PURPOSE

This document is the **single source of truth** for all 20 Nexus AI Jira execution prompts.

Each section defines one Jira ticket's Kiro execution prompt. The corresponding individual prompt
file is generated from this document and stored under `docs/jira/kiro-prompts/`.

Do not edit individual prompt files directly — edit this source document and regenerate.

---

## DOCUMENT STRUCTURE

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
