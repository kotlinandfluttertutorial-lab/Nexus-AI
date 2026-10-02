# Skill: Android Development

## Purpose

Provides reusable engineering expertise for building Android features in Nexus AI using Kotlin, Jetpack, Compose, Coroutines, Flow, ViewModel, Hilt, Room, DataStore, and WorkManager.

## When to Use

Activate this skill when:
- Creating or modifying any Android component (Activity, ViewModel, Worker, DAO, etc.)
- Implementing new Compose screens or shared UI components
- Setting up Hilt modules or dependency bindings
- Working with Room database entities, DAOs, or migrations
- Working with DataStore for persistent preferences
- Scheduling background work with WorkManager
- Debugging lifecycle, configuration change, or memory issues

## Core Rules

Refer to `02-android-development.md` for the authoritative rules. This skill provides implementation guidance for applying those rules.

1. **Kotlin only.** No Java in new code.
2. **Compose for all UI.** No XML layouts.
3. **Lifecycle safety is non-negotiable.** Never hold Activity/Fragment references in ViewModels.
4. **Main thread is sacred.** Never block it. All IO and CPU work moves off main via `Dispatchers.IO` / `Dispatchers.Default`.
5. **Structured concurrency always.** Every coroutine lives in a defined scope.
6. **Hilt everywhere.** No manual DI, no service locators.

## Implementation Guidance

### ViewModel Pattern

```kotlin
@HiltViewModel
class ChatViewModel @Inject constructor(
    private val sendMessageUseCase: SendMessageUseCase,
    private val getMessagesUseCase: GetMessagesUseCase,
) : ViewModel() {

    private val _uiState = MutableStateFlow<ChatUiState>(ChatUiState.Idle)
    val uiState: StateFlow<ChatUiState> = _uiState.asStateFlow()

    private val _events = MutableSharedFlow<ChatUiEvent>()
    val events: SharedFlow<ChatUiEvent> = _events.asSharedFlow()

    fun sendMessage(text: String) {
        viewModelScope.launch {
            _uiState.value = ChatUiState.Loading
            sendMessageUseCase(text)
                .onSuccess { _uiState.value = ChatUiState.Success(it) }
                .onFailure { _uiState.value = ChatUiState.Error(it.toUiError()) }
        }
    }
}
```

### Collecting State in Compose

```kotlin
@Composable
fun ChatScreen(viewModel: ChatViewModel = hiltViewModel()) {
    val uiState by viewModel.uiState.collectAsStateWithLifecycle()

    // Collect one-shot events
    val lifecycleOwner = LocalLifecycleOwner.current
    LaunchedEffect(viewModel.events, lifecycleOwner) {
        viewModel.events
            .flowWithLifecycle(lifecycleOwner.lifecycle)
            .collect { event -> /* handle navigation etc. */ }
    }
}
```

### Room DAO Pattern

```kotlin
@Dao
interface MessageDao {
    @Query("SELECT * FROM messages WHERE conversationId = :id ORDER BY timestamp ASC")
    fun getMessages(id: String): Flow<List<MessageEntity>>

    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insertMessage(message: MessageEntity)

    @Delete
    suspend fun deleteMessage(message: MessageEntity)
}
```

### WorkManager with Hilt

```kotlin
@HiltWorker
class DocumentIndexingWorker @AssistedInject constructor(
    @Assisted context: Context,
    @Assisted workerParams: WorkerParameters,
    private val indexDocumentUseCase: IndexDocumentUseCase,
) : CoroutineWorker(context, workerParams) {

    override suspend fun doWork(): Result {
        val documentId = inputData.getString(KEY_DOCUMENT_ID) ?: return Result.failure()
        return indexDocumentUseCase(documentId)
            .fold(onSuccess = { Result.success() }, onFailure = { Result.retry() })
    }

    companion object {
        const val KEY_DOCUMENT_ID = "document_id"
    }
}
```

### DataStore Access Pattern

Always wrap DataStore behind a repository or data source — never access it directly from a ViewModel or use case.

```kotlin
class SettingsDataSource @Inject constructor(
    private val dataStore: DataStore<Preferences>
) {
    val aiProvider: Flow<String> = dataStore.data.map { prefs ->
        prefs[KEY_AI_PROVIDER] ?: DEFAULT_PROVIDER
    }

    suspend fun setAiProvider(provider: String) {
        dataStore.edit { it[KEY_AI_PROVIDER] = provider }
    }

    private companion object {
        val KEY_AI_PROVIDER = stringPreferencesKey("ai_provider")
        const val DEFAULT_PROVIDER = "gemini"
    }
}
```

## Common Mistakes

| Mistake | Correct Approach |
|---|---|
| `viewModel.someFlow.collect { }` without lifecycle awareness | Use `collectAsStateWithLifecycle()` or `repeatOnLifecycle` |
| Storing `Context` in a ViewModel property | Inject `ApplicationContext` via Hilt if truly needed, or move to Data layer |
| Calling `runBlocking` in a coroutine scope | Use `suspend` all the way; never block inside a coroutine |
| Accessing Room DAO directly from ViewModel | Route through data source → repository → use case |
| `GlobalScope.launch` | Use `viewModelScope`, `lifecycleScope`, or injected `CoroutineScope` |
| `@Singleton` on everything | Scope to the smallest appropriate component |
| `fallbackToDestructiveMigration()` | Write proper Room migrations |
| Using `launchWhenStarted` | Use `repeatOnLifecycle(STARTED)` |
| Not handling `CancellationException` | Always rethrow `CancellationException` if caught |
| Mutable `UiState` exposed directly | Expose `StateFlow` via `asStateFlow()`; mutate via private `MutableStateFlow` |

## Testing Guidance

- Unit-test ViewModels with `TestCoroutineScheduler` and fake use cases — no real Android runtime needed
- Unit-test use cases with fake repositories on JVM
- Unit-test Room DAOs with an in-memory Room database (`Room.inMemoryDatabaseBuilder`)
- Unit-test WorkManager workers by testing the `doWork()` logic directly with fakes
- Use `collectAsStateWithLifecycle()` in Compose tests with `composeTestRule`
- Inject test doubles via Hilt testing APIs (`@UninstallModules`, `@BindValue`)

## Relationship to Steering

This skill applies `02-android-development.md`. If this skill's guidance conflicts with that Steering file, Steering takes precedence. Raise the conflict before proceeding.
