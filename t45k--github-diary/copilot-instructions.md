## github-diary

> GitHub Diary is a Kotlin/Compose Desktop application that manages diary entries stored as Markdown files in GitHub repositories. Users interact through a calendar interface to view, edit, and preview daily entries.

# Project Guidelines: GitHub Diary

## Project Overview
GitHub Diary is a Kotlin/Compose Desktop application that manages diary entries stored as Markdown files in GitHub repositories. Users interact through a calendar interface to view, edit, and preview daily entries.

## Technology Stack
- **Language:** Kotlin 2.3.0, JVM 25
- **UI:** Jetpack Compose Desktop 1.10.0-rc02
- **HTTP:** Ktor Client 3.3.3
- **Testing:** JUnit Jupiter 6.0.0
- **Functional:** Arrow Core (Either for error handling)

## Build Commands
- `./gradlew build`: Compile and build
- `./gradlew run`: Run the application
- `./gradlew test`: Run tests (JUnit 6)
- `./gradlew clean`: Clean build artifacts

## Architecture
The source code is located in `app/src/main/kotlin/`:
- `Main.kt`: Entry point
- `core/entity/`: Domain models (Calendar, DiaryContent, GitHubPersonalAccessToken)
- `core/repository/`: Data access (GitHubClient, DiaryRepository, SettingRepository)
- `core/time/`: DateProvider (JST timezone)
- `ui/AppScreen.kt`: Root composable with routing
- `ui/AppViewModel.kt`: Global state
- `ui/calendar/`: Calendar screen (month view)
- `ui/edit/`: Diary editor
- `ui/preview/`: Markdown preview
- `ui/settings/`: Token/repo configuration
- `ui/navigation/`: NavRoute enum

## Development Patterns
- **ViewModel per screen:** Each screen has a ViewModel extending `androidx.lifecycle.ViewModel` with `mutableStateOf` for Compose state.
- **Repository pattern:** `GitHubClient` handles REST API calls; repositories abstract data access.
- **Functional & Immutable style:** Prefer functional programming and immutable programming models; design logic around pure functions, immutable data, and transformations with minimized side effects.
- **Enum-based routing:** `NavRoute` enum manages navigation via `AppViewModel.currentRoute`.
- **Error Handling:** Use Arrow `Either` in settings validation for functional error handling.

## GitHub API & File Integration
- **Diary path format:** `{year}/{month}/{day}/README.md` (e.g., `2026/01/02/README.md`)
- **Authentication:** Bearer token via Personal Access Token (PAT).
- **Content encoding:** Base64 for file operations.
- **Settings storage:** Local storage at `~/.github_diary/settings.json`.

## Constraints & Configuration
- **Timezone:** Fixed to JST (Asia/Tokyo).
- **Theme:** Dark mode only.
- **Calendar:** Configured for Sunday-start weeks.
- **Diary header:** Uses `# YYYY/MM/DD (Day)` format.
- **Specification:** Full feature specification is available in `spec/spec.md` (Japanese).

## Compose Patterns
- **State hoisting:** Each screen receives `UiState` and callback lambdas from `AppScreen`. ViewModel holds `MutableStateFlow<UiState>` internally and exposes `StateFlow<UiState>`.
- **`remember` / `derivedStateOf`:** Use `remember { mutableStateOf(...) }` for local UI state. Use `remember { derivedStateOf { ... } }` for values computed from other state.
- **Preview:** Place `@Preview` composables in `desktopMain` (e.g., `CalendarScreenPreview.kt`). Use `@PreviewParameter` with `PreviewParameterProvider` for state variants.

---
> Source: [T45K/github-diary](https://github.com/T45K/github-diary) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
