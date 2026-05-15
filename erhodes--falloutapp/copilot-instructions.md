## falloutapp

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

FalloutApp is a Kotlin Multiplatform / Compose Multiplatform character manager for the Fallout TTRPG. Targets: Android, Desktop (JVM), Web (JS + WASM). iOS source set exists but the targets are commented out in `composeApp/build.gradle.kts`.

The Desktop target also runs an embedded Ktor Netty server alongside the Compose UI (see `server/Server.kt`, started from `desktopMain/main.kt`).

## Build & Run (Windows `gradlew.bat`, POSIX `./gradlew`)

- Android debug APK: `gradlew :composeApp:assembleDebug`
- Desktop run: `gradlew :composeApp:run` (main class `com.erhodes.falloutapp.MainKt`)
- Desktop JAR: `gradlew :composeApp:desktopJar`
- Web (WASM, preferred): `gradlew :composeApp:wasmJsBrowserDevelopmentRun`
- Web (JS, legacy browsers): `gradlew :composeApp:jsBrowserDevelopmentRun`
- iOS: open `iosApp/` in Xcode (KMP iOS targets currently disabled in the Gradle build)

## Tests

Tests live in `composeApp/src/commonTest/kotlin/...` using `kotlin.test`.

- Run all tests (desktop target): `gradlew :composeApp:desktopTest`
- Run all tests (all targets): `gradlew :composeApp:allTests`
- Single test class (desktop): `gradlew :composeApp:desktopTest --tests "com.erhodes.falloutapp.CharacterTests"`
- Single test method: append `.methodName` to the `--tests` filter.

Android unit tests use `unitTests.isReturnDefaultValues = true`, so Android-framework stubs return defaults rather than throwing.

## Architecture

Layered MVVM with DI, all in `composeApp/src/commonMain/kotlin/com/erhodes/falloutapp/`:

- `model/` — Domain objects: `Character`, `Item`/`ItemTemplate`, `Perk`, `Recipe`, `Trait`, `Encounter`, `Stats`, `Skills`, plus sub-packages `ability/`, `action/`, `condition/`, `effect/`, `requirement/`. Persisted types are `@Serializable` (kotlinx.serialization).
- `data/` — DataSources and custom serializers. `CharacterDataSource` persists via KStore; `ItemDataSource`, `PerkDataSource`, `RecipeDataSource`, `TraitDataSource`, `EnemyDataSource`, `EnemyItemDataSource` are in-memory template catalogs.
- `repository/` — `CharacterRepository`, `ItemRepository`, `PerkRepository`, `LoginRepository`. Repositories are the mutation entry point; any mutation on `CharacterRepository` is expected to persist via `saveCharacters()`.
- `presentation/` — ViewModels + `*UiState` classes using `StateFlow` / `MutableStateFlow`. `mutableStateListOf` is used for the observable character collection.
- `ui/` — Compose screens and components (Material 3 + Navigation Compose). See `systemPatterns.md` for the navigation graph.
- `di/Di.kt` — Single Koin `appModule`; `initKoin()` is idempotent (swallows "already started"). ViewModels inject via `KoinComponent` + `by inject()`.
- `network/` — Ktor client (`HttpClientManager`, `UserApi`).
- `util/` — Includes `AppLogger` (expect/actual per platform).

### Platform source sets

- `androidMain/` — `MainActivity`, `Storage.android.kt`, `Platform.android.kt`, `AppLogger.android.kt`.
- `desktopMain/` — Compose `main.kt` plus **embedded Ktor server** (`server/Server.kt`, `server/ServerManager.kt`), `UserRepository`, `EncounterViewModel`, `UserViewModel`, admin screens (`AddEnemyScreen`, `UserListScreen`). Desktop has its own Koin module in `di/DesktopDi.kt`. The server currently binds `0.0.0.0:8080` with a `/health` endpoint and is stopped via shutdown hook (`engine.stop(1000, 2000)`).
- `webMain/` — shared web code (KStore storage backend). `jsMain/` and `wasmJsMain/` hold target-specific bits; WASM uses `libs.ktor.client.wasm`.
- `iosMain/` — iOS expect/actual implementations (targets disabled in Gradle).

### Key patterns to preserve

- **Template vs. instance for items**: `ItemTemplate`, `WeaponTemplate`, `ArmorTemplate`, `StackableItemTemplate` are blueprints in `ItemDataSource`. `BasicItem`, `Weapon`, `Armor`, `StackableItem` are the instances owned by a `Character`. `ItemTemplateSerializer` serializes templates by `id` only and rehydrates via `ItemDataSource.getItemTemplateById()` — templates must be registered in `ItemDataSource` before deserialization. `PerkSerializer`, `TraitSerializer`, `RecipeSerializer` follow the same id-lookup pattern against their respective DataSources.
- **Sealed/interface hierarchies** for polymorphism: `Requirement` (`StatRequirement`, `SkillRequirement`), `ItemTemplate` subtypes, `Ability` variants in `model/ability/`.
- **SPECIAL + skills**: 7 SPECIAL stats; 12 skills stored as `ArrayList<Int>` indexed by `Skills` enum ordinal. Max skill = `5 + level/2`. Character weight limits: `loadoutLimit = strength + 4`, `inventoryLimit = 10`.
- **Damage flow**: `Character.takeDamage` routes through equipped armor (absorbs up to toughness, then armor durability, then character) — don't bypass this.
- **Persistence**: Mutations must go through `CharacterRepository` so `saveCharacters()` runs; writing to `Character` fields directly from UI skips serialization.

## Dependencies & versions

Managed via `gradle/libs.versions.toml` (referenced as `libs.*`). Kotlin 2.2.20, Compose Multiplatform 1.9.0, Koin 4.1.1, Ktor 3.0.0, kotlinx-serialization-json 1.9.0, KStore 1.0.0. JDK 11 (`JvmTarget.JVM_11`) for both Android and desktop.

## Supplemental docs

Longer-form context lives in `memory-bank/` (`projectbrief.md`, `productContext.md`, `systemPatterns.md`, `techContext.md`, `activeContext.md`, `progress.md`). Check `activeContext.md` for the current work focus before making non-trivial changes.

---
> Source: [erhodes/FalloutApp](https://github.com/erhodes/FalloutApp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
