## compose-skill

> |


## I. Composable Structure

### 1. Function Separation

Keep composable functions under ~50 lines of meaningful code (ignoring empty lines, braces-only lines).
Separate functions by responsibility: layout, input handling, dialogs, styling.
Mixing responsibilities is fine for small functions.

**Recomposition Scope**
Composable functions create a recomposition scope tied to their content. Deliberately creating small functions can isolate frequently-changing values into narrow scopes. Conversely, avoid unnecessary extraction when you only want readability (e.g. fixing detekt issues) — it creates an unneeded scope.

A single empty line can visually separate layout sections instead of comments — apply sparingly and only where readability suffers.

**Example — good separation for a large function:**
```kotlin
@Composable
private fun PostList(
    postsFeed: PostsFeed,
    favorites: Set<String>,
    showExpandedSearch: Boolean,
    onPostTapped: (postId: String) -> Unit,
    onToggleFavorite: (String) -> Unit,
    modifier: Modifier = Modifier,
    contentPadding: PaddingValues = PaddingValues(0.dp),
    state: LazyListState = rememberLazyListState(),
    searchInput: String = "",
    onSearchInputChanged: (String) -> Unit,
) {
    LazyColumn(
        modifier = modifier,
        contentPadding = contentPadding,
        state = state,
    ) {
        if (showExpandedSearch) {
            item {
                HomeSearch(
                    Modifier.padding(horizontal = 16.dp),
                    searchInput = searchInput,
                    onSearchInputChanged = onSearchInputChanged,
                )
            }
        }

        item { PostListTopSection(postsFeed.highlightedPost, onPostTapped) }

        if (postsFeed.recommendedPosts.isNotEmpty()) {
            item {
                PostListSimpleSection(postsFeed.recommendedPosts, onPostTapped, favorites, onToggleFavorite)
            }
        }

        if (postsFeed.popularPosts.isNotEmpty() && !showExpandedSearch) {
            item { PostListPopularSection(postsFeed.popularPosts, onPostTapped) }
        }

        if (postsFeed.recentPosts.isNotEmpty()) {
            item { PostListHistorySection(postsFeed.recentPosts, onPostTapped) }
        }
    }
}
```
`PostList` handles layout and delegates content rendering to child composables.

**Example — small function with mixed responsibilities (acceptable):**
```kotlin
@Composable
fun JetchatIcon(contentDescription: String?, modifier: Modifier = Modifier) {
    val semantics = if (contentDescription != null) {
        Modifier.semantics {
            this.contentDescription = contentDescription
            this.role = Role.Image
        }
    } else Modifier

    Box(modifier = modifier.then(semantics)) {
        Icon(painterResource(R.drawable.ic_jetchat_back), null, tint = MaterialTheme.colorScheme.primaryContainer)
        Icon(painterResource(R.drawable.ic_jetchat_front), null, tint = MaterialTheme.colorScheme.primary)
    }
}
```

### 2. Default Values for Arguments

1. Expose `Modifier` as the first optional argument with default `Modifier` for any composable with a single top-level composable call. Exception: screen-level composables don't need exposed `Modifier`.
2. No defaults for core data (e.g. `UserProfile` without `name` is useless).
3. No defaults for primary actions (e.g. `onClick` in `Button`).
4. No defaults in single-use private functions.
5. Optional arguments, secondary actions, and styling should have defaults in public/internal composables or in private composables already reused in the file.

```kotlin
// ✅ Correct
@Composable
fun AddTransactionScreen(
    onBack: () -> Unit = {},
    transactionId: Long? = null,
    viewModel: AddTransactionViewModel = koinViewModel(parameters = { parametersOf(transactionId) }),
) { /* Body */ }

@Composable
private fun DescriptionField(  // Used only once — no defaults
    state: TextFieldState,
    error: StringResource?,
    enabled: Boolean,
) { /* Body */ }

// ❌ Incorrect
@Composable
private fun TypeToggle(  // Single-use: `enabled` should not have a default (rule #4)
    isExpense: Boolean,
    enabled: Boolean = true,
    onToggle: (Boolean) -> Unit,
) { /* Body */ }

@Composable
internal fun UserScreen(
    viewModel: ViewModel = koinViewModel(),
    onNavigateBack: () -> Unit,  // Secondary action: should default to {} (rule #5)
) { /* Body */ }
```

### 3. Reusing Components

Check related directories for existing components before creating new ones. Components serve various purposes: common parameter configuration (e.g. `Header` = styled `Text`), layout patterns (e.g. `InfoCard`), or state/logic wrappers (e.g. animation containers).

**Component location:**
- **App-wide**: shared module or `components` package — used across the whole app.
- **Feature-wide**: `components` package in current module or a descriptively named file (e.g. `PostCards.kt`).
- **Screen-level**: same file as other screen code, or a nearby file if the screen file is large.

**Component extraction:**
- Look for recurring patterns with minor variations that can be parameterized.
- Place extracted components at the top-most common location for all usages.
- Expose `modifier: Modifier = Modifier` as the first optional parameter when the component is used in varied layout contexts.
- When unsure about placement or whether extraction makes sense, ask.

```kotlin
// App-wide component: no modifier (top-level layout wrapper)
@Composable
fun JetchatDrawer(
    drawerState: DrawerState = rememberDrawerState(initialValue = Closed),
    selectedMenu: String,
    onProfileClicked: (String) -> Unit,
    onChatClicked: (String) -> Unit,
    content: @Composable () -> Unit,
) { /* ModalNavigationDrawer wrapping ModalDrawerSheet */ }

// Reusable element: exposes modifier for flexible layout placement
@Composable
internal fun PostImage(post: Post, modifier: Modifier = Modifier) {
    Image(
        painter = painterResource(post.imageThumbId),
        contentDescription = null,
        modifier = modifier.size(40.dp, 40.dp).clip(MaterialTheme.shapes.small),
    )
}
```

### 4. Dimensions

Use a single source of dimension constants. Look for an existing one in the project; if absent, prompt the user to create one. All spacing values should be integer multiples of the base spacing (or half). Base value is typically `4.dp`.

```kotlin
object AppDimens {
    val spacing1x = 4.dp
    val spacing2x = 8.dp
    // Add multiplicatives as needed: 4x, 6x, 8x, 10x, 12x, 16x
}
```

### 5. Composable Constants

Constants for UI elements (custom dimensions, sizes, magic numbers) should use UpperCamelCase.
Keep them as top-level `private` properties or group in a `ComponentNameTokens` object.
Avoid placing feature-specific data inside general components — put it in UI state or ViewModel instead.

```kotlin
private val RadioButtonPadding = 2.dp
private val RadioButtonDotSize = 12.dp
private val RadioStrokeWidth = 2.dp
```

### 6. String Resources

Always use string resources (e.g., `stringResource(id = R.string.some_text)`) for all user-visible text, including formatted numeric values. **Never** hardcode string literals in UI code. This is a strict rule to ensure proper internationalization and code maintainability.

---

## II. State Management

### 1. State Hoisting

Hoist UI state to the lowest common ancestor between all composables that read and write it.
Keep state closest to where it is consumed. For complex cases, expose immutable state and events from the state owner. Use state holder classes (e.g. `LazyListState`) for complex UI logic. ViewModel is the highest hoisting level.

For compose-level state, pass as a lambda (to read one property) or as `State<T>` for special state objects. Default to the lambda approach.

```kotlin
@Composable
private fun ConversationScreen(/*...*/) {
    val scope = rememberCoroutineScope()
    val lazyListState = rememberLazyListState()

    MessagesList(messages, lazyListState)

    UserInput(
        onMessageSent = {
            scope.launch { lazyListState.scrollToItem(0) }
        },
    )
}

@Composable
private fun MessagesList(
    messages: List<Message>,
    lazyListState: LazyListState = rememberLazyListState(),
) {
    LazyColumn(state = lazyListState) {
        items(messages, key = { it.id }) { item -> Message(/*...*/) }
    }
    val scope = rememberCoroutineScope()
    JumpToBottom(onClicked = {
        scope.launch { lazyListState.scrollToItem(0) }
    })
}
```

**Lambda state passing** — read state as late as possible, ideally at the edge of interaction with Compose APIs:
```kotlin
@Composable
fun TopLevel() {
    val localState = remember { mutableStateOf("") }
    SubLevel({ localState.value })
}

@Composable
fun SubLevel(provider: () -> String) {
    Card {
        Text(provider()) // Only Text recomposes here
    }
}
```

You can wrap state consumers into small composable functions to create separate recomposition scopes. Some components (those accepting composable lambdas) create their own internal scope. Be careful: inline components (`Box`, `Row`, `Column`) do **not** create a separate scope.

### 2. Property Drilling

Pass only the properties a composable actually uses. Avoid passing entire state objects just to read one field.

```kotlin
// ✅ Good
@Composable
fun CardHeader(header: String) { Text(header) }

// ❌ Bad — CardHeader only needs `header`, not the entire CardState
@Composable
fun CardHeader(card: CardState) { Text(card.header) }
```

### 3. ViewModel Passing

Pass the ViewModel only to the topmost composable. Inside it, collect state and pass data and callbacks downward. Use function references for callbacks (e.g. `viewModel::onPlayEpisodes`).

```kotlin
@Composable
fun QueueScreen(
    onPlayButtonClick: () -> Unit,
    onEpisodeItemClick: (PlayerEpisode) -> Unit,
    onDismiss: () -> Unit,
    modifier: Modifier = Modifier,
    queueViewModel: QueueViewModel = hiltViewModel(),
) {
    val uiState by queueViewModel.uiState.collectAsStateWithLifecycle()

    QueueScreen(
        uiState = uiState,
        onPlayButtonClick = onPlayButtonClick,
        onPlayEpisodes = queueViewModel::onPlayEpisodes,
        onDeleteQueueEpisodes = queueViewModel::onDeleteQueueEpisodes,
        modifier = modifier,
        onEpisodeItemClick = onEpisodeItemClick,
        onDismiss = onDismiss,
    )
}
```

### 4. derivedStateOf

Common misconceptions exist around `derivedStateOf`. Use it when:
- The number of state changes exceeds the desired number of UI updates (e.g. scroll position → show/hide button).
- Caching results of expensive calculations.

Do **not** use it to access properties in classes or to combine 2+ states into 1 (in most cases).
It introduces performance overhead, so only add it when justified. You can ask the user to check recomposition count in Layout Inspector if you suspect (but aren't certain) that `derivedStateOf` is needed.

`derivedStateOf` tracks Compose state values automatically. For non-state data, pass it as a `remember` key:
```kotlin
val dState = remember(key1) {
    derivedStateOf { /* computation using key1 */ }
}
```

```kotlin
@Composable
fun MessageList(messages: List<Message>) {
    Box {
        val listState = rememberLazyListState()
        LazyColumn(state = listState) { /* ... */ }

        val showButton by remember {
            derivedStateOf { listState.firstVisibleItemIndex > 0 }
        }
        AnimatedVisibility(visible = showButton) { ScrollToTopButton() }
    }
}
```

---

## III. ViewModel & Data Flow

### 1. UI State Rules

**MVI Style (default)**
Use a single sealed interface as the source of UI state. This is the default for Compose. If the project architecture is unclear, prefer MVI even if docs mention MVVM.

```kotlin
sealed interface MyUiState {
    data object Error : MyUiState
    data object Loading : MyUiState
    data class Success(val name: String, val surname: String) : MyUiState
}

class MyViewModel : ViewModel() {
    val state: StateFlow<MyUiState>
    // No additional public state variables
}
```

**MVVM Style (legacy/compatibility)**
Multiple smaller specialized states. Use only for compatibility with existing projects.

**Adding new state to a sealed interface:**
1. If the state **cannot co-exist** with others and has **different data** → add a new sealed member.
2. If the state **cannot co-exist** but **shares most data** → use a single data class with a status `Enum` (or boolean flags for 2-3 states max).
3. If the state **can co-exist** with any existing state → add it as a field in the relevant data class.

Additional guidance:
- Use `data object` for states without fields.
- Remove the sealed interface if it has only one implementation.
- All states should implement the sealed interface (directly or indirectly).
- Avoid nesting one state inside another as a field — extract an intermediate data class or merge the states.

```kotlin
// Different data per state (backend-powered screens)
sealed interface TransactionScreenUiState {
    data object Error : TransactionScreenUiState
    data object Loading : TransactionScreenUiState
    data class Success(val data: TransactionsData) : TransactionScreenUiState
}

// Shared data with status flags (input forms, ≤3 states)
data class FormUiState(
    val form: Form,
    val isLoading: Boolean = false,
    val isSuccess: Boolean = false,
)

// Shared data with status enum (4+ states)
data class ComplexFormUiState(
    val form: Form,
    val status: Status = Status.IDLE,
)
```

### 2. State Transfer

**StateFlow (default)** — for imperative VMs using suspend functions:
```kotlin
private val _state = MutableStateFlow<UiState>(initialValue)
val state = _state.asStateFlow()
```

**Flow pipeline with stateIn** — for reactive VMs observing data sources:
```kotlin
val state = combinedDataFlow.stateIn(
    viewModelScope,
    started = SharingStarted.WhileSubscribed(5000),
    initialValue = initialValue,
)
```
No second variable needed — `stateIn` returns immutable `StateFlow<T>`. Both `Eagerly` and `Lazily` are acceptable for `started`. If using `WhileSubscribed`, consider making the delay a module-wide constant.

### 3. Events

Using separate flows, channels, or shared flows for one-off events from ViewModel to Compose is an **antipattern**. Each event should result in an update to **state**.

```kotlin
@Composable
fun AddTransactionScreen(
    onBack: () -> Unit = {},
    transactionId: Long? = null,
    viewModel: AddTransactionViewModel = koinViewModel(parameters = { parametersOf(transactionId) }),
) {
    val state by viewModel.state.collectAsStateWithLifecycle()
    val snackbarHostState = remember { SnackbarHostState() }

    LaunchedEffect(state.saveStatus) {
        when (val status = state.saveStatus) {
            is SaveStatus.Success -> {
                onBack()
                viewModel.clearSaveStatus()
            }
            is SaveStatus.Error -> {
                snackbarHostState.showSnackbar(getString(status.message))
                viewModel.clearSaveError()
            }
            else -> Unit
        }
    }
}
```

**Why clear-last matters:** The state clear function (e.g. `viewModel.clearSaveStatus()`) should always be called **after** all actions (navigation, snackbar, etc.). If you clear the event state *before* performing the action, the state update triggers recomposition, which can restart the `LaunchedEffect` with the cleared state — causing the action (like navigation) to be lost.

**LaunchedEffect guidelines:**
- Place the effect where the driving state lives, not where the output (snackbar, navigator) is used.
- If you have `State<T>` or lambda providers, compute values from them inside the effect body, not outside.
- Use specific state properties as keys (e.g. `state.saveStatus`) rather than the whole `state` to minimize effect restarts.

```kotlin
// ❌ Antipatterns for events:
private val _event = Channel<Event>(Channel.CONFLATED)
private val _event = MutableSharedFlow()
private val event = eventProducerFlow.shareIn(/* ... */)
```

### 4. ViewModel Parameters

When a ViewModel needs data at startup (e.g. a transaction ID), pass it as a **constructor argument**. Avoid creating an init function called via `LaunchedEffect` — it leads to unnecessary state changes and recomposition during initialization.

```kotlin
// ✅ Good — constructor parameter
@Composable
fun TransactionDetailScreen(
    transactionId: Long,
    viewModel: TransactionDetailViewModel = koinViewModel { parametersOf(transactionId) },
) { /* ... */ }

// ❌ Bad — init via LaunchedEffect
@Composable
fun TransactionDetailScreen(
    transactionId: Long,
    viewModel: TransactionDetailViewModel = koinViewModel(),
) {
    LaunchedEffect(transactionId) {
        viewModel.init(transactionId) // Causes unnecessary recomposition
    }
}
```

### 5. UI-Layer Enums

When a UI-layer enum maps each entry to a distinct UI property (label, icon, color, etc.), prefer storing those properties as **constructor parameters** rather than mapping them via `when` expressions. This reduces boilerplate, ensures exhaustiveness at compile time, and keeps the mapping co-located with the enum definition.

Use `@Composable` extension functions for properties that require a Compose context (e.g. `stringResource`).

```kotlin
// Good — properties on enum entries
enum class HomeRange(
    val label: StringResource,
) {
    ONE_MONTH(Res.string.home_range_1m),
    THREE_MONTHS(Res.string.home_range_3m),
    SIX_MONTHS(Res.string.home_range_6m),
    ONE_YEAR(Res.string.home_range_1y);
}

@Composable
private fun HomeRange.text(): String = stringResource(label)

// Bad — separate `when` mapping for each property
enum class HomeRange {
    ONE_MONTH, THREE_MONTHS, SIX_MONTHS, ONE_YEAR;
}

@Composable
private fun HomeRange.label(): String = when (this) {
    HomeRange.ONE_MONTH -> stringResource(Res.string.home_range_1m)
    HomeRange.THREE_MONTHS -> stringResource(Res.string.home_range_3m)
    HomeRange.SIX_MONTHS -> stringResource(Res.string.home_range_6m)
    HomeRange.ONE_YEAR -> stringResource(Res.string.home_range_1y)
}
```

This applies only to enums in the UI layer (e.g. enums defined in or used by ViewModels, UI state, or composables). Domain-layer enums should remain simple if they have no UI-specific properties.

---

## IV. Collections & Stability

Most collections are considered unstable by the Compose compiler, which prevents skipping recomposition. Three mitigation approaches exist, in priority order:

**1. Stability config file**
A text file (typically `stability_config.conf`) that tells the Compose compiler to treat certain types as stable.
- *Detect:* Look for `stabilityConfigurationFiles` or `stabilityConfigurationFile` in Gradle build files.
- *Action:* Add the full collection type name (e.g. `kotlin.collections.List`) to this file.

**2. Immutable collections library**
`org.jetbrains.kotlinx:kotlinx-collections-immutable` provides collection types registered as immutable in Compose.
- *Detect:* Check dependencies for the library.
- *Action:* Add the dependency if missing; use `ImmutableList`, `ImmutableSet`, etc. in UI-facing data classes.

**3. Stable wrapper**
For cases where adding dependencies or modifying build files is impractical (large enterprise projects, open-source contributions):
```kotlin
@JvmInline
@Stable
value class StableSet<T>(private val set: Set<T>) : Set<T> by set

fun <T> Set<T>.stable() = StableSet(this)
```

**Choosing an approach:** Prefer stability config > immutable collections > stable wrappers. If neither config nor library is detected in the project, ask the user which approach they prefer before proceeding.

---

## V. Common UI Patterns

For detailed guidance on common Compose UI patterns, read the relevant reference file:

| Pattern | File | When to read |
|---------|------|--------------|
| Text inputs | `references/text_inputs.md` | Implementing forms, text fields, input validation |
| Dialogs | `references/dialogs.md` | Adding confirmation dialogs, VM-driven or UI-driven dialogs |
| Lists | `references/lists.md` | Implementing LazyColumn, LazyRow, or any scrollable list |
| Selection Input | `references/selection_input.md` | Implementing single or multiple selection views |
---

## VI. Workflow

### 1. Preliminary Research
- Find existing custom components in the project.
- Determine which collections stability approach the project uses.

### 2. Action
- Perform the action requested by the user.
- During initial implementation, spawn research sub-agents to find similar components or patterns in the project for later refactoring. For example: if you write a background color calculation for a swipeable list, search for similar functions elsewhere. This information will be needed in the review step.

### 3. Review
- Verify the code matches requirements and these guidelines.
- Analyze research results to identify possible component extractions.
- If uncertain about a refactoring, ask the user or note it as a follow-up rather than making the change.

---
> Source: [HornedHeck/compose-skill](https://github.com/HornedHeck/compose-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
