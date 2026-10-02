# Skill: Clean Architecture

## Purpose

Provides reusable expertise for structuring Nexus AI modules correctly across the Presentation → Domain → Data layer boundary: entities, use cases, repository interfaces, repository implementations, mappers, and dependency direction verification.

## When to Use

Activate this skill when:
- Creating a new feature module or domain concept
- Adding a use case, repository interface, or repository implementation
- Designing entities or DTOs
- Mapping between data layer types and domain types
- Refactoring code that has violated layer boundaries
- Reviewing a PR for architectural correctness

## Core Rules

Refer to `01-architecture.md` for the authoritative rules. This skill provides implementation patterns for applying them.

1. **Dependency direction is one way: Presentation → Domain → Data.** Never reverse it.
2. **Domain is the center.** It depends on nothing outside itself (except standard Kotlin/minimal Android where unavoidable).
3. **Repository interfaces belong in Domain.** Implementations belong in Data.
4. **Use cases have one responsibility.** Name them as verbs: `SendMessageUseCase`, `IndexDocumentUseCase`.
5. **Mappers live at the boundary.** DTOs map to entities in Data; entities never contain DTO or Room types.
6. **Verify direction after every change.** If Data imports Domain — correct. If Domain imports Data — wrong.

## Implementation Guidance

### Layer Package Structure

```
com.nexusai.
├── presentation.chat          ← Compose, ViewModel, UiState, UiEvent
├── domain.
│   ├── entity.                ← Pure Kotlin domain models
│   ├── repository.            ← Repository interfaces
│   ├── usecase.               ← Use case classes
│   └── model.                 ← Domain-level value types
└── data.
    ├── repository.            ← Repository implementations
    ├── source.remote.         ← Network data sources
    ├── source.local.          ← Room data sources
    ├── dto.                   ← Network response models
    ├── entity.                ← Room database entities
    └── mapper.                ← DTO/DB entity → Domain entity mappers
```

### Entity (Domain)

```kotlin
// Pure Kotlin — no Room, no Retrofit, no provider SDK imports
data class Message(
    val id: String,
    val conversationId: String,
    val content: String,
    val role: MessageRole,
    val timestamp: Instant,
    val isStreaming: Boolean = false,
)

enum class MessageRole { User, Assistant, System }
```

### Repository Interface (Domain)

```kotlin
// Domain layer — no implementation details
interface MessageRepository {
    fun getMessages(conversationId: String): Flow<List<Message>>
    suspend fun saveMessage(message: Message): Result<Unit>
    suspend fun deleteMessage(messageId: String): Result<Unit>
    suspend fun clearConversation(conversationId: String): Result<Unit>
}
```

### Use Case (Domain)

```kotlin
// One responsibility, named as a verb
class SendMessageUseCase @Inject constructor(
    private val messageRepository: MessageRepository,
    private val aiOrchestration: AIOrchestration,
    private val memoryRepository: MemoryRepository,
) {
    /**
     * Sends a user message and returns a Flow of streaming AI response events.
     * Persists both the user message and the completed assistant response.
     */
    operator fun invoke(
        conversationId: String,
        text: String,
    ): Flow<AIStreamEvent> = flow {
        val userMessage = Message(
            id = generateId(),
            conversationId = conversationId,
            content = text,
            role = MessageRole.User,
            timestamp = Clock.System.now(),
        )
        messageRepository.saveMessage(userMessage).getOrThrow()
        emitAll(aiOrchestration.streamChat(conversationId, text))
    }
}
```

### Repository Implementation (Data)

```kotlin
// Data layer — knows about Room, but domain layer does not
class MessageRepositoryImpl @Inject constructor(
    private val messageDao: MessageDao,
    private val messageMapper: MessageMapper,
) : MessageRepository {

    override fun getMessages(conversationId: String): Flow<List<Message>> =
        messageDao.getMessages(conversationId)
            .map { entities -> entities.map(messageMapper::toDomain) }

    override suspend fun saveMessage(message: Message): Result<Unit> =
        runCatching { messageDao.insertMessage(messageMapper.toEntity(message)) }

    override suspend fun deleteMessage(messageId: String): Result<Unit> =
        runCatching { messageDao.deleteMessage(messageId) }

    override suspend fun clearConversation(conversationId: String): Result<Unit> =
        runCatching { messageDao.clearConversation(conversationId) }
}
```

### Mapper (Data)

```kotlin
class MessageMapper @Inject constructor() {

    fun toDomain(entity: MessageEntity): Message = Message(
        id = entity.id,
        conversationId = entity.conversationId,
        content = entity.content,
        role = MessageRole.valueOf(entity.role),
        timestamp = Instant.fromEpochMilliseconds(entity.timestampMs),
    )

    fun toEntity(domain: Message): MessageEntity = MessageEntity(
        id = domain.id,
        conversationId = domain.conversationId,
        content = domain.content,
        role = domain.role.name,
        timestampMs = domain.timestamp.toEpochMilliseconds(),
    )
}
```

### Hilt Binding (Data module)

```kotlin
@Module
@InstallIn(SingletonComponent::class)
abstract class MessageModule {

    @Binds
    @Singleton
    abstract fun bindMessageRepository(impl: MessageRepositoryImpl): MessageRepository
}
```

## Incremental Refactoring Approach

When existing code violates layer boundaries:

1. Identify the violation (e.g., ViewModel accessing Retrofit directly)
2. Define the missing abstraction (repository interface in Domain)
3. Create the implementation (repository impl in Data)
4. Wire via Hilt
5. Update the ViewModel to use the use case
6. Delete the direct dependency
7. Verify: does Domain still import nothing from Data? ✓

Never attempt to refactor everything at once. One violation at a time.

## Dependency Direction Verification Checklist

After any architectural change, verify:

- [ ] Domain module `build.gradle` has no dependency on `:data` or `:presentation` modules
- [ ] Domain classes import no Room, Retrofit, OkHttp, or provider SDK types
- [ ] Domain classes import no Compose types
- [ ] Repository interfaces are in Domain; implementations are in Data
- [ ] Mappers are in Data; they import both domain entities and data entities, but domain entities do not import data entities
- [ ] ViewModels inject use cases, not repositories directly

## Common Mistakes

| Mistake | Correct Approach |
|---|---|
| ViewModel injects a repository directly | ViewModel injects use cases; use cases inject repositories |
| Domain entity has a Room `@Entity` annotation | Separate `MessageEntity` (Data) from `Message` (Domain); use a mapper |
| Use case returns a DTO | Map DTO to domain entity in repository impl; use case returns domain type |
| Repository interface imports Retrofit `Response<T>` | Return domain types or `Result<T>` from interfaces |
| One giant `AppRepository` interface | One interface per bounded domain concept |
| Mapper lives in Domain | Mappers live in Data — they need to know both domain and data types |
| Feature modules import each other | Features communicate via shared domain types or navigation contracts only |

## Testing Guidance

- Test use cases with fake repositories (implement the repository interface in `core/testing/`)
- Test repository implementations with an in-memory Room database — no mocking needed
- Test mappers as pure functions — no infrastructure needed
- Verify that domain test targets (`src/test/`) have zero Android instrumentation dependencies
- Domain tests run on JVM only — if a domain test requires Robolectric, that is a boundary violation

## Relationship to Steering

This skill applies `01-architecture.md`. Every rule here is derived from that Steering file. If there is a conflict, `01-architecture.md` wins.
