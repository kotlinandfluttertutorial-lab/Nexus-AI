# Skill: Jetpack Compose

## Purpose

Provides reusable expertise for building Compose UI in Nexus AI: screens, components, state management, navigation, theming, accessibility, and testing.

## When to Use

Activate this skill when:
- Building or modifying any Compose screen or component
- Designing UiState and UiEvent structures
- Implementing navigation between screens
- Creating shared/reusable UI components in `core/ui/`
- Adding Compose UI tests
- Debugging recomposition, layout, or theming issues

## Core Rules

Refer to `02-android-development.md` for the authoritative Compose rules. This skill provides implementation patterns.

1. **Stateless composables.** Hoist all state to ViewModel or the nearest appropriate owner.
2. **Unidirectional data flow.** State flows down, events flow up.
3. **No business logic in composables.** Composables render state and emit events only.
4. **Material 3 only.** No Material 2 components. No hardcoded colors or dimensions.
5. **Accessible by default.** Provide `contentDescription` on all meaningful icons and images.

## Implementation Guidance

### UiState Design

Model every screen state as an immutable sealed type:

```kotlin
sealed interface ChatUiState {
    data object Idle : ChatUiState
    data object Loading : ChatUiState
    data class Success(
        val messages: List<MessageUi>,
        val inputText: String = "",
        val isStreaming: Boolean = false,
    ) : ChatUiState
    data class Error(val message: UiText) : ChatUiState
}
```

UiEvents for one-shot side effects:

```kotlin
sealed interface ChatUiEvent {
    data class NavigateToDetail(val messageId: String) : ChatUiEvent
    data class ShowSnackbar(val message: UiText) : ChatUiEvent
    data object ScrollToBottom : ChatUiEvent
}
```

### Screen Structure

```kotlin
@Composable
fun ChatScreen(
    viewModel: ChatViewModel = hiltViewModel(),
    onNavigateUp: () -> Unit,
) {
    val uiState by viewModel.uiState.collectAsStateWithLifecycle()

    // One-shot events
    LaunchedEffect(Unit) {
        viewModel.events.collect { event ->
            when (event) {
                is ChatUiEvent.NavigateToDetail -> { /* handle */ }
                is ChatUiEvent.ShowSnackbar -> { /* handle */ }
                ChatUiEvent.ScrollToBottom -> { /* handle */ }
            }
        }
    }

    ChatContent(
        uiState = uiState,
        onSendMessage = viewModel::sendMessage,
        onInputChange = viewModel::onInputChange,
    )
}

// Stateless content composable — testable in isolation
@Composable
private fun ChatContent(
    uiState: ChatUiState,
    onSendMessage: (String) -> Unit,
    onInputChange: (String) -> Unit,
) {
    when (uiState) {
        is ChatUiState.Idle -> IdleContent()
        is ChatUiState.Loading -> LoadingContent()
        is ChatUiState.Success -> SuccessContent(uiState, onSendMessage, onInputChange)
        is ChatUiState.Error -> ErrorContent(uiState.message)
    }
}
```

### Material 3 Theming

```kotlin
// Always use theme tokens — never hardcode
Text(
    text = message.content,
    style = MaterialTheme.typography.bodyMedium,
    color = MaterialTheme.colorScheme.onSurface,
)

Surface(
    color = MaterialTheme.colorScheme.surfaceVariant,
    shape = MaterialTheme.shapes.medium,
) { ... }
```

Define the app theme once in `core/ui/`:

```kotlin
@Composable
fun NexusAITheme(
    darkTheme: Boolean = isSystemInDarkTheme(),
    dynamicColor: Boolean = true,
    content: @Composable () -> Unit,
) {
    val colorScheme = when {
        dynamicColor && Build.VERSION.SDK_INT >= Build.VERSION_CODES.S -> {
            if (darkTheme) dynamicDarkColorScheme(LocalContext.current)
            else dynamicLightColorScheme(LocalContext.current)
        }
        darkTheme -> DarkColorScheme
        else -> LightColorScheme
    }
    MaterialTheme(colorScheme = colorScheme, typography = NexusTypography, content = content)
}
```

### Reusable Component Pattern

```kotlin
// Define test tags as constants on the component
object MessageBubbleTestTags {
    const val Root = "MessageBubble_Root"
    const val Content = "MessageBubble_Content"
}

@Composable
fun MessageBubble(
    message: MessageUi,
    modifier: Modifier = Modifier,
) {
    Surface(
        modifier = modifier.semantics { contentDescription = message.contentDescription }
            .testTag(MessageBubbleTestTags.Root),
        // ...
    ) {
        Text(
            text = message.content,
            modifier = Modifier.testTag(MessageBubbleTestTags.Content),
        )
    }
}

// Always provide a @Preview
@Preview(showBackground = true)
@Composable
private fun MessageBubblePreview() {
    NexusAITheme {
        MessageBubble(message = PreviewData.sampleMessage)
    }
}
```

### Compose Effects — Correct Usage

| Effect | When to use |
|---|---|
| `LaunchedEffect(key)` | Start a coroutine when key changes; cancel/restart on recomposition with new key |
| `DisposableEffect(key)` | Setup/teardown with `onDispose` — for listeners, callbacks |
| `SideEffect` | Sync non-Compose state that must run on every successful recomposition |
| `derivedStateOf` | Expensive computation derived from other state — memoized |
| `remember { }` | Survive recomposition, reset on leave |
| `rememberSaveable { }` | Survive recomposition AND configuration changes |

### Stable State for Performance

```kotlin
@Immutable
data class MessageUi(
    val id: String,
    val content: String,
    val isFromUser: Boolean,
    val timestamp: String,
)

@Stable
class ChatInputState(initialText: String = "") {
    var text by mutableStateOf(initialText)
}
```

## Common Mistakes

| Mistake | Correct Approach |
|---|---|
| Business logic in a composable | Move to ViewModel or use case |
| `mutableStateOf` in ViewModel | Use `MutableStateFlow` in ViewModel; `collectAsStateWithLifecycle()` in Compose |
| `collectAsState()` without lifecycle | Use `collectAsStateWithLifecycle()` |
| Hardcoded colors `Color(0xFF...)` | Use `MaterialTheme.colorScheme.*` tokens |
| Missing `contentDescription` on icons | Always provide for meaningful interactive elements |
| Sharing ViewModel across unrelated screens | One ViewModel per screen; share data via repositories |
| Triggering side effects in composition body | Use `LaunchedEffect`, `DisposableEffect`, or `SideEffect` |
| No test tags | Define `TestTag` constants on every screen/component |
| Missing `@Preview` | Add at least one preview per public composable |
| Large monolithic composable | Extract into smaller focused composables |

## Testing Guidance

```kotlin
@get:Rule
val composeTestRule = createComposeRule()

@Test
fun `ChatContent shows loading indicator when state is Loading`() {
    composeTestRule.setContent {
        NexusAITheme {
            ChatContent(
                uiState = ChatUiState.Loading,
                onSendMessage = {},
                onInputChange = {},
            )
        }
    }
    composeTestRule.onNodeWithTag(ChatScreenTestTags.LoadingIndicator).assertIsDisplayed()
}

@Test
fun `send button calls onSendMessage with input text`() {
    var capturedMessage = ""
    composeTestRule.setContent {
        NexusAITheme {
            ChatContent(
                uiState = ChatUiState.Success(messages = emptyList()),
                onSendMessage = { capturedMessage = it },
                onInputChange = {},
            )
        }
    }
    composeTestRule.onNodeWithTag(ChatScreenTestTags.Input).performTextInput("Hello")
    composeTestRule.onNodeWithTag(ChatScreenTestTags.SendButton).performClick()
    assertThat(capturedMessage).isEqualTo("Hello")
}
```

Always test stateless `Content` composables in isolation — pass state directly, no ViewModel needed in the test.

## Relationship to Steering

This skill applies `02-android-development.md` (Compose section) and `01-architecture.md` (UI Architecture Pattern). Steering takes precedence over any guidance here.
