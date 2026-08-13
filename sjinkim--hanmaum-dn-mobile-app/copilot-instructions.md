## hanmaum-dn-mobile-app

> Repository-level instructions for Codex. Treat this file as the durable source of truth for how to work in this repo. Keep it concise; detailed workflows live under `docs/codex/`, `.codex/commands/`, and `.agents/skills/`.

# AGENTS.md — hanmaum-dn-mobile-app

Repository-level instructions for Codex. Treat this file as the durable source of truth for how to work in this repo. Keep it concise; detailed workflows live under `docs/codex/`, `.codex/commands/`, and `.agents/skills/`.

## Project Snapshot

- Kotlin Multiplatform mobile app for Android and iOS.
- Shared UI: Compose Multiplatform 1.10.0 + Material3.
- Architecture: feature-first clean architecture with shared KMP domain/data/presentation layers.
- Auth: Keycloak; tokens stored through `TokenStorage` using multiplatform-settings.
- Networking: Ktor 3.3.3 through `core/network/NetworkClient.kt` only.
- DI: Koin 4.0.0 via `di/AppModule.kt`, `DnChurchApp`, and `KoinInit.kt`.
- Navigation: type-safe `@Serializable` route objects in `core/navigation/Routes.kt`.

## Start-of-Session Protocol

1. Read `tasks/lessons.md` before changing anything.
2. Check `git status --short --branch`; if there are user changes, list them and do not overwrite them.
3. Inspect related code before planning or editing.
4. For any task with 3+ steps, architectural choices, or uncertain scope, use a written plan first.
5. If work goes sideways, stop, re-plan, then continue with the smallest safe correction.

## Senior Mobile Engineering Defaults

- Solve the root cause, not the symptom.
- Prefer small, reviewable changes that preserve existing style.
- Shared business logic belongs in `commonMain`; platform APIs stay in platform source sets or behind `expect`/`actual`/interfaces.
- UI must be state-driven: ViewModels expose immutable UI state via `StateFlow`; composables render state and send events back.
- Repositories are the boundary to data sources; ViewModels/composables must not call Ktor, Firebase, Settings, GPS, or platform APIs directly.
- Dependencies are not added casually. Ask before new production dependencies or risky upgrades.
- Do not introduce database/schema/server contract changes unless explicitly requested.
- Security/privacy: never commit secrets, tokens, API keys, keystores, or real user data. Keep environment URLs in BuildKonfig or environment config.

## Architecture Rules

Feature layout under `composeApp/src/commonMain/kotlin/com/hanmaum/dn/mobile/features/<feature>/`:

- `domain/model/` — plain shared models.
- `domain/repository/` — repository interfaces.
- `data/model/` — DTOs and serialization types.
- `data/repository/` — Ktor/settings/cache-backed implementations.
- `presentation/` — ViewModels, UI state, Compose screens/components.

Shared infrastructure lives in `core/`:

- `core/network/NetworkClient.kt` owns Ktor client configuration, base URL handling, JSON, logging, and bearer auth.
- `core/domain/repository/TokenStorage.kt` and `core/data/repository/TokenStorageImpl.kt` own token persistence.
- `core/navigation/Routes.kt` owns type-safe navigation route objects.
- `core/domain/model/NavRoute.kt` is the platform-agnostic navigation intent enum for ViewModels.
- `core/presentation/theme/` owns design-system tokens; do not hardcode one-off colors/shapes/typography in screens.

## Networking Rules

- All backend calls go through the injected shared `HttpClient` from `createHttpClient(tokenStorage)`.
- Do not create ad-hoc Ktor clients in features unless a feature explicitly needs an isolated third-party client and the auth-leak risk is reviewed.
- Bearer tokens must only be sent to the Hanmaum backend host. Never leak Keycloak tokens to pCloud, Google Calendar, or other external hosts.
- Login/Keycloak calls use absolute URLs and bypass backend base-path injection.
- Feature repositories return `Result<T>` or existing project error patterns; avoid throwing across presentation boundaries.

## UI, Design, and Accessibility

All screens must conform to `designs/dn_app/DESIGN.md`.

- New screen from scratch: run `lazyweb:lazyweb-quick-references` first when the lazyweb tool is available; otherwise state it is unavailable and continue with `DESIGN.md`.
- Significant redesign: run `lazyweb:lazyweb-design-improve` with a current screenshot when available; otherwise state the fallback.
- Minor modifications: follow `DESIGN.md` tokens directly.
- Animations: use spring-based specs with `animateFloatAsState` / `AnimatedVisibility`; never linear or ease-in-out transitions.
- Compose performance: avoid expensive work in composables; compute in ViewModels or `remember`; use stable lazy-list keys; use `derivedStateOf` for rapidly changing derived UI state.
- Accessibility: support readable labels/content descriptions, sufficient contrast, scalable text, and tap targets appropriate for mobile.

## Build, Test, and Verification Commands

```bash
# Android debug APK
./gradlew :composeApp:assembleDebug

# Full KMP test suite
./gradlew :composeApp:allTests

# Android compile sanity check
./gradlew :composeApp:compileDebugKotlinAndroid --no-daemon

# iOS simulator tests when shared/platform code can affect iOS
./gradlew :composeApp:iosSimulatorArm64Test

# Single Android unit test class
./gradlew :composeApp:testDebugUnitTest --tests "com.hanmaum.dn.mobile.YourTestClass"

# Clean build
./gradlew clean
```

For iOS app runtime verification, open `/Users/seungjinkim/Documents/Private_Projects/hanmaum-dn-mobile-app/iosApp/iosApp.xcodeproj` in Xcode and run on a simulator/device.

## Definition of Done

Before calling work complete:

1. Add/update tests for behavior changes.
2. Run the smallest relevant tests first, then `./gradlew :composeApp:allTests` when feasible.
3. For shared/common code that may affect iOS, run `./gradlew :composeApp:iosSimulatorArm64Test` when feasible.
4. For Android deliverables, run `./gradlew :composeApp:assembleDebug` or explain why not.
5. Review the diff for regressions, secrets, auth leakage, hardcoded URLs, TODO/FIXME, dead code, and architecture drift.
6. Summarize changed files, verification results, and residual risk.

If a check cannot be run, say exactly why and what risk remains.

## Codex Workflow Assets

Use these reusable assets when relevant:

- Playbook: `docs/codex/mobile-engineering-playbook.md`
- Review checklist: `docs/codex/code-review.md`
- Prompt/command templates: `.codex/commands/*.md`
- Team skills: `.agents/skills/*/SKILL.md`

Recommended command templates:

- Onboarding: `.codex/commands/onboard.md`
- Feature work: `.codex/commands/feature.md`
- Bug fixing: `.codex/commands/bugfix.md`
- Pre-PR review: `.codex/commands/pr-review.md`
- Done check: `.codex/commands/done.md`
- Commit prep: `.codex/commands/commit.md`

## OpenAI/Codex Documentation

Always use the OpenAI developer documentation MCP server if work involves OpenAI API, ChatGPT Apps SDK, Codex, Codex configuration, skills, plugins, or MCP. Prefer official OpenAI docs over memory or third-party summaries.

## Git Rules

- Never add `Co-Authored-By:` trailers to commits.
- Do not rebase, reset, discard, or overwrite user work without explicit permission.
- Commit messages: `<type>(<scope>): <imperative summary max 72 chars>`.
- Types: `feat`, `fix`, `refactor`, `test`, `chore`, `docs`, `perf`, `revert`.

---
> Source: [SJinKim/hanmaum-dn-mobile-app](https://github.com/SJinKim/hanmaum-dn-mobile-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
