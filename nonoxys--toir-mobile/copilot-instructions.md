## toir-mobile

> ./gradlew :android:app:assembleDebug     # Android debug APK

# KMMTemplate

## Build Commands

```shell
./gradlew :android:app:assembleDebug     # Android debug APK
./gradlew :android:app:assembleRelease   # Android release APK
./gradlew detekt                          # Static analysis
```

iOS: open `iosApp/` in Xcode.

## Architecture

KMM project: Android + iOS, shared Kotlin code.
MVI stack: MVIKotlin + Koin + Ktor + Compose Multiplatform + moko-resources.
Each feature: 4 modules `api` → `impl` → `presentation` → `ui`.

## Rules (`.claude/rules/`)

| File                        | Description                                                                      |
|-----------------------------|----------------------------------------------------------------------------------|
| `mobile-overview.mdc`       | Tech stack versions, project structure, feature architecture, naming conventions |
| `mobile-mvi.mdc`            | Store / BaseExecutor / Reducer / StoreFactory / BaseViewModel patterns           |
| `mobile-di-navigation.mdc`  | Koin module setup, navigation routes, Screen API                                 |
| `mobile-compose.mdc`        | Recomposition optimization, composable splitting, Preview rules                  |
| `mobile-data-layer.mdc`     | Remote/Domain/UI entities, Mapper, DTO rules, Repository contract                |
| `mobile-error-handling.mdc` | coRunCatching, wrapResultSuccess/Failure, Napier logging                         |
| `mobile-network.mdc`        | NetworkEnvironment, KtorClient, ApiClient, platform DI                           |
| `mobile-resources.mdc`      | moko-resources setup, MR usage, string key naming                                |
| `mobile-code-rules.mdc`     | Access modifiers, class member ordering                                          |

## API контракт

Бэкенд: `~/IdeaProjects/toir-backend/src/main/resources/openapi/documentation.yaml`

Все эндпоинты, поля DTO, коды ошибок и enum-значения — только оттуда. Не угадывай поля по названию.
При расхождении между кодом и yaml — yaml главнее, сообщи об этом.

## Key Gotchas

- Use `coRunCatching` (not `runCatching`) in suspend functions — preserves CancellationException.
- Executors extend `BaseExecutor`, override `suspendExecuteIntent` / `suspendExecuteAction`.
- ViewModels must call `store.dispose()` in `onCleared()`.
- Strings via `MR.strings.*` — no hardcoded strings in Composables.
- DTO fields: always `@Serializable` + `@SerialName` on every field, no default values (use nullable).
- All deps in `gradle/libs.versions.toml` — never hardcode versions.
- New features via IntelliJ plugin **KMP Module Generator** (templates in `module-templates/`).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/NonoxyS) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
