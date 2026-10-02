# Skill: AI Provider

## Purpose

Provides reusable expertise for implementing AI provider integrations in Nexus AI: building provider implementations that satisfy the domain interfaces, handling streaming, mapping provider-specific types to domain types, managing errors, and testing providers with fakes.

## When to Use

Activate this skill when:
- Implementing a new AI provider (Gemini, OpenAI, on-device, future)
- Adding streaming support to a provider
- Mapping provider SDK models to domain types
- Handling provider-specific errors and rate limiting
- Configuring AI request parameters (model, temperature, tools, context)
- Writing tests for provider implementations
- Reviewing a provider implementation for boundary violations

## Core Rules

Refer to `06-ai-architecture.md` for the authoritative AI architecture rules. This skill provides implementation patterns.

1. **Implement the domain interface.** Every provider implements `AIProvider` / `AIStreamClient` — nothing more exposed.
2. **No provider types outside the provider class.** SDK models are mapped at the boundary, inside the provider implementation.
3. **All streaming via `Flow<AIStreamEvent>`.** Consumers never see provider SDK streaming types.
4. **Handle every error case.** Map to the `AIError` sealed hierarchy. Never let raw provider exceptions escape.
5. **Respect coroutine cancellation.** Streaming flows must cancel cleanly.
6. **Never log API keys or response content** that may contain sensitive data.

## Implementation Guidance

### Domain Interfaces (defined in Domain layer)

These interfaces are the contract every provider must implement. Names are illustrative — exact names defined in Specs:

```kotlin
// Domain layer — no provider SDK imports here
interface AIProvider {
    val id: String
    val supportedCapabilities: Set<AICapability>
    suspend fun generate(request: AIRequest): Result<AIResponse>
    fun stream(request: AIRequest): Flow<AIStreamEvent>
}

data class AIRequest(
    val messages: List<AIMessage>,
    val tools: List<ToolDefinition> = emptyList(),
    val systemPrompt: String? = null,
    val config: AIGenerationConfig = AIGenerationConfig(),
)

data class AIGenerationConfig(
    val temperature: Float = 0.7f,
    val maxOutputTokens: Int = 2048,
    val topP: Float = 0.95f,
)

sealed interface AIStreamEvent {
    data object Started : AIStreamEvent
    data class Token(val text: String) : AIStreamEvent
    data class ToolCall(val id: String, val name: String, val input: Map<String, Any>) : AIStreamEvent
    data class ToolResult(val id: String, val result: String) : AIStreamEvent
    data class Completed(val fullText: String, val usage: TokenUsage?) : AIStreamEvent
    data class Error(val error: AIError) : AIStreamEvent
}
```

### Provider Implementation Structure (Data layer)

```kotlin
// Data layer — provider SDK imports are ONLY here
class GeminiAIProvider @Inject constructor(
    @Named("gemini_api_key") private val apiKey: String,
    private val requestMapper: GeminiRequestMapper,
    private val responseMapper: GeminiResponseMapper,
    private val errorMapper: GeminiErrorMapper,
) : AIProvider {

    override val id = "gemini-1.5-pro"
    override val supportedCapabilities = setOf(
        AICapability.Chat,
        AICapability.Streaming,
        AICapability.Tools,
        AICapability.Vision,
    )

    private val generativeModel by lazy {
        GenerativeModel(modelName = id, apiKey = apiKey)
    }

    override suspend fun generate(request: AIRequest): Result<AIResponse> =
        runCatching {
            val geminiRequest = requestMapper.toGemini(request)
            val response = generativeModel.generateContent(geminiRequest)
            responseMapper.toDomain(response)
        }.mapFailure { errorMapper.toDomain(it) }

    override fun stream(request: AIRequest): Flow<AIStreamEvent> = flow {
        emit(AIStreamEvent.Started)
        try {
            val geminiRequest = requestMapper.toGemini(request)
            generativeModel.generateContentStream(geminiRequest).collect { chunk ->
                // Map each SDK chunk to domain event
                responseMapper.chunkToEvents(chunk).forEach { emit(it) }
            }
            emit(AIStreamEvent.Completed(fullText = "", usage = null)) // accumulate in orchestration
        } catch (e: CancellationException) {
            throw e // Never swallow cancellation
        } catch (e: Exception) {
            emit(AIStreamEvent.Error(errorMapper.toDomain(e)))
        }
    }.flowOn(Dispatchers.IO)
}
```

### Request Mapper (Data — provider-specific)

```kotlin
class GeminiRequestMapper @Inject constructor() {

    fun toGemini(request: AIRequest): GenerateContentRequest {
        val contents = request.messages.map { message ->
            content(role = message.role.toGeminiRole()) {
                text(message.content)
            }
        }
        return GenerateContentRequest(
            model = "gemini-1.5-pro",
            contents = contents,
            // Map tools if present
            tools = if (request.tools.isNotEmpty()) listOf(tools { request.tools.forEach { tool ->
                functionDeclarations.add(tool.toGeminiFunctionDeclaration())
            }}) else null,
        )
    }
}
```

### Error Mapping

```kotlin
class GeminiErrorMapper @Inject constructor() {

    fun toDomain(throwable: Throwable): AIError = when (throwable) {
        is GoogleGenerativeAIException -> when {
            throwable.message?.contains("API key") == true -> AIError.AuthError
            throwable.message?.contains("429") == true -> AIError.RateLimitError(retryAfterMs = null)
            throwable.message?.contains("quota") == true -> AIError.RateLimitError(retryAfterMs = null)
            throwable.message?.contains("context") == true -> AIError.ContextLengthError
            throwable.message?.contains("SAFETY") == true -> AIError.ContentFilterError
            else -> AIError.ProviderError(code = throwable.message ?: "unknown", provider = "gemini")
        }
        is IOException -> AIError.NetworkError(cause = throwable)
        is TimeoutCancellationException -> AIError.TimeoutError
        else -> AIError.UnknownError(cause = throwable)
    }
}
```

### On-Device Provider

```kotlin
class OnDeviceAIProvider @Inject constructor(
    private val modelManager: OnDeviceModelManager,
    private val requestMapper: OnDeviceRequestMapper,
    private val responseMapper: OnDeviceResponseMapper,
    private val errorMapper: OnDeviceErrorMapper,
) : AIProvider {

    override val id = "gemini-nano"
    override val supportedCapabilities = setOf(AICapability.Chat, AICapability.Streaming)

    override fun stream(request: AIRequest): Flow<AIStreamEvent> = flow {
        if (!modelManager.isModelAvailable()) {
            emit(AIStreamEvent.Error(AIError.ModelNotAvailable))
            return@flow
        }
        // Warm up model if needed, then stream
        val session = modelManager.createSession()
        try {
            emit(AIStreamEvent.Started)
            session.generateStream(requestMapper.toLocal(request)).collect { chunk ->
                emit(responseMapper.chunkToEvent(chunk))
            }
        } catch (e: CancellationException) {
            throw e
        } catch (e: Exception) {
            emit(AIStreamEvent.Error(errorMapper.toDomain(e)))
        } finally {
            session.close()
        }
    }.flowOn(Dispatchers.Default)
}
```

### Provider Registration via Hilt

```kotlin
@Module
@InstallIn(SingletonComponent::class)
object AIProviderModule {

    @Provides
    @Singleton
    fun provideAIProviderRegistry(
        geminiProvider: GeminiAIProvider,
        openAIProvider: OpenAIProvider,
        onDeviceProvider: OnDeviceAIProvider,
    ): AIProviderRegistry = AIProviderRegistry(
        providers = mapOf(
            "gemini" to geminiProvider,
            "openai" to openAIProvider,
            "on-device" to onDeviceProvider,
        )
    )
}
```

## Common Mistakes

| Mistake | Correct Approach |
|---|---|
| Returning `GenerateContentResponse` from a provider method | Map to `AIResponse` inside the provider; never expose SDK types |
| Catching `CancellationException` and not rethrowing | Always rethrow `CancellationException` — it controls coroutine cancellation |
| Leaving a streaming flow open indefinitely | Always emit `Completed` or `Error` as the terminal event |
| Provider class has 10+ injected dependencies | Extract request mapper, response mapper, error mapper as separate injected classes |
| Logging `request.messages` content | Log only request ID and provider name; never message content |
| Using `blocking` SDK calls inside a flow | Wrap in `withContext(Dispatchers.IO)` or use SDK's async/Flow APIs |
| Hardcoded model name in provider class | Inject model name via configuration / Hilt `@Named` |
| Not handling partial streaming failures | Emit `Error` event; don't leave consumer hanging |

## Testing Guidance

- Use `FakeAIProvider` from `core/testing/` in all tests above the provider layer
- Test provider implementations directly with recorded/stubbed SDK responses
- Never make real network calls in provider unit tests
- Test error mapping exhaustively — every provider error code should have a test
- Test streaming cancellation: cancel the collection mid-stream and verify no resource leaks
- Test that `CancellationException` propagates correctly and is not swallowed

## Relationship to Steering

This skill applies `06-ai-architecture.md`. It also intersects with `05-security-standards.md` for API key handling and `04-testing-standards.md` for fake providers. Steering takes precedence.
