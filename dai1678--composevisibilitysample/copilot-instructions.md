## composevisibilitysample

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ComposeVisibilitySample is a Kotlin Multiplatform (Android + iOS) demo app showcasing Jetpack Compose's `onVisibilityChanged` API. The app displays images from Lorem Picsum API in a scrollable list and logs visibility events when an image is 50% visible for 1+ second.

**Key Documentation:**
- `docs/requirements.md` - Functional requirements, API specs, log format
- `docs/architecture.md` - Detailed architecture, testing strategy, implementation status
- `README.md` - Quick start guide

## Commands

### Build & Run
```bash
# Android
./gradlew :composeApp:assembleDebug
./gradlew :composeApp:installDebug

# iOS framework
./gradlew :composeApp:linkDebugFrameworkIosSimulatorArm64
```

### Testing
```bash
# Run all tests (39 tests total)
./gradlew :composeApp:testDebugUnitTest

# Run tests with output
./gradlew :composeApp:testDebugUnitTest --rerun-tasks

# Run specific test class
./gradlew :composeApp:testDebugUnitTest --tests "dev.dai.compose.visibility.sample.ui.viewmodel.ImageListViewModelTest"

# Run single test
./gradlew :composeApp:testDebugUnitTest --tests "dev.dai.compose.visibility.sample.ui.viewmodel.ImageListViewModelTest.uiState is Success when images are loaded successfully"
```

### Other Commands
```bash
# View dependencies
./gradlew :composeApp:dependencies

# Compile common Kotlin metadata
./gradlew :composeApp:compileCommonMainKotlinMetadata

# Clean build
./gradlew clean
```

## Architecture

This project follows Android Architecture Guidelines with strict layer separation:

### Three-Layer Architecture

```
UI Layer (commonMain/ui)
    ↓ StateFlow
Domain Layer (commonMain/domain)
    ↓ Result<T>
Data Layer (commonMain/data)
```

**Critical Design Patterns:**

1. **Unidirectional Data Flow (UDF)**: ViewModel exposes `StateFlow<ImageListUiState>`, UI observes and reacts
2. **Result Type for Error Handling**: All repository/use case methods return `Result<T>`, never throw exceptions upward
3. **Dependency Injection with Koin**: All dependencies managed through `di/AppModule.kt`
4. **expect/actual for Platform-Specific Code**: `HttpClientFactory` uses expect/actual pattern
   - `commonMain/di/HttpClientFactory.kt` (expect declaration)
   - `androidMain/di/HttpClientFactory.android.kt` (OkHttp engine)
   - `iosMain/di/HttpClientFactory.ios.kt` (Darwin engine)

### Key Architectural Rules

- **Domain Models are Platform-Independent**: `ImageItem`, `VisibilityLog`, `MicrosecondTimestamp` in `domain/model`
- **DTOs Only in Data Layer**: `ImageResponse` in `data/model`, converted via `toDomainModel()` extension
- **Use Cases are Mandatory**: All business logic must go through use cases, never call repositories directly from ViewModel
- **ViewModel is Shared**: `ImageListViewModel` in `commonMain` shared between Android and iOS

### Package Structure

```
composeApp/src/
├── commonMain/kotlin/dev/dai/compose/visibility/sample/
│   ├── ui/          # Composables, ViewModels
│   │   ├── screen/      # ImageListScreen
│   │   ├── component/   # ImageCard, ErrorView, LoadingView
│   │   └── viewmodel/   # ImageListViewModel
│   ├── domain/      # Use Cases, Models, Repository Interfaces
│   │   ├── model/       # ImageItem, VisibilityLog, MicrosecondTimestamp
│   │   ├── repository/  # ImageRepository (interface)
│   │   └── usecase/     # GetImagesUseCase, LogImageVisibilityUseCase
│   ├── data/        # Repository Implementations, Data Sources, DTOs
│   │   ├── repository/  # ImageRepositoryImpl
│   │   ├── remote/      # RemoteDataSource, PicsumRemoteDataSource
│   │   └── model/       # ImageResponse (DTO)
│   └── di/          # AppModule (Koin), HttpClientFactory (expect/actual)
│
├── commonTest/      # Domain/Data layer tests (Kotlin Test)
│   ├── domain/
│   ├── data/
│   └── fake/        # FakeImageRepository, FakeRemoteDataSource
│
└── androidUnitTest/ # ViewModel tests (JUnit + MainDispatcherRule)
    ├── ui/viewmodel/
    └── util/        # MainDispatcherRule
```

## Testing Strategy

### Test Distribution
- **commonTest** (30 tests): Domain models, use cases, repositories (uses Kotlin Test)
- **androidUnitTest** (9 tests): ViewModel (uses JUnit + MainDispatcherRule)
- **Total**: 39 tests, 80%+ coverage

### Testing Patterns
1. **Fake Pattern Preferred**: Use `FakeImageRepository`, `FakeRemoteDataSource` from `commonTest/fake`
2. **MainDispatcherRule for ViewModel Tests**: Always use `@get:Rule val mainDispatcherRule = MainDispatcherRule()` in ViewModel tests
3. **No MockK in Common Code**: MockK only for platform-specific classes in `androidUnitTest`

### Test File Locations
- Domain/Data tests: `commonTest/` (can run on all platforms)
- ViewModel tests: `androidUnitTest/` (requires JUnit, MainDispatcherRule)

## onVisibilityChanged Implementation

The core feature uses `Modifier.onVisibilityChanged` in `ImageCard.kt`:

```kotlin
Modifier.onVisibilityChanged(
    minFractionVisible = 0.5f,  // 50% threshold
    minDurationMs = 1000L,       // 1 second duration
    onVisibilityChanged = { imageId, position ->
        viewModel.onImageVisible(imageId, position)
    }
)
```

**Log Format** (output by `LogImageVisibilityUseCase`):
```json
{"id":"0","position":1,"time":"1234567890.123456"}
```

- `time` field uses `MicrosecondTimestamp` value class
- Format: "seconds.microseconds" (10 digits.6 digits with zero padding)

## Platform-Specific Code

### HTTP Client Configuration
- **Android**: OkHttp engine (official recommendation)
- **iOS**: Darwin engine (native URLSession, TLS support)
- **Logging**: Napier integration with `LogLevel.ALL`

Example in `HttpClientFactory.android.kt`:
```kotlin
actual fun createHttpClient(): HttpClient {
    return HttpClient(OkHttp) {
        install(Logging) {
            logger = object : Logger {
                override fun log(message: String) {
                    Napier.v(message = message, tag = "HTTP Client")
                }
            }
            level = LogLevel.ALL
        }
        // ...
    }.also { Napier.base(DebugAntilog()) }
}
```

## Common Development Patterns

### Adding a New Domain Model
1. Create in `commonMain/domain/model/`
2. Keep platform-independent (no Android/iOS specific types)
3. Add tests in `commonTest/domain/model/`

### Adding a New Use Case
1. Create in `commonMain/domain/usecase/`
2. Inject repository via constructor
3. Return `Result<T>` for operations that can fail
4. Register in `di/AppModule.kt` as `factory { YourUseCase(get()) }`
5. Add tests in `commonTest/domain/usecase/`

### Adding a New Repository Method
1. Add to interface in `domain/repository/`
2. Implement in `data/repository/` with try-catch returning `Result<T>`
3. Add tests using `FakeRemoteDataSource` in `commonTest/data/repository/`

### Testing ViewModels
1. Place test in `androidUnitTest/ui/viewmodel/`
2. Always add `@get:Rule val mainDispatcherRule = MainDispatcherRule()`
3. Use `runTest { }` for coroutine tests
4. Use `advanceUntilIdle()` to wait for coroutines
5. Use `FakeImageRepository` for isolation

## Important Constraints

**Out of Scope** (do not implement):
- Pagination/infinite scroll
- Offline support/local caching
- Image detail screens
- Favorite functionality
- Search functionality

**Git Workflow**:
- Commit directly to `main` branch (no feature branches)
- Do not push until a complete feature set is done

**Documentation Updates**:
When making significant changes, update:
- `docs/architecture.md` for architectural changes
- `docs/requirements.md` for requirement changes
- `README.md` for setup/build instructions
- This file (`CLAUDE.md`) for development patterns

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Dai1678) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
