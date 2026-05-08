## rune

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Run Commands

```bash
# Run plugin in a sandboxed IDE instance
./gradlew runIde

# Run tests
./gradlew check

# Run a single test class
./gradlew test --tests "com.github.cnrture.rune.SomeTest"

# Build distribution ZIP
./gradlew buildPlugin

# Publish to JetBrains Marketplace
./gradlew publishPlugin

# Run UI tests (starts a robot server on port 8082)
./gradlew runIdeForUiTests

# Verify plugin compatibility
./gradlew runPluginVerifier
```

**Target platform:** IntelliJ Community (IC) version `2024.3.1`, compatible with `241–253.*`. JVM toolchain: Java 21. Kotlin 2.3.20, Compose Desktop 1.10.1. Depends on bundled `org.jetbrains.plugins.terminal`.

**Key dependencies:** kotlinx-serialization-json 1.10.0, FreeMarker 2.3.34. Version catalog in `gradle/libs.versions.toml`.

**Gradle memory:** Daemon uses `-Xmx2g -XX:MaxMetaspaceSize=512m`, Kotlin daemon uses `-Xmx2g`. Configuration cache and build cache are enabled.

**Note:** No test files currently exist in the project. JUnit 4.13.2 is configured as a dependency.

## Architecture

IntelliJ Platform plugin ("Rune") built with Kotlin and Jetpack Compose Desktop for UI. No DI framework — all dependencies are wired manually.

**Single tool window** registered in `plugin.xml`:
- **`ClaudeToolWindowFactory`** → "Claude" panel (right sidebar): Claude CLI terminal integration with session management, skill/agent pickers, command palette

### Layer structure

```
toolwindow/ (Compose UI in ComposePanel)
    ↓
domain/usecase (business logic)
    ↓
data/repository (SkillRepository)
    ↓
service/ (FileScanner, Settings, VCS platform services)
common/ (CliUtils, Constants, VcsProvider)
```

### Key packages

| Package | Responsibility |
|---|---|
| `toolwindow/` | Claude terminal UI: `ClaudeTerminalContent`, `SessionTabBar`, `TerminalInputBar`, `UnifiedCommandPalette`, `ClaudeSessionService` |
| `domain/usecase/` | `ScanSkillsUseCase` |
| `data/` | `SkillRepository` / `SkillRepositoryImpl` – scans markdown files, 5-minute cache |
| `service/` | `FileScanner`, `PluginSettingsService`, `PluginConfigurable`, `JiraService`, `VcsCacheService`, `GitHubPlatformService`, `BitbucketCloudPlatformService`, `BitbucketCredentialService` |
| `common/` | `CliUtils` (CLI resolution + process execution), `Constants`, `VcsProvider` / `VcsProviderDetector` |
| `components/` | Reusable Compose components (all prefixed `TP`: `TPTabRow`, `TPActionCard`, `TPText`, `TPDialogWrapper`, etc.) |
| `theme/` | `TPTheme` / `TPColor` – always use `TPTheme.colors.*` for colors |
| `actions/` | VCS actions, editor notifications, dialogs (commit message generation, PR creation, skill creation) |

### Service access pattern

Services are IntelliJ `@Service` annotated, accessed via:
```kotlin
PluginSettingsService.getInstance(project)   // project-scoped
ClaudeSessionService.getInstance(project)    // project-scoped (in toolwindow package)
VcsCacheService.getInstance()                // app-scoped
```

### Settings persistence

- **`PluginSettingsService`** (project-scoped, `runeplugin.xml`) – skills/agents root paths, commit message prompt, Jira/VCS settings. Configured via IDE Settings > Tools > Rune Settings (`PluginConfigurable`)
- **`VcsCacheService`** (app-scoped, `githubCache.xml`) – cached collaborators, labels, user details per repo

### VCS platform abstraction

`VcsPlatformService` interface with two implementations:
- **`GitHubPlatformService`** – uses GitHub CLI (`gh`) for PR operations
- **`BitbucketCloudPlatformService`** – uses Bitbucket REST API with credentials from `BitbucketCredentialService` (IDE credential store)

`VcsProviderDetector` parses git remote URLs (SSH/HTTPS) to auto-detect GitHub vs Bitbucket.

### Claude terminal integration

`ClaudeSessionService` manages terminal sessions (create, switch, close) using `JBTerminalWidget`. The terminal finds `claude` CLI via `CliUtils.findClaudeCli()` which resolves via login shell (`which`) with fallback to common install locations.

`GenerateCommitMessageAction` runs `claude -p <prompt>` with a 30-second timeout to generate commit messages.

### Skill file format

Skills are discovered by scanning a configurable root directory for `.md` files. Skills tab uses strict filtering (`SKILL.md` suffix); Agents tab accepts any `.md` file. The first non-empty line of the file body becomes `description`.

### Plugin actions (registered in plugin.xml)

| Action | Trigger | Description |
|---|---|---|
| `GenerateCommitMessageAction` | VCS commit dialog | Generates commit message from staged diff via Claude CLI |
| `CreateReviewPRAction` | VCS commit dialog | Creates review branch and PR (GitHub/Bitbucket) |
| `FixPRCommentsAction` | VCS commit dialog | Auto-fix PR review comments via Claude |
| `AskClaudeAction` | Editor/Console right-click menu | Sends selected code to Claude terminal |
| `CheckSkillBestPracticesAction` | Editor/Project right-click menu | Reviews SKILL.md against best practices |
| `SkillBestPracticesNotificationProvider` | Opens SKILL.md files | Editor notification banner with validation actions |

### Compose Desktop notes

- UI uses Jetpack Compose Desktop via `org.jetbrains.compose` plugin, rendered inside `ComposePanel` (Swing interop)
- Skiko render API is set to SOFTWARE for compatibility: `System.setProperty("skiko.renderApi", "SOFTWARE")`
- All platform Skiko natives are bundled (macOS arm64/x64, Linux x64/arm64, Windows x64)
- Always wrap composables in `TPTheme { }` and use `TPTheme.colors.*` for theming

## Conventions

### Error handling

Repositories return `Result<T>` (success/failure) instead of throwing exceptions. External process calls (Claude CLI, git, gh) go through `CliUtils.runProcess()` / `CliUtils.runGit()` with timeout handling. Timeouts and delays are centralized in `Constants`.

### Dialog validation pattern

Dialogs like `CreateSkillDialog` use a state data class with computed validation functions (`nameErrors()`, `nameWarnings()`, `nameHints()`) that return nullable strings. The "Create" button is disabled until all error functions return null. Warnings and hints are displayed but don't block submission.

### Editor notification providers

Implement `EditorNotificationProvider` + `DumbAware` for file-specific banners. See `SkillBestPracticesNotificationProvider` for the pattern: check file name in `collectNotificationData()`, return `EditorNotificationPanel` with action links.

### Caching

`FileScanner` uses a 5-minute in-memory cache for directory scans. Call `invalidateCache()` explicitly when settings change (e.g., root path updates). `VcsCacheService` persists collaborator/label data per repo across IDE restarts.

## CI/CD

GitHub Actions workflows in `.github/workflows/`:
- **`build.yml`** — triggered on push to main and PRs: build, test, Qodana inspection, Plugin Verifier
- **`release.yml`** — triggered on GitHub release: publish plugin to JetBrains Marketplace
- **`run-ui-tests.yml`** — manual trigger: robot server UI tests

---
> Source: [cnrture/Rune](https://github.com/cnrture/Rune) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
