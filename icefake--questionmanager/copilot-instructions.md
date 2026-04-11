## questionmanager

> **Generated:** 2026-03-11

# PROJECT KNOWLEDGE BASE

**Generated:** 2026-03-11
**Commit:** a953731
**Branch:** master

## OVERVIEW
Android question manager app using Kotlin, Jetpack Compose, MVVM architecture, Room database, Retrofit for DeepSeek API, Jsoup for web parsing.

## STRUCTURE
```
./
├── app/                          # Android application module
├── build.gradle.kts              # Root build script
├── gradle/                       # Gradle wrapper
├── gradle.properties             # Gradle properties
├── settings.gradle.kts           # Project settings
├── ARCHITECTURE.md               # Detailed architecture document
└── AGENTS.md                     # This file
```

## WHERE TO LOOK
| Task | Location | Notes |
|------|----------|-------|
| Adding a new screen | `app/src/main/java/com/example/questionmanager/ui/screen/` | Follow existing pattern: `*Screen.kt` + `*ViewModel.kt` |
| Database changes | `app/src/main/java/com/example/questionmanager/data/local/db/` | Entities in `entity/`, DAOs in `dao/`, migrations in `AppDatabase.kt` |
| API integration | `app/src/main/java/com/example/questionmanager/data/remote/` | Retrofit services in `api/`, models in `model/` |
| Business logic | `app/src/main/java/com/example/questionmanager/domain/usecase/` | Use cases encapsulate complex operations |
| UI components | `app/src/main/java/com/example/questionmanager/ui/component/` | Reusable Compose components |
| Dependency injection | `app/src/main/java/com/example/questionmanager/di/` | Hilt modules provide dependencies |
| Utilities | `app/src/main/java/com/example/questionmanager/util/` | Extension functions, constants, shared helpers |
| Settings/Preferences | `app/src/main/java/com/example/questionmanager/data/local/datastore/` | DataStore for preferences, encrypted storage for secrets |
| Testing | `app/src/test/` & `app/src/androidTest/` | Unit tests and instrumented tests |

## CODE MAP
Key architectural components (see ARCHITECTURE.md for full details):

| Symbol | Type | Location | Role |
|--------|------|----------|------|
| `QuestionManagerApp` | class | `app/src/main/java/com/example/questionmanager/QuestionManagerApp.kt` | Hilt application entry point |
| `MainActivity` | class | `app/src/main/java/com/example/questionmanager/MainActivity.kt` | Single Activity hosting Compose UI |
| `AppDatabase` | abstract class | `app/src/main/java/com/example/questionmanager/data/local/db/AppDatabase.kt` | Room database with 3 entities |
| `QuestionRepository` | class | `app/src/main/java/com/example/questionmanager/data/repository/QuestionRepository.kt` | Central data coordinator |
| `AiRepository` | class | `app/src/main/java/com/example/questionmanager/data/repository/AiRepository.kt` | AI API calls with rate limiting semaphore |
| `HomeViewModel` | class | `app/src/main/java/com/example/questionmanager/ui/screen/home/HomeViewModel.kt` | Example ViewModel using StateFlow |

## CONVENTIONS
- **Naming**: Entities end with `Entity`, DAOs with `Dao`, Repositories with `Repository`, ViewModels with `ViewModel`, UseCases with `UseCase`.
- **Architecture**: MVVM with Repository pattern; UI observes StateFlow from ViewModel.
- **Dependency Injection**: Hilt used throughout; modules in `di/` package.
- **Database**: Room with schema export enabled (`app/schemas/` must be committed). AutoMigration preferred.
- **Networking**: Retrofit with Kotlinx Serialization; API key stored in EncryptedSharedPreferences.
- **Concurrency**: Kotlin Coroutines + Flow; IO dispatcher for database/network operations.
- **Error handling**: Result wrapper for suspend functions; UI shows error states.
- **Testing**: Unit tests use MockK, Turbine for Flow testing; Hilt test modules available.

## ANTI-PATTERNS (THIS PROJECT)
1. **DO NOT** put business logic in ViewModel – delegate to UseCase.
2. **DO NOT** access Room DAO directly from UI layer – use Repository.
3. **DO NOT** hardcode API keys – use SecureDataStore.
4. **DO NOT** perform network/database operations on Main thread – use coroutines.
5. **DO NOT** ignore schema versioning – always increment version and provide migration.
6. **DO NOT** commit generated files (`app/build/`, `.gradle/`, `.idea/`).

## UNIQUE STYLES
- **Rate limiting**: AI API calls protected with `Semaphore(permits = 3)` to avoid 429 errors.
- **AI response parsing**: Multi‑strategy fallback parser (`AiResponseParser`) for drill‑down JSON extraction.
- **Web parsing**: Jsoup for static HTML, with detection for SPA pages (fallback warning).
- **Streaming answers**: Support for incremental answer generation with `Flow<String>`.
- **Encrypted storage**: Sensitive data (API key) encrypted via `security‑crypto` `EncryptedSharedPreferences`.
- **Default prompts**: Seeded database with four default system prompts on first launch.
- **FTS + LIKE fallback search**: Room FTS4 with Chinese tokenization + LIKE fallback on error.
- **Entity‑domain mapping**: Pure Kotlin domain models with `toDomain()`/`toEntity()` extension mapping.
- **Dynamic base URL**: OkHttp interceptor rewrites API base URL at runtime for custom endpoints.
- **Retry with backoff**: Exponential backoff retry logic with HTTP error detection.

## COMMANDS
```bash
# Build
./gradlew build

# Run tests
./gradlew test
./gradlew connectedAndroidTest

# Install debug version
./gradlew installDebug

# Generate Room code
./gradlew kspKotlin

# Clean build
./gradlew clean

# Lint check
./gradlew lint
```

## CI/CD (GitHub Actions)
- **Workflow file**: `.github/workflows/android.yml`
- **Issues detected**:
  1. **Overly broad permissions**: `contents: write` at workflow level (should be scoped per-job)
  2. **Duplicate build in release job**: Rebuilds APK instead of downloading artifact from `build` job
  3. **Inverted artifact upload**: `if: failure()` uploads artifacts only when build fails (should be on success)
  4. **Missing job dependency**: No `needs: build` + artifact download in release job
  5. **Implicit Android SDK**: No explicit SDK setup (relies on `ubuntu-latest` having it)
- **Recommendations**:
  - Scope permissions per-job: `build` = `contents: read`, `release` = `contents: write`
  - Pass artifact from `build` to `release` job
  - Change artifact upload to `if: always()` or remove condition
  - Add explicit Android SDK setup with caching

## NOTES
- **Room schemas**: Export is enabled; `app/schemas/` directory must be kept under version control.
- **API key**: Must be set in Settings screen before using AI features.
- **DeepSeek API**: Uses `deepseek-chat` model; temperature and max tokens configurable.
- **Web parsing**: Limited to server‑rendered HTML; dynamic SPAs may not work.
- **Drill‑down**: Generates 5 related questions; user selects which to create as linked entries.
- **Hilt**: Application annotated with `@HiltAndroidApp`, Activity with `@AndroidEntryPoint`.
- **Compose**: Uses Material 3 theme; navigation via `Navigation Compose`.
- **DataStore**: Two implementations – `SettingsDataStore` (plain) and `SecureDataStore` (encrypted).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/IceFake)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/IceFake)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
