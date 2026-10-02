# 02 — Android Development

## Language

Kotlin only. No Java in new code.

Use idiomatic Kotlin: data classes, sealed classes/interfaces, extension functions, scope functions, named arguments, default parameters.

---

## Jetpack Compose

All UI is implemented in Jetpack Compose. No XML layouts in new feature code.

Compose rules:
- Composables are stateless where possible; hoist state to ViewModel
- Use `remember` and `rememberSaveable` correctly — understand the difference
- Use `derivedStateOf` for computed state that depends on other state
- Use `LaunchedEffect` for one-shot side effects tied to the composition lifecycle
- Use `DisposableEffect` for resources that must be cleaned up
- Use `SideEffect` only for non-suspending synchronization with non-Compose code
- Avoid side effects directly inside composable function bodies
- Use `key()` when recomposing lists with stable identities
- Annotate stable data classes with `@Stable` or `@Immutable` where appropriate to aid recomposition optimization

---

## Material 3

Use Material 3 components and theming throughout. Do not mix Material 2 and Material 3.

- Use `MaterialTheme` tokens for colors, typography, and shapes — no hardcoded color values
- Support both light and dark themes
- Use dynamic color where appropriate and configured
- Follow Material 3 component guidelines for interactive states

---

## ViewModel

- One ViewModel per screen/feature (not one per composable)
- ViewModels survive configuration changes — do not hold View or Context references directly
- Use `viewModelScope` for coroutines; scope is automatically cancelled on ViewModel clearing
- Inject use cases, not repositories, directly into ViewModels
- Expose `StateFlow<UiState>` for UI state
- Expose `SharedFlow<UiEvent>` or `Channel<UiEvent>` for one-shot events (navigation, toast)

---

## Coroutines and Flow

- All async work uses coroutines. No callbacks where coroutines are possible.
- Use `suspend` functions for single-value async operations
- Use `Flow` for streams of values
- Use `StateFlow` for observable state
- Use `SharedFlow` for events (hot, multicast)
- Use structured concurrency: always launch coroutines in a defined scope
- Use `withContext(Dispatchers.IO)` for IO-bound work; never block `Dispatchers.Main`
- Use `Dispatchers.Default` for CPU-intensive work
- Cancel coroutines properly — respect lifecycle cancellation
- Use `supervisorScope` when child failures should not cancel siblings

---

## Lifecycle Awareness

- Collect Flows in a lifecycle-aware manner: use `collectAsStateWithLifecycle()` in Compose
- Do not collect Flows with `lifecycleScope.launchWhenStarted` (deprecated pattern)
- Use `repeatOnLifecycle(Lifecycle.State.STARTED)` when collecting outside Compose
- ViewModels must not hold references to Activity, Fragment, or any Context subclass that has a lifecycle shorter than the ViewModel

---

## Hilt

- Use Hilt for all dependency injection
- Use `@HiltViewModel` for ViewModels
- Use `@Singleton` sparingly and document why
- Use `@InstallIn` scopes correctly: `SingletonComponent`, `ViewModelComponent`, `ActivityComponent` as appropriate
- Define Hilt modules in the layer that owns the binding (Data layer provides repository implementations, not Domain)

---

## Room

- Define database entities in the Data layer only
- Access Room only through data sources, never directly from ViewModels or use cases
- Use `suspend` functions and `Flow` in DAOs
- Use migrations for schema changes — never use `fallbackToDestructiveMigration` in production
- Define type converters for complex types; avoid storing raw JSON blobs where structured columns are practical

---

## DataStore

- Use `Proto DataStore` for typed preferences; use `Preferences DataStore` only for simple key-value needs
- Access DataStore through a repository or data source abstraction — never directly from ViewModels
- DataStore operations are always asynchronous

---

## WorkManager

- Use WorkManager for work that must survive process death or UI lifecycle changes
- Define `Constraints` explicitly (network required, battery not low, etc.)
- Use `CoroutineWorker` for suspend-based work
- Inject dependencies into Workers using Hilt WorkManager integration (`@HiltWorker`)
- Handle `Result.success()`, `Result.failure()`, and `Result.retry()` correctly
- Do not use WorkManager for work that must complete immediately in the foreground

---

## Configuration Changes and Process Recreation

- Do not store large objects in `onSaveInstanceState` — use ViewModel for in-memory state and Room/DataStore for persistent state
- Test behavior after process recreation for flows that persist state to storage
- Use `rememberSaveable` in Compose for UI state that must survive configuration changes but not process death

---

## Performance

- Never block the main thread. No `Thread.sleep()`, `runBlocking` on main, or synchronous IO on main
- Use lazy loading and pagination for large lists (`LazyColumn` + Paging 3 where appropriate)
- Profile with Android Studio before optimizing — measure first
- Avoid unnecessary recompositions: stable state, correct keys, `@Stable`/`@Immutable` annotations

---

## Platform Boundary

- Keep Android-specific code (Context, Uri, Intent, etc.) at the outer edges of the architecture — Data layer and Presentation layer
- Domain entities and use cases should not require Android imports where avoidable
- This makes domain logic unit-testable on JVM without Android instrumentation
