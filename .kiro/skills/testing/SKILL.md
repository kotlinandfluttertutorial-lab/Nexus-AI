# Skill: Testing

## Purpose

Provides reusable expertise for writing deterministic, isolated, readable tests across all Nexus AI subsystems: use cases, ViewModels, repositories, AI orchestration, MCP, agents, RAG, tools, memory, WorkManager, and Compose UI.

## When to Use

Activate this skill when:
- Writing unit tests for any domain, data, or presentation class
- Writing Compose UI tests for any screen
- Creating shared test fakes in `core/testing/`
- Debugging flaky or non-deterministic tests
- Designing the test strategy for a new feature or subsystem
- Reviewing tests in a PR

## Core Rules

Refer to `04-testing-standards.md` for the authoritative rules. This skill provides implementation patterns.

1. **Deterministic always.** No real network, no real AI providers, no `Thread.sleep()`.
2. **Fakes over mocks** for complex collaborators. Mocks for simple interaction verification.
3. **Coroutines use `runTest`** and virtual time. Never real time delays in tests.
4. **Flows use Turbine.** Collect and assert emissions precisely.
5. **Compose tests use test tags.** Never text strings for node lookup.
6. **Never weaken an assertion** to make a test pass.

## Shared Test Fakes (`core/testing/`)

All fakes are shared across modules. Do not create one-off mocks for complex collaborators.

### FakeAIProvider

```kotlin
class FakeAIProvider : AIProvider {
    var streamEvents: List<AIStreamEvent> = emptyList()
    var shouldThrow: AIError? = null
    val capturedRequests = mutableListOf<AIRequest>()

    override fun streamChat(request: AIRequest): Flow<AIStreamEvent> = flow {
        capturedRequests.add(request)
        shouldThrow?.let { throw it }
        streamEvents.forEach { emit(it) }
    }
}
```

### FakeRepository

```kotlin
class FakeMessageRepository : MessageRepository {
    private val messages = mutableMapOf<String, MutableList<Message>>()
    var saveError: Throwable? = null

    override fun getMessages(conversationId: String): Flow<List<Message>> =
        flowOf(messages[conversationId] ?: emptyList())

    override suspend fun saveMessage(message: Message): Result<Unit> {
        saveError?.let { return Result.failure(it) }
        messages.getOrPut(message.conversationId) { mutableListOf() }.add(message)
        return Result.success(Unit)
    }

    override suspend fun deleteMessage(messageId: String): Result<Unit> {
        messages.values.forEach { list -> list.removeIf { it.id == messageId } }
        return Result.success(Unit)
    }

    override suspend fun clearConversation(conversationId: String): Result<Unit> {
        messages.remove(conversationId)
        return Result.success(Unit)
    }
}
```

### FakeMCPClient

```kotlin
class FakeMCPClient : MCPClient {
    var availableTools: List<MCPToolDefinition> = emptyList()
    var invocationResults = mutableMapOf<String, Result<ToolOutput>>()
    val capturedInvocations = mutableListOf<Pair<String, ToolInput>>()

    override suspend fun discoverTools(): Result<List<MCPToolDefinition>> =
        Result.success(availableTools)

    override suspend fun invokeTool(name: String, input: ToolInput): Result<ToolOutput> {
        capturedInvocations.add(name to input)
        return invocationResults[name] ?: Result.failure(MCPError.ToolNotFound(name))
    }
}
```

### FakeEmbeddingProvider

```kotlin
class FakeEmbeddingProvider : EmbeddingProvider {
    // Deterministic: hash-based fixed-dimension vectors
    override suspend fun embed(text: String): Result<Embedding> =
        Result.success(Embedding(FloatArray(DIMENSION) { i -> (text.hashCode() xor i).toFloat() }))

    companion object { const val DIMENSION = 384 }
}
```

### FakeVectorStore

```kotlin
class FakeVectorStore : VectorStore {
    private val store = mutableListOf<Pair<Chunk, Embedding>>()

    override suspend fun upsert(chunk: Chunk, embedding: Embedding): Result<Unit> {
        store.add(chunk to embedding)
        return Result.success(Unit)
    }

    // Returns stored chunks in insertion order — deterministic
    override suspend fun search(query: Embedding, topK: Int): Result<List<ScoredChunk>> =
        Result.success(store.take(topK).map { (chunk, _) -> ScoredChunk(chunk, score = 1.0f) })
}
```

## Use Case Testing Pattern

```kotlin
class SendMessageUseCaseTest {

    private val fakeMessageRepository = FakeMessageRepository()
    private val fakeAIProvider = FakeAIProvider()
    private val useCase = SendMessageUseCase(fakeMessageRepository, FakeAIOrchestration(fakeAIProvider))

    @Test
    fun `given valid input, when invoked, then message is saved and stream is emitted`() = runTest {
        fakeAIProvider.streamEvents = listOf(
            AIStreamEvent.Token("Hello"),
            AIStreamEvent.Completed("Hello"),
        )

        val emissions = useCase("conv-1", "Hi").toList()

        assertThat(emissions).hasSize(2)
        assertThat(fakeMessageRepository.capturedRequests).isNotEmpty()
    }

    @Test
    fun `given AI provider error, when invoked, then error is propagated`() = runTest {
        fakeAIProvider.shouldThrow = AIError.NetworkError

        val result = runCatching { useCase("conv-1", "Hi").toList() }

        assertThat(result.isFailure).isTrue()
    }
}
```

## ViewModel Testing Pattern

```kotlin
class ChatViewModelTest {

    @get:Rule val mainDispatcherRule = MainDispatcherRule()

    private val fakeSendMessage = FakeSendMessageUseCase()
    private val viewModel = ChatViewModel(fakeSendMessage)

    @Test
    fun `sendMessage transitions from Idle to Loading to Success`() = runTest {
        viewModel.uiState.test {
            assertThat(awaitItem()).isInstanceOf(ChatUiState.Idle::class.java)

            viewModel.sendMessage("Hello")
            assertThat(awaitItem()).isInstanceOf(ChatUiState.Loading::class.java)
            assertThat(awaitItem()).isInstanceOf(ChatUiState.Success::class.java)
        }
    }
}

// Reusable test rule for Main dispatcher
class MainDispatcherRule(
    val dispatcher: TestCoroutineDispatcher = TestCoroutineDispatcher(),
) : TestWatcher() {
    override fun starting(description: Description) = Dispatchers.setMain(dispatcher)
    override fun finished(description: Description) = Dispatchers.resetMain()
}
```

## Flow Testing with Turbine

```kotlin
@Test
fun `message flow emits new message after save`() = runTest {
    val repo = FakeMessageRepository()

    repo.getMessages("conv-1").test {
        assertThat(awaitItem()).isEmpty()

        repo.saveMessage(testMessage)
        assertThat(awaitItem()).hasSize(1)

        cancelAndIgnoreRemainingEvents()
    }
}
```

## Compose UI Testing Pattern

```kotlin
@get:Rule val composeTestRule = createComposeRule()

@Test
fun `ChatContent shows message list in success state`() {
    val messages = listOf(MessageUi(id = "1", content = "Hello", isFromUser = true))

    composeTestRule.setContent {
        NexusAITheme {
            ChatContent(
                uiState = ChatUiState.Success(messages = messages),
                onSendMessage = {},
                onInputChange = {},
            )
        }
    }

    composeTestRule.onNodeWithTag(ChatScreenTestTags.MessageList).assertIsDisplayed()
    composeTestRule.onNodeWithTag("MessageBubble_1").assertIsDisplayed()
}
```

## Agent Testing Pattern

```kotlin
@Test
fun `agent stops after max steps are reached`() = runTest {
    val fakeProvider = FakeAIProvider()
    // AI always wants to call a tool — should stop at step limit
    fakeProvider.streamEvents = listOf(AIStreamEvent.ToolCall("search", mapOf("q" to "test")))

    val agent = AgentEngine(
        aiOrchestration = FakeAIOrchestration(fakeProvider),
        tools = listOf(FakeTool("search")),
        maxSteps = 3,
    )

    val result = agent.execute(AgentTask("find something"))

    assertThat(result).isInstanceOf(AgentResult.StepLimitReached::class.java)
    assertThat(fakeProvider.capturedRequests).hasSize(3)
}
```

## Common Mistakes

| Mistake | Correct Approach |
|---|---|
| `Thread.sleep(500)` in tests | Use `advanceTimeBy()` or `advanceUntilIdle()` from `TestCoroutineScheduler` |
| Real network call in unit test | Use `FakeAIProvider` or `FakeMCPClient` |
| `verify(mock).method()` on a complex collaborator | Build a fake instead; verify via state/output |
| Single `verify(mock)` replacing a real assertion | Assert the actual outcome, not just that a method was called |
| Missing `cancelAndIgnoreRemainingEvents()` at end of Turbine block | Always cancel or consume remaining events to avoid test hangs |
| `@Ignore` without a Jira ticket | Add `// TODO: NAI-XX re-enable after fix` with ticket reference |
| Sharing mutable state between `@Test` functions | Always construct fresh fakes in each test or `@Before` |
| Testing private methods | Test public behavior; private methods are covered by testing the public surface |

## Relationship to Steering

This skill applies `04-testing-standards.md`. All fake implementations must follow `03-coding-standards.md`. Steering takes precedence over any guidance here.
