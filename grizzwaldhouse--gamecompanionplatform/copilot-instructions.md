## gamecompanionplatform

> provides analytics, includes a paid feature system with activation codes, and

# Arcadia Tracker — Claude Code Project Instructions

## Project Overview
WPF game companion app (.NET 8.0 / C# 12) for Star Rupture. Tracks save data,
provides analytics, includes a paid feature system with activation codes, and
provides real-time QA telemetry with event-driven bug capture and reporting.

## Architecture
- `src/Core/` — Interfaces, models, Result<T> monad
- `src/Engine/` — Reusable services (Entitlements, SaveSafety, Tasks, UI, QATelemetry)
- `src/Modules/` — Game-specific (StarRupture, SaveModifier)
- `src/Apps/ArcadiaTracker.App/` — WPF application (MVVM)
- `tests/` — xunit + FluentAssertions test projects

### QA Telemetry Pipeline (Engine Layer — Game-Agnostic)
- `src/Engine/GameCompanion.Engine.QATelemetry/` — Event-driven QA infrastructure
  - `Interfaces/` — IEventBus, IContextBuffer, ISaveFileWatcher, IRuleEngine,
    ICaptureSessionManager, IGameAdapter, IReportBuilder
  - `Models/` — TelemetryEvent, DetectionRule, CaptureSession, SaveStateSnapshot
  - `Services/` — EventBus, ContextBuffer, SaveFileWatcher, RuleEngine,
    CaptureSessionManager, ReportBuilder

### Game Adapters (Module Layer — Game-Specific)
- `StarRuptureGameAdapter` — implements IGameAdapter for Star Rupture
- `TelemetryBridgeService` — wires LogisticsAnomalyService results into the event bus

## Key Conventions
- Use `Result<T>` monad for all domain error handling (no thrown exceptions)
- Use `required init` properties for immutable models
- Use `Unit` type for void success results
- Capabilities are HMAC-SHA256 signed tokens, not boolean flags
- Admin access: separate namespace (`admin.*`), never piggybacks on paid features
- Non-discoverability: premium UI only appears after capability check passes
- Atomic file writes: temp file + File.Move(overwrite: true)
- **Defensive Coding**: See `.claude/skills/defensive_coding.skill.md` for null safety and DI patterns.

## Coding Standards
- File-scoped namespaces (`namespace X;`)
- `sealed class` for all concrete implementations (no accidental inheritance)
- XML doc comments (`/// <summary>`) on all public types and members
- Comments should explain WHY, not WHAT — describe intent, not mechanics
- No AI-generated boilerplate comments; every comment should add value a human
  developer would find useful
- Keep methods focused — one concern per method
- Prefer composition over inheritance
- Use collection expressions (`[]`) for empty/inline collections

### Naming Conventions
- PascalCase for types, methods, properties, events
- camelCase with underscore prefix for private fields (`_fieldName`)
- Suffix services with `Service`, view models with `ViewModel`
- Suffix interfaces with their concern (e.g., `IEventBus`, not `IEventBusInterface`)

### Event-Driven Design (QA Telemetry)
- No polling — use OS-level events (FileSystemWatcher) and in-process event bus
- Rolling buffers instead of unbounded storage
- Separation of concerns: capture, detection, and reporting are independent
- Detection rules are data-driven (JSON config), not hardcoded
- All telemetry services implement game-agnostic interfaces; game logic lives
  in adapters that implement `IGameAdapter`

### Support Report Export (QA Feedback → Developer Portal)
- `QAFeedbackExportService.GenerateSupportReportText()` — full scan → plain-text report
- `QAFeedbackExportService.GenerateSingleAnomalySupportText()` — single anomaly report
- Reports are plain text (no markdown) — web forms render `**` and ```` literally
- Format: ASCII sections (`===`, `---`, `[ ]`), grouped by severity, with investigation hints
- Flow: generate report → copy to clipboard → open browser to support portal → user pastes
- Support portal URL: `https://starrupture-game.com/support/`
- Per-anomaly "Report to Support" button on each anomaly card in the scanner tab
- Bulk "Report to Support" action bar appears after scan finds issues

## Build & Test
- Build: `dotnet build src/Apps/ArcadiaTracker.App/ArcadiaTracker.App/ArcadiaTracker.App.csproj`
- Tests: xunit + FluentAssertions, test projects in `tests/`
- To verify code: review for compile errors, run existing test patterns
- QA Telemetry tests: `tests/GameCompanion.Engine.QATelemetry.Tests/`
- Support report tests: `tests/GameCompanion.Module.StarRupture.Tests/QAFeedbackExportServiceTests.cs`

## Security Rules
- All crypto uses System.Security.Cryptography (no custom implementations)
- HMAC comparison: always use CryptographicOperations.FixedTimeEquals()
- Encryption: AES-GCM only (not CBC)
- Key derivation: HKDF with domain-separated contexts
- Activation codes: HMAC-verified, one-time use
- Admin tokens: signed, encrypted at rest, time-bound (max 30 days)
- Never store secrets in plaintext

## Important Files
- Solution: `GameCompanionPlatform.sln` (root)
- DI wiring: `src/Apps/ArcadiaTracker.App/ArcadiaTracker.App/App.xaml.cs`
- Main window: `src/Apps/ArcadiaTracker.App/ArcadiaTracker.App/MainWindow.xaml.cs`
- Entitlements: `src/Engine/GameCompanion.Engine.Entitlements/`
- QA Telemetry: `src/Engine/GameCompanion.Engine.QATelemetry/`
- QA Feedback export: `src/Modules/GameCompanion.Module.StarRupture/Services/QAFeedbackExportService.cs`
- QA Feedback view: `src/Apps/ArcadiaTracker.App/ArcadiaTracker.App/Views/QAFeedbackView.xaml(.cs)`
- QA Feedback ViewModel: `src/Apps/ArcadiaTracker.App/ArcadiaTracker.App/ViewModels/QAFeedbackViewModel.cs`
- Admin architecture: `docs/ADMIN-RELEASE-ARCHITECTURE.md`
- Admin setup guide: `docs/ADMIN-SETUP-GUIDE.md`
- UX/monetization report: `docs/UX-MONETIZATION-REPORT.md`

## When Adding Features
1. Define capability action in `CapabilityActions` if paid
2. Create view in `Views/` with capability gating
3. Wire into `MainWindow.xaml` (nav item) and `MainWindow.xaml.cs` (routing)
4. Register services in `App.xaml.cs` ConfigureServices()
5. Write tests in corresponding test project
6. Keep premium nav items `Visibility="Collapsed"` by default

## When Adding QA Telemetry Features
1. Define new `TelemetryEventCategory` if needed
2. Create detection rules as JSON (not hardcoded C#)
3. Implement game-specific logic in the adapter (`IGameAdapter`), not in engine services
4. Wire new services in `App.xaml.cs` under the QA Telemetry Pipeline section
5. Write tests in `tests/GameCompanion.Engine.QATelemetry.Tests/`
6. Use the `TelemetryBridgeService` to connect existing scan results to the event bus

## When Adding Support Report Features
1. Add export method to `QAFeedbackExportService` returning `Result<string>`
2. Use plain text only — no markdown syntax (web forms render it literally)
3. Use ASCII structure: `===` headers, `---` dividers, indented key-value pairs
4. Add investigation hints per anomaly type via `GetInvestigationHint()`
5. Add ViewModel command with `[RelayCommand]` and can-execute guard
6. Wire click handler in code-behind: generate → clipboard → browser open
7. Handle `ExternalException` on clipboard and catch browser launch failures
8. Add tests in `QAFeedbackExportServiceTests.cs`

## Environment Setup
- .NET 8.0 SDK required (net8.0 for engine projects, net8.0-windows for WPF/modules)
- Visual Studio 2022+ or Rider recommended
- No external services required — all data stored locally under `%LOCALAPPDATA%\ArcadiaTracker\`
- Star Rupture save files expected at `%LOCALAPPDATA%\StarRupture\Saved\SaveGames\`

## Code Analysis Pipeline

### Overview
PowerShell-based static analysis that catches silent WPF crashes and convention
violations without requiring .NET SDK. Runs on PRs via GitHub Actions (~2 min).

### Running Locally
```bash
powershell -ExecutionPolicy Bypass -Command "& '.github/scripts/Invoke-CodeAnalysis.ps1' -RepositoryRoot 'C:\Users\daley\GameCompanionPlatform'"
```

### Checks (ERROR = blocks merge, WARNING = advisory)
| Check | Level | Script | Auto-fixable? |
|-------|-------|--------|--------------|
| Unsealed public classes | ERROR | `Test-SealedClasses.ps1` | Yes (`Add-SealedKeyword.ps1`) |
| Missing StaticResource keys | ERROR | `Test-XamlResources.ps1` | No |
| MessageBox in exception handlers | ERROR | `Test-ExceptionPatterns.ps1` | No |
| ProgressBar TwoWay binding | WARNING | `Test-WpfAntiPatterns.ps1` | Yes (`Fix-BindingModes.ps1`) |
| DependencyObject in Task.Run | WARNING | `Test-WpfAntiPatterns.ps1` | No |
| Implicit Template style | WARNING | `Test-WpfAntiPatterns.ps1` | No |
| Block-scoped namespaces | WARNING | `Test-Conventions.ps1` | No |
| throw in domain services | WARNING | `Test-Conventions.ps1` | No |

### Suppression
Add `<!-- analysis-ignore -->` comment on a XAML line to suppress StaticResource checks.

### Auto-Fix
Run via GitHub Actions "Auto-Fix" workflow (`workflow_dispatch`) or locally:
```bash
powershell -ExecutionPolicy Bypass -File ".github/scripts/fixes/Add-SealedKeyword.ps1" -RepositoryRoot .
powershell -ExecutionPolicy Bypass -File ".github/scripts/fixes/Fix-BindingModes.ps1" -RepositoryRoot .
```

### File Structure
```
.github/
  workflows/
    code-analysis.yml          — PR analysis (no .NET SDK needed)
    autofix.yml                — manual dispatch auto-fix → creates PR
  scripts/
    Invoke-CodeAnalysis.ps1    — orchestrator
    checks/                    — individual check scripts
    fixes/                     — auto-fix scripts
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/GrizzwaldHouse) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
