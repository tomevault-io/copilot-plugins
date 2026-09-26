## project-organizer

> > All examples are taken verbatim from this codebase.

# Conventions

> All examples are taken verbatim from this codebase.

## Language

- **Kotlin only.** There are zero `.java` files in `src/`. New code must be
  Kotlin.
- Target JVM is **21** (`build.gradle.kts:69-72`).

## Naming

| Element | Style | Example |
| --- | --- | --- |
| Package | lowercase, no underscores | `io.github.artemzuev.projectorganizer.ui.welcome` |
| Class / object | UpperCamelCase | `ProjectCatalogPanel`, `IdUtils`, `GroupWatcherService` |
| Function | lowerCamelCase | `notifyCatalogChanged`, `discoverRecursive` |
| Property | lowerCamelCase | `pendingFocusId`, `autoImportEnabled` |
| Local val/var | lowerCamelCase | `val normalizedPath = …` |
| Test class | `<ClassUnderTest>Test` | `CatalogManagerTest`, `AutoDiscoverServiceTest` |
| Bundle key | dot-segmented | `auto.import.notification.title`, `settings.group.watcher` |
| XML tag | UpperCamelCase | `@Tag("ProjectCatalogState")`, `@Tag("ManagedProject")` |

`MyMessageBundle` keeps the historical IntelliJ Plugin Template name — do
not rename it; tests, plugin.xml `<resource-bundle>`, and
`@PropertyKey(resourceBundle = BUNDLE)` references depend on it.

## File Organization

- **One top-level class per file.** Helpers / nested data classes that are
  tightly coupled may live alongside (e.g. `AutoDiscoverService.kt` also
  hosts `ProjectCategory`, `ClassificationResult`, `ProjectPattern`,
  `PackageJsonAnalysis`, `ProjectClassifier`).
- Companion objects go at the **bottom** of the class:
  ```kotlin
  class CatalogManager(...) {
      // public API
      companion object {
          private val LOG = Logger.getInstance(CatalogManager::class.java)
      }
  }
  ```
- Extension functions live next to the type they extend, or in the file of
  the consuming class when local (e.g. `ProjectCatalogPanel`).

## Error Handling

There is **no custom exception hierarchy**. Failure modes are handled with:

1. **`try { … } catch (e: Exception) { LOG.warn(…) }`** for non-fatal I/O.
   See `GroupWatcherService.runDebouncedScan`, `ProjectOpenActivity.checkAutoImport`.
2. **Early-return / null-return** for missing entities. Mutators on
   `CatalogManager` do `state.projects[id] ?: return` rather than throwing —
   the catalog is the user's data, deletes can race with edits.
3. **`require` is rare and only used in tests.** Production code is
   permissive by design.
4. **No `Result<T>` / `Either<L,R>`.** The codebase relies on Kotlin's
   nullable types and try/catch.

DO

```kotlin
try {
    LocalFileSystem.getInstance().removeWatchedRoots(watchRequests)
} catch (e: Exception) {
    LOG.warn("GroupWatcher: failed to unregister watch roots", e)
}
```

DON'T — fail loudly on user data:

```kotlin
// WRONG
val project = state.projects[id] ?: error("project missing")
```

## Logging

```kotlin
class Foo {
    // ...
    companion object {
        private val LOG = Logger.getInstance(Foo::class.java)
    }
}
```

- Always use `com.intellij.openapi.diagnostic.Logger`. Never `println`,
  `kotlin.io`, or `System.out`.
- Place the `LOG` field in a private companion at the bottom of the class.
- Keep messages prefixed by the service name in
  long-lived background services, e.g. `LOG.info("GroupWatcher: registered …")`.

## Localization

- Every user-facing string MUST go through `MyMessageBundle.message("key", …)`.
- New keys are added simultaneously to all four resource files:
  `src/main/resources/messages/MyMessageBundle.properties`, `_zh`, `_ja`, `_ko`.
- `BundleParityTest` fails the build if any key is missing in any locale.
- ZH / JA / KO keys that ship machine-translated MUST be preceded by a
  `# review` comment line. This convention is greppable
  (`grep -c '^# review' MyMessageBundle_zh.properties`) and flags keys
  awaiting native-speaker QA. EN keys are never marked `# review`.

## Persistence Files

| File | Owner | Content |
| --- | --- | --- |
| `project-catalog.xml` | `ProjectCatalogService` | Project tree, groups, display prefs |
| `project-tasks.xml` | `TaskCatalogService` | Tasks, time entries, tracker config |

Both are application-level (`RoamingType.DEFAULT`) and synchronize via
IntelliJ Settings Sync. Each has its own `PersistentStateComponent` and its
own MessageBus topic (`ProjectCatalogListener.TOPIC` / `TaskCatalogListener.TOPIC`).

## Threading

- IntelliJ enforces **EDT for UI mutations**. Use
  `ApplicationManager.getApplication().invokeLater { … }` (or
  `invokeAndWait { … }`) when mutating UI from a background thread.
- `GroupWatcherService.runOnEdt { … }` shows the right pattern for "EDT or
  inline".
- Background work goes on
  `AppExecutorUtil.getAppScheduledExecutorService()`. Do not spawn raw
  `Thread`s.
- `ProjectOpenActivity.execute` is `suspend`. It immediately re-enters
  `invokeLater { … }` to do its work on the EDT.

## IntelliJ-Platform Patterns

| Pattern | When to use | Example |
| --- | --- | --- |
| `@Service(Service.Level.APP)` | App-wide singleton, no constructor params | `ProjectCatalogService`, `GroupWatcherService` |
| `PersistentStateComponent<T>` | Persisted application state | `ProjectCatalogService` |
| `MessageBus` topic | Decoupled change broadcast | `ProjectCatalogListener.TOPIC` |
| `ProjectActivity` | Hook into project open | `ProjectOpenActivity` |
| `ToolWindowFactory + DumbAware` | Tool window that survives indexing | `ProjectCatalogToolWindowFactory` |
| `DialogWrapper` | All modal dialogs | `EditProjectDialog`, `EditGroupDialog`, … |
| `BoundConfigurable` + Kotlin DSL `panel { … }` | Settings page | `ProjectOrganizerConfigurable` |
| `AnAction, DumbAware` | Every action | `QuickOpenProjectAction`, `AddAction`, … |
| `AsyncFileListener` | React to VFS events without blocking | `GroupWatcherService.WatchedRootsListener` |
| `ProgressIndicator` + `ProgressManager.run(...)` | Long scans | `AutoDiscoverDialog` (via `discoverRecursive`) |

## State Mutation

There is exactly one rule:

> Every state mutation goes through `CatalogManager`, and every public
> `CatalogManager` method ends with `onChanged()` or `fireFocus()`.

Bulk methods (`bulkAdd`, `replaceCatalog`, `mergeCatalog`) deliberately fire
**one** event per logical operation. Don't loop over `addProject()` — use
`bulkAdd` so the tree rebuilds once.

## Path Comparison

Always go through `PathUtils`:

```kotlin
PathUtils.normalizePath(path)         // toRealPath() with lowercase fallback
PathUtils.isSamePath(a, b)            // canonical equality
PathUtils.isValidPath(maybePath)      // null-safe Files.exists
```

- The watcher dedups VFS events with `normalizePath`.
- `ProjectOpener.findOpenProject` compares with `isSamePath`.
- The auto-import balloon checks against `state.dismissedAutoImportPaths`
  using `normalizePath`.

## Kotlin Idioms in Use

- **Nullable types over `Optional`.**
- **Sequences for big walks** (the BFS in `discoverRecursive` uses an
  explicit `LinkedList` queue rather than a sequence to keep cancellation
  cheap).
- **`fun interface`** for single-method listeners (`ProjectCatalogListener`).
- **`object` for singletons** without state (`AutoDiscoverService`,
  `ProjectClassifier`, `ProjectOpener`, `MyMessageBundle`, `PathUtils`,
  `IdUtils`, `TerminalExecutionHelper`, `WatchedGroupNotifications`,
  `CatalogImportExportService`).
- **Data classes for transient bundles** (`DiscoveredProject`,
  `ClassificationResult`, `AddedItems`).
- **`@Suppress("SpellCheckingInspection")`** for marker-name lists in
  `ProjectClassifier` — keep it scoped, don't disable globally.

## Testing Conventions

- **JUnit 4** + **`BasePlatformTestCase`** when an `Application` is needed.
- Pure unit tests use plain `@Test`.
- Test fixtures are constructed inside the test — never load external files.
- Use fixed string IDs (`"test-id-1"`) instead of `UUID.randomUUID()`.
- Use `tempDir` (provided by IntelliJ test framework) for file-system tests.
- New model fields → extend `ProjectCatalogStateSerializationTest`.
- New CRUD method → extend `CatalogManagerTest`.
- New marker / classifier rule → extend `AutoDiscoverServiceTest` (look for
  `classify*` cases).

---
> Source: [artem-zuev/project-organizer](https://github.com/artem-zuev/project-organizer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-26 -->
