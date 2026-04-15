## localmind

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

LocalMind is a Kotlin Multiplatform (KMP) cross-platform productivity application for Android (API 24+) and iOS (15+). It's an AI-powered task and note manager with a privacy-first, fully offline approach.

**Core Flow:** Voice/text capture → On-device AI parsing → Structured tasks → Local storage

**Status:** Early development phase (foundation being built)

## Build Commands

```bash
# Android debug build
./gradlew :composeApp:assembleDebug

# Android release bundle (for Play Store upload)
./gradlew :composeApp:bundleRelease

# Run all tests
./gradlew :composeApp:test

# Run specific test class
./gradlew :composeApp:testDebugUnitTest --tests "com.markduenas.localmind.ai.RuleBasedParserTest"

# Clean build
./gradlew clean

# Check dependencies
./gradlew :composeApp:dependencies
```

For iOS builds, open `/iosApp/iosApp.xcodeproj` in Xcode.

## Architecture

### Module Structure (Target)

```
composeApp/src/commonMain/kotlin/com/markduenas/localmind/
├── di/                 # Koin dependency injection modules
├── data/
│   ├── local/          # SQLDelight database, DAOs
│   └── repository/     # Repository implementations
├── domain/
│   ├── model/          # Domain entities (Task, Tag, Capture)
│   └── usecase/        # Business logic (ParseCaptureUseCase, etc.)
├── ai/
│   ├── LLMService.kt   # Cactus SDK LLM wrapper
│   ├── STTService.kt   # Cactus SDK speech-to-text wrapper
│   ├── TaskParser.kt   # LLM prompt orchestration
│   └── RuleBasedParser.kt  # Fallback regex parser
├── ui/
│   ├── theme/          # Material 3 theming
│   ├── navigation/     # Compose Navigation
│   ├── capture/        # Quick capture screens
│   ├── tasks/          # Task list views (Today, Upcoming)
│   ├── review/         # Parse review/edit screen
│   └── settings/       # App settings
└── util/               # Shared utilities
```

### Key Patterns

**Expect/Actual:** Platform-specific implementations
- Database drivers: `DatabaseDriverFactory` expect/actual
- Notifications: `NotificationHelper` expect/actual

**Data Flow:**
```
UI (Compose) → ViewModel (StateFlow) → UseCase → Repository → SQLDelight
                                           ↓
                                    AI Service (Cactus SDK)
```

**Dependency Injection:** Koin 4.x with viewModelOf, singleOf, factoryOf

## Tech Stack

| Layer | Technology |
|-------|------------|
| UI | Compose Multiplatform 1.10+ |
| State | Kotlin Coroutines + Flow |
| DI | Koin 4.x |
| Database | SQLDelight 2.x + SQLCipher |
| LLM | Cactus SDK 1.2+ (qwen3-0.6, gemma3-270m) |
| Speech | Cactus STT (whisper-tiny) |
| Testing | kotlin-test + Turbine |

## Key Files

- `localmind-specification.md` - Full product specification with features, architecture, competitive analysis
- `IMPLEMENTATION_PLAN.md` - Detailed phase-by-phase implementation guide with file checklists
- `gradle/libs.versions.toml` - Dependency version catalog
- `composeApp/src/commonMain/sqldelight/` - Database schema (when created)

## Implementation Phases

1. **Foundation** - Dependencies, Koin DI, SQLDelight schema
2. **Domain/Data** - Models, repositories, use cases
3. **AI Integration** - Cactus SDK, task parser, speech-to-text
4. **UI** - Theme, navigation, screens (Capture, Review, Tasks, Settings)
5. **Platform** - Notifications (WorkManager/UNNotification), widgets
6. **Polish** - Error handling, performance, testing

## Development Notes

### Cactus SDK Initialization
Android requires context initialization in MainActivity:
```kotlin
CactusContextInitializer.initialize(this)
```

### SQLCipher Integration
- Encryption is **always on** — there is no user-facing toggle
- `linkSqlite = false` in SQLDelight config so SQLCipher provides the SQLite implementation
- iOS: SQLCipher added via CocoaPods with `isStatic = true`
- Android: `net.zetetic:sqlcipher-android` + `androidx.security:security-crypto`
- Encryption keys are generated once per install and stored in platform secure storage:
  - Android: `EncryptedSharedPreferences` (backed by Android Keystore)
  - iOS: Keychain with `kSecAttrAccessibleAfterFirstUnlock`
- Existing unencrypted databases are automatically migrated on first launch
- Key loss on app data clear is expected — privacy-first full reset

### Encryption Export Compliance
- **Google Play Store:** No encryption declarations required
- **Apple App Store:** `ITSAppUsesNonExemptEncryption = false` is set in `Info.plist`, which tells Apple the encryption is exempt from export compliance documentation. This is valid because SQLCipher is used solely for local data-at-rest protection (exempt under EAR 740.17(b)(1)), not for communications or external authentication

### Performance Targets
- App cold start: < 2s
- Voice transcription: < 2s for 10s audio
- LLM parse time: < 3s
- Memory peak (LLM): < 1.5 GB

## References

- [Cactus SDK Kotlin](https://github.com/cactus-compute/cactus-kotlin)
- [SQLDelight + SQLCipher](https://touchlab.co/multiplatform-encryption-with-sqldelight-and-sqlcipher)
- [NotelyVoice](https://github.com/tosinonikute/NotelyVoice) - Similar KMP architecture reference

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mduenas)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/mduenas)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
