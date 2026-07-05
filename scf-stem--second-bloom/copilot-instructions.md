## second-bloom

> - This repository is currently a single-module Android app built with Gradle and Jetpack Compose.

# AGENTS.md

## Project Snapshot

- This repository is currently a single-module Android app built with Gradle and Jetpack Compose.
- The only application module is `:app`.
- The codebase is still early-stage, but it is no longer just a UI skeleton.
- The app now includes a lightweight domain layer, a presentation layer centered on one `ViewModel`, a small repository layer, a mock remote API, a real remote API scaffold, and file-based local persistence for MVP history.
- Treat this file as an AI-collaboration guide, not a replacement for a full onboarding README.

## System Boundaries

- Keep the current architecture lightweight unless a task explicitly requires expansion.
- The app currently contains:
  - Compose UI screens, navigation, and theme code
  - Domain models in `app/src/main/java/com/scf/secondbloom/domain/model`
  - A presentation state holder in `app/src/main/java/com/scf/secondbloom/presentation/remodel/RemodelViewModel.kt`
  - A small repository layer in `app/src/main/java/com/scf/secondbloom/data/repository`
  - A mock remote API and a real HTTP API scaffold in `app/src/main/java/com/scf/secondbloom/data/remote`
  - File-based local persistence for saved analysis and plan history in `app/src/main/java/com/scf/secondbloom/data/local`
- There is still no dependency injection container, no background service, no multi-module split, and no production backend implementation checked into this repository.
- Do not introduce new modules, heavy frameworks, Room, Retrofit, or complex architectural abstractions without a clear requirement.
- Prefer incremental changes that fit the existing single-module Compose structure.

## Project Structure

- `app/src/main/java/com/scf/secondbloom`
  Application entry package.
- `app/src/main/java/com/scf/secondbloom/data`
  Lightweight data layer for local persistence, remote APIs, and repositories.
- `app/src/main/java/com/scf/secondbloom/domain/model`
  Domain models used across UI, repository, and persistence code.
- `app/src/main/java/com/scf/secondbloom/presentation/remodel`
  Main workflow state and business actions exposed to the UI.
- `app/src/main/java/com/scf/secondbloom/ui`
  Compose UI, screens, shared UI components, and theme code.
- `app/src/main/java/com/scf/secondbloom/navigation`
  Navigation routes and navigation-related definitions.
- `docs/api`
  In-repository backend contract documentation. The current MVP contract lives here.
- `app/src/test`
  Local unit tests that run on the host machine.
- `app/src/androidTest`
  Instrumentation tests that run on a connected Android device or emulator.

## Run & Test Commands

- `sh gradlew testDebugUnitTest`
  Run local unit tests for the debug build.
- `sh gradlew assembleDebugAndroidTest`
  Build the instrumentation test APK and verify androidTest sources compile.
- `sh gradlew connectedDebugAndroidTest`
  Run instrumentation tests on a connected device or running emulator.
- `sh gradlew installDebug`
  Install the debug build on a connected device or running emulator.

Known command caveat:

- `gradlew` currently does not have execute permission in this workspace, so `./gradlew ...` fails with a permission error.
- Use `sh gradlew ...` by default.
- If local workflow requires direct execution, add permission explicitly with `chmod +x gradlew`.
- If Gradle reports Kotlin daemon cache or incremental compilation issues during repeated runs, `sh gradlew --stop` is the safe first recovery step before rerunning commands sequentially.

## API Docs

- Current status: a lightweight backend contract document is present at `docs/api/mvp-contract.md`.
- A system-level backend design document now also lives at `docs/api/backend-architecture.md`.
- A delivery-focused backend plan and SQL draft now also live at `docs/api/backend-delivery-plan.md` and `docs/api/postgres-schema-draft.sql`.
- A first runnable backend implementation now exists outside this repository at `/Users/peng/AndroidStudioProjects/second-bloom-backend`.
- The app defaults to `MockRemodelApi`, but can switch to `RealRemodelApi` through `BuildConfig` flags defined in `app/build.gradle.kts`.
- Do not invent or imply external API contracts that do not exist in code or docs.
- `docs/api/` is the canonical location for API documentation in this repository.
- Keep this file limited to linking or summarizing API docs, not duplicating detailed API specs.

## Working Rules / Known Gotchas

- Treat `androidTest` as the current integration-test layer. There is no separate end-to-end test framework yet.
- Instrumentation tests require a connected device or started emulator before running `connectedDebugAndroidTest`.
- `assembleDebugAndroidTest` is useful as a fast verification step, but it does not execute tests on a device.
- The current tests are no longer just template examples. They cover important screen states and `RemodelViewModel` behavior, but they are still not full end-to-end coverage.
- When adding meaningful product behavior, prefer adding or updating real screen, navigation, or interaction tests rather than expanding template-only tests.
- Local history is currently stored as a JSON snapshot in app-private storage, not in a database. Preserve that lightweight approach unless there is a clear reason to migrate.
- Real backend support is scaffolded, not complete. If you change request or response shapes, keep `docs/api/mvp-contract.md`, `docs/api/backend-architecture.md`, `docs/api/backend-delivery-plan.md`, `docs/api/postgres-schema-draft.sql`, remote DTOs, and `RealRemodelApi` aligned in the same change.
- After every meaningful planning or development task, update `PLANS.md` so the ExecPlan progress, discoveries, decisions, and current next steps remain aligned with the actual repository state.
- When planning or implementing work, decompose the task as much as reasonably possible and prefer delegating clearly bounded, non-overlapping subtasks to subagents in parallel when that can accelerate delivery without creating merge conflicts.
- Keep documentation aligned with the actual repository state. If architecture or tooling changes, update this file with the new source of truth.

---
> Source: [scf-stem/second-bloom](https://github.com/scf-stem/second-bloom) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-05 -->
