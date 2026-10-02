# Skill: Agent

## Purpose

Provides reusable expertise for implementing AI agents in Nexus AI: structured task execution, planning, tool selection, tool execution, observation loops, step limits, cancellation, error recovery, and testing.

## When to Use

Activate this skill when:
- Implementing an agent engine or agent execution loop
- Designing agent task models and execution context
- Integrating agents with orchestration and tools
- Implementing step limits, timeouts, and cancellation
- Handling agent error recovery and fallback
- Writing tests for agent execution
- Reviewing agent code for unbounded recursion or missing safety limits

## Core Rules

Refer to `01-architecture.md` (Agent Architecture) and `06-ai-architecture.md` (Agent + AI) for the authoritative rules. This skill provides implementation patterns.

1. **Agents never call AI providers directly.** Always via Orchestration.
2. **Agents never call MCP transport directly.** Always via `ApplicationTool` abstractions.
3. **Maximum step limit is mandatory** and enforced before execution begins.
4. **Every step has a timeout.** No step runs indefinitely.
5. **Coroutine cancellation propagates** to all in-flight tool executions.
6. **Never implement unbounded recursive execution.** Finite loops only.

## Agent Execution Lifecycle

```
AgentTask (structured input)
    ↓
Task Understanding  — parse and validate the task
    ↓
Context Collection  — gather memory, tools, relevant state
    ↓
Planning            — AI generates execution plan
    ↓
Tool Selection      — AI selects the next tool to call
    ↓
Tool Validation     — validate tool exists and input is valid
    ↓
Tool Execution      — execute tool with timeout
    ↓
Observation         — record tool result, update state
    ↓
Next Step Decision  — AI decides: continue or complete
    ↓
Completion          — result, step limit, cancellation, or error
```

## Implementation Guidance

### Agent Task and Result Models (Domain)

```kotlin
data class AgentTask(
    val id: String = generateId(),
    val instruction: String,
    val context: Map<String, String> = emptyMap(),
    val availableToolNames: List<String>? = null, // null = all registered tools
)

sealed interface AgentResult {
    data class Completed(val output: String, val steps: Int) : AgentResult
    data class StepLimitReached(val lastOutput: String, val steps: Int) : AgentResult
    data class Cancelled(val completedSteps: Int) : AgentResult
    data class Failed(val error: AgentError, val completedSteps: Int) : AgentResult
}

sealed interface AgentError {
    data class ToolExecutionFailed(val toolName: String, val cause: ToolError) : AgentError
    data class PlanningFailed(val cause: AIError) : AgentError
    data class InvalidToolCall(val toolName: String, val reason: String) : AgentError
    data object ContextTooLarge : AgentError
    data class UnexpectedError(val cause: Throwable) : AgentError
}
```

### Agent Engine (Domain)

```kotlin
class AgentEngine @Inject constructor(
    private val orchestration: AIOrchestration,
    private val toolRegistry: ToolRegistry,
    private val memoryRepository: MemoryRepository,
    private val config: AgentConfig,
) {
    /**
     * Executes an agent task with structured step loop, enforced step limit,
     * per-step timeout, and coroutine cancellation support.
     */
    suspend fun execute(task: AgentTask): AgentResult {
        val tools = resolveTools(task.availableToolNames)
        val context = AgentContext(task = task, tools = tools)
        var steps = 0

        while (steps < config.maxSteps) {
            // Check for coroutine cancellation before each step
            currentCoroutineContext().ensureActive()

            steps++
            val stepResult = executeStep(context, steps)

            when (stepResult) {
                is StepResult.Complete -> return AgentResult.Completed(stepResult.output, steps)
                is StepResult.Continue -> context.addObservation(stepResult.observation)
                is StepResult.Failed -> return AgentResult.Failed(stepResult.error, steps)
            }
        }

        return AgentResult.StepLimitReached(
            lastOutput = context.lastObservation ?: "No output",
            steps = steps,
        )
    }

    private suspend fun executeStep(
        context: AgentContext,
        stepNumber: Int,
    ): StepResult = withTimeout(config.stepTimeoutMs) {
        try {
            // Ask AI what to do next
            val planEvent = orchestration.planNextStep(context)

            when (planEvent) {
                is PlanEvent.Complete -> StepResult.Complete(planEvent.output)
                is PlanEvent.CallTool -> {
                    val tool = toolRegistry.resolve(planEvent.toolName)
                        ?: return@withTimeout StepResult.Failed(
                            AgentError.InvalidToolCall(planEvent.toolName, "tool not found")
                        )
                    val toolResult = tool.execute(planEvent.input)
                    StepResult.Continue(
                        observation = Observation(
                            stepNumber = stepNumber,
                            toolName = planEvent.toolName,
                            result = toolResult,
                        )
                    )
                }
            }
        } catch (e: CancellationException) {
            throw e // Propagate — do not convert to Failed
        } catch (e: Exception) {
            StepResult.Failed(AgentError.UnexpectedError(e))
        }
    }

    private fun resolveTools(toolNames: List<String>?): List<ApplicationTool> =
        toolNames?.mapNotNull { toolRegistry.resolve(it) }
            ?: toolRegistry.allTools()
}
```

### Agent Configuration

```kotlin
data class AgentConfig(
    val maxSteps: Int = 10,
    val stepTimeoutMs: Long = 30_000,
    val totalTimeoutMs: Long = 300_000,
)
```

Default values are conservative. Features can configure per-use-case but must not set `maxSteps = Int.MAX_VALUE` or `stepTimeoutMs = Long.MAX_VALUE`.

### Agent Context

```kotlin
data class AgentContext(
    val task: AgentTask,
    val tools: List<ApplicationTool>,
    val observations: MutableList<Observation> = mutableListOf(),
) {
    val lastObservation: String? get() = observations.lastOrNull()?.result?.toString()

    fun addObservation(observation: Observation) {
        observations.add(observation)
        // Enforce context growth limit
        if (observations.size > MAX_OBSERVATIONS) {
            observations.removeAt(0) // Slide window
        }
    }

    companion object { private const val MAX_OBSERVATIONS = 50 }
}
```

### Exposing Agent Execution to UI

```kotlin
// ViewModel exposes agent state as UiState — no agent internals leak to Compose
sealed interface AgentUiState {
    data object Idle : AgentUiState
    data class Running(val currentStep: Int, val maxSteps: Int, val lastAction: String) : AgentUiState
    data class Completed(val output: String, val steps: Int) : AgentUiState
    data class Failed(val message: UiText) : AgentUiState
}

@HiltViewModel
class AgentViewModel @Inject constructor(
    private val executeAgentUseCase: ExecuteAgentUseCase,
) : ViewModel() {

    private val _uiState = MutableStateFlow<AgentUiState>(AgentUiState.Idle)
    val uiState: StateFlow<AgentUiState> = _uiState.asStateFlow()

    private var agentJob: Job? = null

    fun startAgent(instruction: String) {
        agentJob = viewModelScope.launch {
            _uiState.value = AgentUiState.Running(0, DEFAULT_MAX_STEPS, "Starting...")
            val result = executeAgentUseCase(instruction)
            _uiState.value = when (result) {
                is AgentResult.Completed -> AgentUiState.Completed(result.output, result.steps)
                is AgentResult.StepLimitReached -> AgentUiState.Failed(UiText.StepLimitReached)
                is AgentResult.Cancelled -> AgentUiState.Idle
                is AgentResult.Failed -> AgentUiState.Failed(result.error.toUiText())
            }
        }
    }

    fun cancelAgent() {
        agentJob?.cancel()
        _uiState.value = AgentUiState.Idle
    }
}
```

## Common Mistakes

| Mistake | Correct Approach |
|---|---|
| `while (true)` execution loop | Always `while (steps < config.maxSteps)` |
| Agent calls `AIProvider.stream()` directly | Agent calls `orchestration.planNextStep()` only |
| Agent calls `MCPClient` directly | Agent calls `tool.execute()` via `ApplicationTool` abstraction |
| Catching `CancellationException` in the step loop | Rethrow it — cancellation must propagate |
| No per-step timeout | Wrap every step in `withTimeout(config.stepTimeoutMs)` |
| Agent result not modeled as sealed type | Use `AgentResult` sealed interface; callers must handle all cases |
| Passing unbounded `context.observations` to AI | Slide the observation window; bound context size |
| Exposing `AgentContext` or `Observation` to Compose | Map to `AgentUiState` in ViewModel; Compose sees only UI types |

## Testing Guidance

```kotlin
@Test
fun `agent completes successfully within step limit`() = runTest {
    val fakeOrchestration = FakeAIOrchestration()
    fakeOrchestration.planResponses = listOf(
        PlanEvent.CallTool("search", mapOf("q" to "kotlin")),
        PlanEvent.Complete("Kotlin is a modern language."),
    )
    val fakeTool = FakeTool("search", Result.success(ToolOutput("search results")))

    val agent = AgentEngine(fakeOrchestration, FakeToolRegistry(listOf(fakeTool)), AgentConfig(maxSteps = 5))
    val result = agent.execute(AgentTask(instruction = "tell me about kotlin"))

    assertThat(result).isInstanceOf(AgentResult.Completed::class.java)
    assertThat((result as AgentResult.Completed).steps).isEqualTo(2)
}

@Test
fun `agent returns StepLimitReached when AI always requests a tool`() = runTest {
    val fakeOrchestration = FakeAIOrchestration()
    fakeOrchestration.alwaysRespond(PlanEvent.CallTool("search", mapOf("q" to "test")))

    val agent = AgentEngine(fakeOrchestration, FakeToolRegistry(listOf(FakeTool("search"))), AgentConfig(maxSteps = 3))
    val result = agent.execute(AgentTask(instruction = "keep searching"))

    assertThat(result).isInstanceOf(AgentResult.StepLimitReached::class.java)
}

@Test
fun `agent cancels cleanly when coroutine is cancelled`() = runTest {
    val fakeOrchestration = FakeAIOrchestration()
    fakeOrchestration.delayEachStep(500.milliseconds)

    val agent = AgentEngine(fakeOrchestration, FakeToolRegistry(), AgentConfig(maxSteps = 10))
    val job = launch { agent.execute(AgentTask(instruction = "do something slow")) }

    advanceTimeBy(600)
    job.cancel()
    job.join()

    assertThat(job.isCancelled).isTrue()
}
```

## Relationship to Steering

This skill applies `01-architecture.md` (Agent Architecture) and `06-ai-architecture.md` (Agent + AI). Testing patterns apply `04-testing-standards.md`. Steering takes precedence over any guidance here.
