## recallly

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Recallly is a B2B, offline-first, AI-powered native Android app for field professionals (Sales Reps, Field Engineers, Insurance Adjusters). It eliminates manual CRM data entry using on-device AI and system integrations.

- **Package**: `com.at.recallly`
- **Min SDK**: 28 (Android 9) / **Target SDK**: 36 / **Compile SDK**: 36
- **JVM Target**: Java 11
- **Compose BOM**: 2026.02.01
- **AGP**: 8.13.2 / **Gradle**: 8.13 / **Kotlin**: 2.3.10

## Build Commands

```bash
./gradlew build              # Full build
./gradlew assembleDebug      # Build debug APK
./gradlew assembleRelease    # Build release APK
./gradlew test               # Run unit tests
./gradlew connectedAndroidTest  # Run instrumented tests (requires device/emulator)
./gradlew :app:testDebugUnitTest --tests "com.at.recallly.ExampleUnitTest"  # Single test class
./gradlew clean              # Clean build artifacts
```

On Windows, use `gradlew.bat` instead of `./gradlew`.

## Secrets & Local Configuration

`local.properties` (gitignored) must contain these keys, which get injected as `BuildConfig` fields:

- `WEB_CLIENT_ID` — Firebase/Google Sign-In OAuth client ID
- `GEMINI_API_KEY` — Gemini AI API key for field extraction
- `ADMOB_APP_ID` — AdMob application ID (injected via manifest placeholder; defaults to test ID)
- `ADMOB_REWARDED_PRE_RECORD_ID` — Rewarded ad unit for pre-record (defaults to test ID)
- `ADMOB_REWARDED_POST_SAVE_ID` — Rewarded ad unit for post-save (defaults to test ID)
- `BILLING_SUBSCRIPTION_ID` — Play Billing subscription product ID (defaults to `recallly_premium_monthly`)

These are read at build time via `Properties` in `app/build.gradle.kts`.

## Architecture

**Clean Architecture + MVVM** with Unidirectional Data Flow (UDF). Single `:app` module.

### Layer Structure (under `com.at.recallly`)

- **`presentation/`** — UI layer (Compose screens, ViewModels, Navigation)
  - ViewModels expose exactly ONE `StateFlow<UiState>` and accept `UiEvent` sealed interfaces
  - No business logic in ViewModels
  - Feature screens organized by feature (e.g., `auth/` has LoginScreen, SignUpScreen, AuthViewModel, AuthUiState, AuthUiEvent)
- **`domain/`** — Business logic (pure Kotlin models, repository interfaces, UseCases)
  - No Android dependencies allowed here
  - UseCases use `operator fun invoke()` pattern
- **`data/`** — Data layer (Room DAOs, DataStore, repository implementations, API clients)
- **`core/`** — Shared infrastructure
  - `di/` — Koin dependency injection modules
  - `result/` — `Result<T>` sealed class (Success/Error) for error handling
  - `theme/` — Material3 Color, Theme, Typography
  - `util/` — DispatcherProvider, Constants, Extensions, RecalllyException

### Navigation Flow

`Splash → (Login/SignUp) → Language Selection → Persona → Fields → WorkSchedule → DataConsent → Home`

Routes defined as `@Serializable` sealed classes in `presentation/navigation/Route.kt`. The `RecalllyNavGraph` observes auth + onboarding state via `LaunchedEffect` to auto-navigate. Settings has sub-routes with parameter passing (e.g., `SettingsFieldSelection(fromPersonaChange: Boolean)`).

### Key Files

- **Entry point**: `MainActivity.kt` → `RecalllyNavGraph` → screens
- **Application**: `RecalllyApplication.kt` (Koin + Timber initialization, loads saved language via `runBlocking`)
- **DI**: `core/di/AppModule.kt` (single Koin module — all registrations here)
- **Navigation**: `presentation/navigation/RecalllyNavGraph.kt` + `Route.kt`
- **Database**: `data/local/db/RecalllyDatabase.kt` (Room, schema exports to `app/schemas/`)
- **Preferences**: `data/local/datastore/PreferencesManager.kt`
- **Voice note storage**: `data/local/file/VoiceNoteFileStorage.kt` — JSON file at `filesDir/voice_notes.json`, Mutex for thread safety
- **PDF export**: `data/export/PdfExportService.kt` — native Android `PdfDocument` API, A4 pages, uses `FieldLocalizer` for localized field names
- **Language**: `core/util/LanguageManager.kt` — manages locale switching via `AppCompatDelegate.setApplicationLocales`
- **Ads**: `data/ad/RewardedAdManager.kt` — Google Mobile Ads rewarded ads (initialized in `RecalllyApplication`)
- **Billing**: `data/billing/BillingClientWrapper.kt` + `PremiumPreferences.kt` — Google Play Billing for premium subscription
- **Reminders**: `data/notification/AlarmReminderScheduler.kt` — `AlarmManager`-based reminders with `ReminderReceiver` + `BootReceiver` for persistence across reboots

## Speech Recognition (Dual-Mode)

The app uses two speech recognition paths, chosen at mic-tap time via `ConnectivityChecker`:

- **Online**: Android `SpeechRecognizer` with `RecognitionListener` (real-time partial results)
- **Offline**: whisper.cpp via JNI (batch transcription after recording finishes)

### whisper.cpp Native Integration

- **Git submodule** at `app/src/main/cpp/whisper.cpp/` (from `ggml-org/whisper.cpp`)
- **CMake** build at `app/src/main/cpp/CMakeLists.txt`, JNI bridge in `app/src/main/cpp/jni.c`
- **NDK**: r26.3 (`26.3.11579264`) — required for 16KB page alignment (Android 15+ Play Store requirement)
- **ABI targets**: `arm64-v8a` (fp16 variant), `armeabi-v7a` (vfpv4 variant), plus generic fallback
- **Model**: `ggml-base.en.bin` (~142MB), downloaded at runtime from Hugging Face to `filesDir/models/`
- After cloning, initialize the submodule: `git submodule update --init --recursive`

### Key Whisper Files

- `data/whisper/WhisperContext.kt` — JNI wrapper
- `data/whisper/WhisperModelManager.kt` — Model download/state management
- `data/whisper/AudioRecorder.kt` — 16kHz PCM Float32 capture via `AudioRecord`
- `data/repository/WhisperRepositoryImpl.kt` — Wires whisper components together

### AI Extraction Pipeline

- `data/remote/GeminiExtractionService.kt` — Gemini 2.5 Flash with persona-specific prompts
- `data/worker/ExtractionWorker.kt` + `ExtractionWorkScheduler.kt` — WorkManager-based background extraction
- `domain/usecase/voice/ExtractFieldsUseCase.kt` — Orchestrates extraction from transcript

### HomeViewModel Recording States

`HomeViewModel` (14 DI parameters) manages the recording lifecycle with these states:
- `RecordingState.Idle` → `Recording` (online) or `RecordingWhisper` (offline) → `Transcribing` → `Processing`
- `ModelDownloadState`: `NotDownloaded` → `Downloading(progress)` → `Downloaded` | `Error`
- Whisper mode runs 3 coroutine jobs: `whisperTimerJob` (elapsed time), `whisperAmplitudeJob` (visual feedback), `whisperSilenceJob` (3-second silence timeout at amplitude threshold 0.01f)

## Dependency Management

Dependencies managed via version catalog at `gradle/libs.versions.toml`. Add new dependencies there, not in `build.gradle.kts`.

### Key Dependencies

| Category | Library |
|----------|---------|
| DI | Koin |
| DB | Room (KSP for annotation processing) |
| Prefs | DataStore Preferences |
| Nav | Navigation Compose |
| Auth | Firebase Auth + Credential Manager |
| Calendar | Google Calendar API |
| Billing | Google Play Billing |
| Background | WorkManager |
| Logging | Timber |
| AI | Google Generative AI (Gemini) |
| Ads | Google Mobile Ads (AdMob rewarded ads) |
| Serialization | Kotlinx Serialization JSON |

## Multi-Language Support

4 supported languages: English (`en`), Arabic (`ar`), Spanish (`es`), Turkish (`tr`).

- String resources in `values/`, `values-ar/`, `values-es/`, `values-tr/` (~282 strings each)
- String naming convention: `common_`, `login_`, `signup_`, `error_`, `persona_`, `field_sr_`, `field_fe_`, `field_ia_`
- `LanguageManager` handles locale switching and provides whisper language codes + SpeechRecognizer locales
- `LocalizationExtensions.kt` — Composable extensions using `stringResource(R.string.*)` for UI
- `FieldLocalizer` — non-Composable object for context-based string lookup (used in PDF export)
- RTL support enabled in manifest; `localeConfig` declared in `@xml/locales_config`

## Architectural Rules

- 100% Kotlin. No Java.
- Kotlin Coroutines & Flow/StateFlow for concurrency.
- Domain layer must have zero Android dependencies.
- Repositories implement domain interfaces in the data layer.
- Strictly offline-first — no cloud database.
- KSP + Room compiler are commented out — KSP 2.3+ requires AGP 9.0+ which Android Studio doesn't yet support. Uncomment when upgrading to AGP 9.
- Voice notes are stored as JSON files via `VoiceNoteFileStorage` (not Room) — loaded into memory on app start via `VoiceNoteRepositoryImpl.loadFromDisk()`. The repository maintains an in-memory `MutableStateFlow` cache and persists on every add/update/delete.
- Custom fields follow the same file-based pattern via `CustomFieldFileStorage` + `CustomFieldRepositoryImpl` (loaded from disk at Koin init via `runBlocking`).
- No custom lint, detekt, or ktlint config — relies on IDE defaults.
- No CI/CD pipeline configured.

---
> Source: [7pak/Recallly](https://github.com/7pak/Recallly) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
