## puzzle-dazzle

> **Mazele Dazzle** is a mobile maze generation and play app for Android (iOS planned for future).

# Agent Instructions

## Project Overview

**Mazele Dazzle** is a mobile maze generation and play app for Android (iOS planned for future).

### Key Features
- Generate mazes with configurable size (10x10, 20x20, 30x30), difficulty (Easy/Medium/Hard), and shape (Rectangle/Circle/Diamond/Heart)
- Interactive maze play mode with joystick controls and pinch-to-zoom
- Save mazes as PNG images to device gallery
- Share via social networks/messengers
- Freemium model: 5 free mazes/day, unlimited with Google Play subscription

### Tech Stack
- **.NET 9** with **C# 13**, nullable reference types enabled, implicit usings enabled
- **MAUI** (Multi-platform App UI) - Android-only target (`net9.0-android`)
- **xUnit** for unit tests with **coverlet** for coverage
- **SkiaSharp** for image processing (export margins)
- **Google Play Billing Client** (`Xamarin.Android.Google.BillingClient`) for subscriptions
- Target: Android API 28+ (Android 9.0+)

### Documentation
- `docs/concept.en.md` - Full project concept and business model
- `docs/screens.en.md` - Screen descriptions and UI requirements
- `docs/monetization.md` - Subscription/freemium model details
- `docs/change-requests/` - Change request specifications

## Build, Test, and Run Commands

Development is done natively on Windows. MAUI builds do NOT work under WSL/Ubuntu.

```powershell
# Build entire solution
dotnet build PuzzleDazzle.sln

# Build Android app only
dotnet build src/PuzzleDazzle/PuzzleDazzle.csproj -f net9.0-android

# Build core library only (fast, no Android SDK needed)
dotnet build src/PuzzleDazzle.Core/PuzzleDazzle.Core.csproj

# Run all tests
dotnet test tests/PuzzleDazzle.Core.Tests/PuzzleDazzle.Core.Tests.csproj

# Run a single test by fully-qualified name
dotnet test tests/PuzzleDazzle.Core.Tests/PuzzleDazzle.Core.Tests.csproj --filter "FullyQualifiedName~MazeGeneratorTests.Generate_ValidDimensions_CreatesMazeWithCorrectSize"

# Run all tests in a single test class
dotnet test tests/PuzzleDazzle.Core.Tests/PuzzleDazzle.Core.Tests.csproj --filter "FullyQualifiedName~UsageTrackingServiceTests"

# Run on connected device/emulator (fast rebuild + deploy)
dotnet build src/PuzzleDazzle/PuzzleDazzle.csproj -f net9.0-android -t:Run

# Build release APK + AAB
dotnet publish src/PuzzleDazzle/PuzzleDazzle.csproj -f net9.0-android -c Release
```

**Notes:**
- The Core library targets `net9.0` (not Android), so `dotnet build` and `dotnet test` work from any OS/WSL.
- The MAUI app targets `net9.0-android` only and requires Android SDK on Windows.
- XAML Hot Reload is unreliable; use fast rebuild (`-t:Run`) for iteration.
- No linter or static analysis is configured. No CI/CD pipeline exists.

## Project Structure

```
PuzzleDazzle/
├── PuzzleDazzle.sln
├── src/
│   ├── PuzzleDazzle/                  # MAUI app (Android-only)
│   │   ├── Controls/                  # Custom UI controls (MazeView, JoystickView, PlayMazeView, EmojiAnimationView)
│   │   ├── Rendering/                 # Maze renderers (IMazeRenderer, ClassicMazeRenderer)
│   │   ├── Services/                  # App-level services (MazeExportService, MauiPreferencesService, subscription impls)
│   │   ├── Resources/                 # Fonts (FontAwesome, OpenSans), icons, styles
│   │   ├── Platforms/Android/         # Android-specific code (MainActivity, manifest)
│   │   ├── GenerationPage.xaml(.cs)   # Main screen - generate, save, share mazes
│   │   ├── MazePlayPage.xaml(.cs)     # Interactive maze play screen
│   │   ├── SettingsPage.xaml(.cs)     # Size/difficulty/shape settings
│   │   ├── PremiumUpgradePage.xaml(.cs) # Subscription purchase screen
│   │   └── MauiProgram.cs             # DI registration and app bootstrap
│   └── PuzzleDazzle.Core/            # Core library (platform-independent, net9.0)
│       ├── Models/                    # Maze, Cell, MazeShape, MazeDifficulty, MazeAlgorithm
│       ├── Generation/                # MazeGenerator + algorithm interface
│       │   └── Algorithms/            # RecursiveBacktrackingAlgorithm
│       └── Services/                  # IPreferencesService, ISubscriptionService, UsageTrackingService
└── tests/
    └── PuzzleDazzle.Core.Tests/       # xUnit tests (mirrors Core structure)
        ├── Models/                    # CellTests, MazeTests, MazeShapeTests
        ├── Generation/                # MazeGeneratorTests
        └── Services/                  # UsageTrackingServiceTests (with in-memory test doubles)
```

## Code Style Guidelines

### C# Conventions
- **Namespaces**: File-scoped (`namespace PuzzleDazzle.Core.Models;`), matching folder structure
- **Nullable**: Enabled globally. Use `null!` for properties initialized by framework. Use `?` for genuinely nullable types
- **Implicit usings**: Enabled. No need to import `System`, `System.Collections.Generic`, `System.Linq`, etc.
- **Access modifiers**: Explicit `public`/`private`. Default to `private` for fields and helper methods
- **Fields**: `_camelCase` with underscore prefix for private fields (`private readonly Random _random;`)
- **Properties**: PascalCase. Use `{ get; private set; }` for externally-read/internally-written properties
- **Methods**: PascalCase. Async methods use `Async` suffix (`GenerateAsync`, `CanGenerateAsync`)
- **Constants**: PascalCase (`private const string GenerationCountKey = "DailyGenerationCount";`)
- **Enums**: PascalCase members, defined in the same file as the primary class they relate to (e.g., `MazeDifficulty` in `Maze.cs`, `ShapeType` in `MazeShape.cs`)
- **Braces**: Allman style (opening brace on new line). Single-statement `if` bodies may omit braces
- **Indentation**: Tabs in MAUI app project, 4-space indentation in Core library. Follow existing file conventions
- **XML doc comments**: Required on all public types, interfaces, and non-trivial public methods using `<summary>` tags
- **Event handlers**: `async void` only for MAUI event handlers (e.g., `OnStartClicked`). All other async methods return `Task`/`Task<T>`

### Imports / Usings
- Rely on implicit usings for `System.*` namespaces
- Explicit `using` only for project namespaces (`using PuzzleDazzle.Core.Models;`)
- Order: framework usings first, then project usings
- No `global using` statements beyond the implicit ones (xUnit test project has `<Using Include="Xunit" />` in csproj)

### Architecture Patterns
- **Core library** (`PuzzleDazzle.Core`): Platform-independent. NO MAUI/Android references. Contains models, algorithms, and service interfaces
- **App project** (`PuzzleDazzle`): MAUI-specific implementations. References Core
- **Dependency inversion**: Core defines interfaces (`IPreferencesService`, `ISubscriptionService`), app provides implementations
- **DI registration** in `MauiProgram.cs` with `#if DEBUG` / `#else` for mock vs real services
- **Code-behind pattern**: Pages use XAML + code-behind (no MVVM ViewModels yet). Controls access elements by `x:Name`
- **Conditional compilation**: `#if DEBUG` for development conveniences (unlimited access, mock services, debug info in alerts)

### Error Handling
- Guard clauses with `ArgumentException` / `InvalidOperationException` for invalid inputs (see `Maze` constructor)
- `try/catch` with `DisplayAlert` for user-facing errors in page event handlers
- Catch-and-log pattern in background services (e.g., `GooglePlaySubscriptionService`)
- Return `null` for out-of-bounds access rather than throwing (see `Maze.GetCell`)
- Suppress non-critical errors silently (e.g., media scanner failures in `MazeExportService`)

### Testing Patterns
- **Framework**: xUnit with `[Fact]` and `[Theory]` / `[InlineData]` attributes
- **Naming**: `MethodName_Condition_ExpectedResult` (e.g., `Generate_ValidDimensions_CreatesMazeWithCorrectSize`)
- **Structure**: Arrange/Act/Assert pattern with comments separating sections
- **Test doubles**: In-memory implementations defined in the same test file (e.g., `InMemoryPreferencesService`, `TestSubscriptionService`)
- **Seeded randomness**: Use `new MazeGenerator(seed: 42)` for deterministic/reproducible tests
- **Test file location**: Mirrors source structure (`Core/Models/Maze.cs` -> `Core.Tests/Models/MazeTests.cs`)
- **Only Core is tested**: The test project only covers `PuzzleDazzle.Core`. MAUI app code has no automated tests

### XAML Conventions
- XML namespace aliases: `controls` for `PuzzleDazzle.Controls`, `local` for `PuzzleDazzle`
- Colors via `{DynamicResource}` referencing `Colors.xaml` (Primary, Gray100-900)
- Font Awesome icons via `{x:Static local:FontAwesomeIcons.IconName}` with `FontFamily="FontAwesome"`
- Semantic properties (`SemanticProperties.Hint`) on interactive elements for accessibility

## Issue Tracking

This project uses **bd** (beads) for issue tracking. Run `bd onboard` to get started.

## Quick Reference

```bash
bd ready              # Find available work
bd show <id>          # View issue details
bd update <id> --status in_progress  # Claim work
bd close <id>         # Complete work
bd sync               # Sync with git
```

## Landing the Plane (Session Completion)

**When ending a work session**, you MUST complete ALL steps below. Work is NOT complete until `git push` succeeds.

**MANDATORY WORKFLOW:**

1. **File issues for remaining work** - Create issues for anything that needs follow-up
2. **Run quality gates** (if code changed) - Tests, linters, builds
3. **Update issue status** - Close finished work, update in-progress items
4. **PUSH TO REMOTE** - This is MANDATORY:
   ```bash
   git pull --rebase
   bd sync
   git push
   git status  # MUST show "up to date with origin"
   ```
5. **Clean up** - Clear stashes, prune remote branches
6. **Verify** - All changes committed AND pushed
7. **Hand off** - Provide context for next session

**CRITICAL RULES:**
- Work is NOT complete until `git push` succeeds
- NEVER stop before pushing - that leaves work stranded locally
- NEVER say "ready to push when you are" - YOU must push
- If push fails, resolve and retry until it succeeds

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/alkozlov)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/alkozlov)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
