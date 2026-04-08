## stally

> This document defines the repository-specific agent behavior contract for

# AGENTS.md

This document defines the repository-specific agent behavior contract for
Stally.

## Agent Philosophy

- Follow existing repository conventions as the source of truth.
- Prefer minimal, safe changes over speculative refactors.
- Treat sibling repositories and external packages as read-only reference
  material unless the user explicitly asks otherwise.

## Naming and Language Rules

Use English for:

- Branch names
- Code comments
- Documentation
- Identifiers

Avoid non-English text unless required for UI localization or legal content.

## Markdown Guidelines

All Markdown files must follow:

https://github.com/DavidAnson/markdownlint/blob/main/doc/Rules.md

## Swift Code Guidelines

### Follow SwiftLint rules

All Swift code must comply with the project's SwiftLint configuration.

### Avoid abbreviated variable names

#### Preferred

- `result`
- `image`
- `button`

#### Not preferred

- `res`
- `img`
- `btn`

### Use `.init(...)` when return type is explicit

#### Preferred

```swift
var user: User {
    .init(name: "Alice")
}
```

#### Not preferred

```swift
var user: User {
    User(name: "Alice")
}
```

### Multiline control-flow formatting

Do NOT use single-line bodies for control-flow statements or trailing closures.

#### Preferred

```swift
guard let currentUser else {
    return
}

if isDebugMode {
    logger.debug("Entering debug state")
}

tasks.filter { task in
    task.isCompleted
}
```

#### Not preferred

```swift
guard let currentUser else { return }
if isDebugMode { logger.debug("Entering debug state") }
tasks.filter { $0.isCompleted }
```

## Repository Boundaries

- `Stally/Sources/` owns app composition, SwiftUI screens, platform
  integrations, preference-backed UI state, and runtime/bootstrap setup.
- `Stally/Resources/` owns app-localized resources and assets used by the app
  target.
- `StallyLibrary/` owns reusable domain and persistence logic, SwiftData
  models, services, calculators, backup import/export logic, and shared deep
  linking definitions.
- Prefer moving reusable business or persistence logic into `StallyLibrary/`
  rather than growing `Stally/Sources/`.
- Keep the app target thin: it should assemble dependencies and present UI, not
  absorb shared domain logic.
- `Designs/Architecture/`, `Designs/Decisions/`, and `Designs/Overviews/`
  define the current outer-architecture intent. Update them when you change
  repository structure or responsibility boundaries.

## Build and Test Entry Point

Agents MUST use one of these standardized entrypoints:

```sh
bash ci_scripts/tasks/verify_task_completion.sh
bash ci_scripts/tasks/verify_repository_state.sh
```

Agents may run `bash ci_scripts/tasks/check_environment.sh --profile verify`
first to diagnose missing local prerequisites.
When Swift files are edited, agents should run
`bash ci_scripts/tasks/format_swift.sh` before the final verification gate.
`bash ci_scripts/tasks/verify_task_completion.sh` is the non-destructive
verification gate.
`bash ci_scripts/tasks/verify_pre_commit.sh` reruns the same non-destructive
verification shell for manual final checks and `.pre-commit-config.yaml`.
SwiftLint is resolved from the `SimplyDanny/SwiftLintPlugins` package declared
in `Stally.xcodeproj`, not from a separately installed `swiftlint` binary.

Use these narrower entrypoints only when the task specifically needs them:

```sh
bash ci_scripts/tasks/build_app.sh
bash ci_scripts/tasks/test_shared_library.sh
bash ci_scripts/tasks/verify_pre_commit.sh
```

## CI Artifact Layout

CI run artifacts are written under `.build/ci/runs/<RUN_ID>/`.
Each run stores `summary.md`, `commands.txt`, `meta.json`, `logs/`,
`results/`, and `work/`.
Shared CI directories are under `.build/ci/shared/` (`cache/`,
`DerivedData/`, `tmp/`, `home/`).
Only the newest 5 run directories are retained.
The entire `.build/ci` directory is disposable.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/muhiro12)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/muhiro12)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
