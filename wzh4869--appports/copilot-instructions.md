## appports

> This file provides guidance to AI coding assistants when working with code in this repository.

# AI Assistant Guide

This file provides guidance to AI coding assistants when working with code in this repository.

## Guiding Principles (MUST FOLLOW)

- **Keep it clear**: Write Swift code that is easy to read, maintain, and explain. Prefer clarity over cleverness.
- **Match the house style**: Reuse existing patterns, naming, and conventions found in the codebase.
- **Search smart**: Use glob/grep for codebase exploration before making assumptions about structure or patterns.
- **Log centrally**: Route all logging through `AppLogger.shared` with the right context—never use `print` in production code.
- **Always propose before executing**: Before making any changes, clearly explain your planned approach and wait for explicit user approval.
- **Build and test before completion**: Coding tasks are only complete after a successful `xcodebuild clean build` (Release). If the change touches a tested module, run the corresponding test suite.
- **Write conventional commits**: Commit small, focused changes using Conventional Commit messages (`feat:`, `fix:`, `docs:`, `refactor:`, `test:`).
- **Build after every code change**: Verify compilation succeeds before delivering any code change.

## Project Overview

AppPorts is a native macOS desktop app (Swift/SwiftUI) that migrates applications from `/Applications` to external storage while keeping a functional local "portal" via Stub Portal (launcher script). It also supports migrating data directories (`~/Library/` subfolders and dot-folders like `~/.npm`) and user-selected custom folders. Minimum deployment target: macOS 12.0 (Monterey).

## Build & Test Commands

If Xcode is installed outside `/Applications`, export `DEVELOPER_DIR` before running `xcodebuild`:
```bash
export DEVELOPER_DIR=/Volumes/hano/Applications/Xcode.app/Contents/Developer
```

**Build** (Xcode project, no SPM):
```bash
xcodebuild clean build -scheme "AppPorts" -configuration Release -destination 'platform=macOS' \
  CODE_SIGN_IDENTITY="" CODE_SIGNING_REQUIRED=NO CODE_SIGN_ENTITLEMENTS="" CODE_SIGNING_ALLOWED=NO
```

**Run all tests:**
```bash
xcodebuild test -scheme "AppPorts" -configuration Debug -destination 'platform=macOS' \
  CODE_SIGN_IDENTITY="" CODE_SIGNING_REQUIRED=NO CODE_SIGN_ENTITLEMENTS="" CODE_SIGNING_ALLOWED=NO
```

**Run a single test class** (e.g. `DataDirScannerTests`):
```bash
xcodebuild test -scheme "AppPorts" -destination 'platform=macOS' \
  -only-testing:"AppPortsTests/DataDirScannerTests" \
  CODE_SIGN_IDENTITY="" CODE_SIGNING_REQUIRED=NO CODE_SIGNING_ALLOWED=NO
```

**Run localization audit:**
```bash
xcodebuild test -scheme "AppPorts" -destination 'platform=macOS,arch=arm64' \
  -only-testing:"AppPortsTests/LocalizationAuditTests" \
  CODE_SIGNING_ALLOWED=NO -derivedDataPath /tmp/AppPortsDerived
```

### Test Suites Overview

| Test Suite | Module | When to Run |
|------------|--------|-------------|
| `DataDirMoverTests` | Data directory migration | When touching `DataDirMover` |
| `DataDirScannerTests` | Data directory scanning | When touching `DataDirScanner` |
| `CustomDirScannerTests` | Custom directory configs/scanning/validation | When touching `CustomDirModels`, `CustomDirScanner`, or custom directory UI |
| `AppMigrationServiceTests` | App migration | When touching `AppMigrationService` |
| `AppScannerTests` | App scanning | When touching `AppScanner` |
| `AppLoggerTests` | Logging & diagnostics | When touching `AppLogger` |
| `LocalizationAuditTests` | Localization | When touching user-facing copy |
| `UpdateCheckerTests` | Release/update lookup | When touching `UpdateChecker` |

## Architecture

### Directory Structure

```
AppPorts/
├── AppPorts.xcodeproj/             # Xcode project (no SPM, no external deps)
├── AppPorts/                       # Main source
│   ├── Appports.swift              # @main entry point + AppDelegate
│   ├── ContentView.swift           # Main window and top-level tab routing
│   ├── WelcomeView.swift           # First-launch welcome screen
│   ├── AboutView.swift             # About dialog with GitHub contributors
│   ├── Localizable.xcstrings       # String catalog (20+ languages)
│   ├── Models/
│   │   ├── AppModels.swift         # AppItem, AppMoverError, AppContainerKind
│   │   ├── DataDirItem.swift       # DataDirItem, DataDirType, DataDirPriority
│   │   ├── CustomDirModels.swift   # CustomDirConfig, validation, display entries
│   │   └── AppLanguageOption.swift # Language catalog (AppLanguageCatalog)
│   ├── Views/
│   │   ├── DataDirsView.swift      # Tool dirs + app data management
│   │   ├── CustomDirsView.swift    # User-selected folder migration tab
│   │   ├── AppStoreSettingsView.swift # Settings sheet
│   │   └── Components/
│   │       ├── AppIconView.swift   # Async app icon loader
│   │       ├── AppRowView.swift    # App list row + context menu
│   │       ├── DataDirRowView.swift # Data dir row + tree indentation
│   │       ├── CustomDirRowView.swift # Custom folder row
│   │       ├── StatusBadge.swift   # Status pill badges (link/framework/type)
│   │       ├── ProgressOverlay.swift # Migration progress overlay
│   │       └── HelpButton.swift    # Popover help button
│   ├── Services/
│   │   ├── AppMigrationService.swift # Core migration engine (~1500 lines)
│   │   ├── AppLogger.swift         # Logging + diagnostics export (1116 lines)
│   │   └── CodeSigner.swift        # Code signing backup/restore (599 lines)
│   └── Utils/
│       ├── AppScanner.swift        # App scanner actor (1060+ lines)
│       ├── DataDirScanner.swift    # Data dir scanner actor (~1400 lines)
│       ├── DataDirMover.swift      # Data dir migration actor (1003 lines)
│       ├── CustomDirScanner.swift  # Custom folder status scanner
│       ├── FileCopier.swift        # File copy with progress (458 lines)
│       ├── FolderMonitor.swift     # DispatchSource filesystem watcher
│       ├── LanguageManager.swift   # Global i18n manager + String.localized
│       ├── UpdateChecker.swift     # GitHub release checker
│       └── LocalizedByteCountFormatter.swift
├── AppPortsTests/                  # Unit tests
│   ├── AppMigrationServiceTests.swift
│   ├── AppScannerTests.swift
│   ├── CustomDirScannerTests.swift
│   ├── DataDirScannerTests.swift
│   ├── DataDirMoverTests.swift
│   ├── AppLoggerTests.swift
│   ├── LocalizationAuditTests.swift
│   └── UpdateCheckerTests.swift
└── User_docs/                      # VitePress documentation site
```

### Build Settings

| Setting | Value |
|---------|-------|
| Bundle ID | `com.shimoko.AppPorts` |
| Marketing Version | `1.8.0` (`MARKETING_VERSION` in `project.pbxproj`) |
| Deployment Target | macOS 12.0 (Monterey) |
| Swift Version | 5.0 |
| App Sandbox | **Disabled** (required for /Applications access) |
| Hardened Runtime | Enabled |
| External Dependencies | **None** (pure Apple frameworks) |
| Info.plist | Auto-generated (`GENERATE_INFOPLIST_FILE = YES`) |
| Entitlements | None (no sandbox) |
| UI Framework | Pure SwiftUI (no storyboards/xibs) |

### Core Pattern: Actor-based Concurrency

All heavy operations run as Swift `actor` types to ensure thread safety off the main thread:
- `AppScanner` — scans `/Applications` and external drive for apps
- `DataDirScanner` — scans `~/Library/` and known dot-folders for associated data
- `DataDirMover` — migrates/restores data directories
- `CustomDirScanner` — scans saved custom folder migration configs
- `FileCopier` — file copy with progress callback
- `CodeSigner` — ad-hoc code signing with original signature backup/restore

### Component Dependency Graph

```
    Appports.swift (@main)
         │
    ┌────┴────┐
    ▼         ▼
WelcomeView  ContentView
             │
    ┌────────┼────────────┬──────────────┐
    ▼        ▼            ▼              ▼
 Apps UI  DataDirsView  CustomDirsView  Settings/
                                            AboutView
    │        │            │
    ▼        ▼            ▼
┌─────────────────────────────────────────┐
│           Services & Utils              │
│                                         │
│  AppMigrationService ──► FileCopier     │
│       │                   CodeSigner    │
│       ▼                                 │
│  AppScanner ─────────► AppItem          │
│                                         │
│  DataDirScanner ─────► DataDirItem      │
│  DataDirMover ───────► FileCopier       │
│                                         │
│  CustomDirScanner ───► CustomDirPair    │
│                          CustomDirConfig │
├─────────────────────────────────────────┤
│        Cross-Cutting (Singletons)       │
│  AppLogger.shared ◄── used everywhere   │
│  LanguageManager.shared ◄── all views   │
│  FolderMonitor ◄── ContentView          │
│  UpdateChecker ◄── ContentView          │
└─────────────────────────────────────────┘
```

### App Lifecycle

1. **Launch** → `AppMoverApp` (@main) → `AppLogger.shared.logLaunchSession()` logs system diagnostics
2. **First launch** → `WelcomeView` (feature cards, Full Disk Access permission check, language switcher)
3. **Main app** → `ContentView` with three top-level tabs:
   - **Apps tab**: `HSplitView` — local apps (left) / external apps (right). Multi-select → batch migrate/link/restore
   - **Data Dirs tab**: `DataDirsView` with two sub-tabs — Tool Dirs (`~/.npm`, `~/.m2`, `.gradle`, `.android`, `.pub-cache`, etc.) and App Data (`~/Library/` subdirs)
   - **Directory Migration tab**: `CustomDirsView` for arbitrary user folders under the current user's home directory
4. **Background**: `FolderMonitor` watches `/Applications`, external drive, and user-configured custom local app scan dirs with 1s debounce for auto-rescan
5. **Menu bar**: Language switcher (20+ languages), log management, diagnostics export

### File Responsibilities

#### Entry Point & Top-Level Views

| File | Role |
|------|------|
| `Appports.swift` | `@main` entry, `AppDelegate` (prevents terminate on last window close), menu bar commands, `LanguageManager` locale injection |
| `ContentView.swift` | Main view: app list management, migration/link/restore operations, top-level tab switcher (`apps`, `dataDirs`, `customDirs`), FolderMonitor integration, debounced rescanning, custom local app scan directories (persisted via UserDefaults, with per-directory FolderMonitor), Stub Portal version sync on rescan, inline helper views (`HeaderView`, `ActionFooter`, `EmptyStateView`, `TabButton`). Real-path resolution for linked apps (`resolveRealAppURL` — parses stub portal launcher script or symlink target). URL-based signing helpers (`performResign(at:bundleID:silent:)`, `performBackupSignature(at:bundleID:)`, `getBundleIdentifier(from:)`) for data directory migration signing flow |
| `WelcomeView.swift` | First-launch screen: feature cards, Full Disk Access guidance, language switcher |
| `AboutView.swift` | About dialog: version info, contributors (fetched from GitHub API with disk cache), links |
| `DataDirsView.swift` | Built-in data directory UI: tool dirs and app-associated data. Passes the selected external root into scanner calls so missing local entries can surface as `待接回`. |
| `CustomDirsView.swift` | Custom folder migration UI: two-pane local/external list, add sheet, batch migrate/relink/restore, progress overlay on the parent view. Reuses `DataDirMover` through `CustomDirEntry.dataDirItem`. |

#### Models

| File | Key Types |
|------|-----------|
| `AppModels.swift` | `AppItem` (name, path, status, flags: isSystemApp/isRunning/isAppStoreApp/isIOSApp/isResigned/isElectronApp/isSparkleApp/hasSelfUpdater/needsLock, size, containerKind), `AppMoverError`, `AppContainerKind` (.standaloneApp/.singleAppContainer/.appSuiteFolder) |
| `DataDirItem.swift` | `DataDirItem` (name, path, type, priority, status, size, linkedDestination, tree children), `DataDirType` (12 types), `DataDirPriority` (.critical/.recommended/.optional), `DataDirError` |
| `CustomDirModels.swift` | `CustomDirConfig` (local path, external base, computed external destination), `CustomDirStatus`, `CustomDirEntry`, `CustomDirPair`, `CustomDirValidator`, `CustomDirLocalOpenPanelGuard` |
| `AppLanguageOption.swift` | `AppLanguageOption`, `AppLanguageCatalog` — 3 primary + 16 AI-translated languages |

#### Services

| File | Role |
|------|------|
| `AppMigrationService.swift` | Core migration engine: move-and-link (copy→delete→create portal), link app, delete link, move back. Portal strategy selection, self-updater detection (Sparkle/Electron/custom), uchg lock/unlock, Finder-based MAS app deletion, macOS 15.1+ MAS external install, rollback on failure. Stub Portal version sync (`refreshStubPortal`) updates local Info.plist when external app is updated via App Store, with Launch Services refresh (`lsregister -f`) |
| `AppLogger.swift` | Singleton logger: file logging with rotation (2MB default), multi-level (INFO/ERROR/DIAG/DISK/PERF/TRACE/WARN), system diagnostics (hardware/software/disk), operation summaries (JSON), diagnostic package export (ZIP with redacted logs) |
| `CodeSigner.swift` | Actor: ad-hoc re-signing (`codesign --force --sign -`), signature backup/restore via plist files in `~/Library/Application Support/AppPorts/signature-backups/`, handles symlinked Contents (temp real copy for signing), root-installed app permission elevation via AppleScript, retry logic |

#### Utils

| File | Role |
|------|------|
| `AppScanner.swift` | Actor: scans /Applications, external dirs, and user-configured custom local app scan dirs. Detects portal types (wholeApp/deepContents/stubPortal), system/running/AppStore/iOS/Electron/Sparkle/self-updater apps, resigned status, folder containers, deduplication by bundleID/name, macOS 15.1+ MAS external scanning. Info.plist in-memory cache (per-scan) reduces redundant disk reads. `codesign -dvv` timeout protection (10s). `calculateDirectorySize` has 500k file count safety cap. `resolveExternalRealApp(from:)` — parses stub portal launcher or symlink to find real external app for resigned status checking |
| `DataDirScanner.swift` | Actor: scans 30+ known dotFolders (npm, maven, Gradle, Android, Flutter/Dart, bun, conda, ollama, torch, whisper, cursor, vscode, docker, etc.), ~/Library/ subdirs matching by bundleID/appName, tree construction, status detection, managed link metadata verification. `scanKnownDotFolders(externalRootURL:)` surfaces missing local tool dirs as `待接回` when the canonical external directory exists. Bundle ID suffix extraction filters generic TLD words (app, com, org, etc.) to avoid over-matching container directories |
| `CustomDirScanner.swift` | Actor: scans saved `CustomDirConfig` entries and returns local/external `CustomDirPair` state (`本地`, `已链接`, `待接回`, `孤立链接`, `目标冲突`, `未找到`). |
| `DataDirMover.swift` | Actor: migrate (copy→backup local source→symlink), restore (delete symlink→copy back), create link, normalize managed link, `.appports-link-metadata.plist` management, conflict detection, interrupted migration recovery, protected path detection. Used by both built-in data directories and custom directory migration. |
| `FileCopier.swift` | Actor: recursive directory copy preserving permissions/xattrs/timestamps, byte-level progress callbacks (5MB/50-file thresholds), symlink handling, socket skipping, EINTR retry for external storage |
| `FolderMonitor.swift` | DispatchSource (kqueue) filesystem watcher with 1s debounce |
| `LanguageManager.swift` | Singleton `ObservableObject`: language selection (UserDefaults), `Locale` for SwiftUI environment, `String.localized` extension with .lproj fallback chain |
| `UpdateChecker.swift` | Fetches latest release from GitHub API, semantic version comparison |

### Migration Strategies

AppPorts picks a local portal strategy per app:

1. **Stub Portal** (default for all `.app` bundles): creates a minimal fake `.app` shell with a bash launcher script that `open`s the real external app. No symlinks, no arrow icon, self-updaters cannot penetrate. Two sub-variants:
   - **macOS Stub Portal**: for native macOS apps
   - **iOS Stub Portal**: for iOS apps on Mac (icon extraction from `WrappedBundle/`)

2. **Whole App Symlink** (folders/suites, non-`.app` paths): symlinks the entire `.app` bundle or directory.

3. **Deep Contents Wrapper** (legacy, deprecated): only detected during restore of old migrations. No longer used for new portals.

`AppMigrationService.preferredPortalKind(for:)` decides which strategy to use by inspecting the bundle.

### Self-Updater Detection

AppPorts detects self-updating apps and applies lock protection (`chflags -R uchg`) to prevent external copies from being modified:

- **Sparkle**: `Sparkle.framework` / `Squirrel.framework`, Info.plist keys (`SUFeedURL`, etc.)
- **Electron**: `Electron Framework.framework`, `app-update.yml`
- **Custom updaters**: `LaunchServices/`, `KSProductID`, binaries containing "update"

### Data Directory Link Management

`DataDirScanner` and `DataDirMover` both track managed symlinks using `.appports-link-metadata.plist` sidecar files in the external destination. This distinguishes AppPorts-created links from pre-existing symlinks. Status values: `本地` (local), `已链接` (linked), `待接回` (awaiting relink), `现有软链` (pre-existing symlink), `待规范` (needs normalization).

Known tool directories use canonical external destinations under `<externalRoot>/<DataDirType.rawValue>/<lastPathComponent>`, for example `<externalRoot>/工具目录/.gradle`. `scanKnownDotFolders(externalRootURL:)` should hide items that are missing locally and externally, but surface `待接回` when the canonical external directory already exists.

Custom directory migration stores configs in `UserDefaults` under `customDirConfigs`. A config keeps the real local source and an external base directory; the actual destination is `externalBaseURL / localURL.lastPathComponent`. Local sources must be real directories under the current user's home, cannot be the home directory itself, cannot pass through symlinked path components, and cannot overlap with another managed custom directory.

### Real-Path Resolution for Linked Apps

For linked apps (status `已链接`), the local path may be a Stub Portal shell or a whole-app symlink — neither is the real application package. Signing operations (resign, backup, restore) must target the real external app to take effect. Two resolution methods:

- **`resolveRealAppURL(for:)`** in `ContentView.swift` — resolves the real app path for any `AppItem`:
  - Whole App Symlink: resolves symlink target via `FileManager.destinationOfSymbolicLink(atPath:)`
  - Stub Portal: extracts `REAL_APP='...'` from the `Contents/MacOS/launcher` script using regex
  - Non-linked apps: returns `app.displayURL` as-is
- **`resolveExternalRealApp(from:)`** in `AppScanner.swift` — same logic, used for resigned status detection. Checks both the local Bundle ID and the real external app's Bundle ID for signature backup existence.

Both methods are `nonisolated` (no MainActor dependency) and support the two active portal strategies. `Deep Contents Wrapper` is legacy and not handled here.

### Key Data Models

- `AppItem` (in `Models/AppModels.swift`): represents an app with name, path, status, flags (isSystemApp, isRunning, isAppStoreApp, isIOSApp, isResigned), containerKind, and size info.
- `DataDirItem` (in `Models/DataDirItem.swift`): represents a data directory with type (applicationSupport, containers, caches, dotFolder, etc.), priority, and link status.
- `CustomDirConfig` / `CustomDirPair` (in `Models/CustomDirModels.swift`): represents a user-selected folder migration and its local/external row state.
- `AppContainerKind`: `.standaloneApp`, `.singleAppContainer`, `.appSuiteFolder`

### UI Structure

- `Appports.swift`: `@main` entry, handles `WelcomeView` → `ContentView` flow, menu bar commands (language, logging)
- `ContentView`: main window with top-level tab switcher (Apps / Data Dirs / Directory Migration). Apps tab is an `HSplitView` — local apps left, external apps right
- `DataDirsView`: built-in data directory tab with Tool Dirs and App Data sub-tabs
- `CustomDirsView`: directory migration tab for arbitrary user folders, with local and external panes

### Localization System

- Strings live in `AppPorts/Localizable.xcstrings` (20+ languages)
- `LanguageManager` singleton with `.localized` extension on `String` for imperative code
- SwiftUI `Text("key")` literals are acceptable for `LocalizedStringKey` APIs
- AppKit/imperative strings must use `.localized` explicitly
- Language catalog in `AppLanguageCatalog` (in `Models/AppLanguageOption.swift`) — the single source of truth for supported languages
- `LocalizationAuditTests` validates translation coverage, placeholder consistency, absence of empty keys, absence of stale string catalog entries, and active `.localized` / `NSLocalizedString` references
- Hidden SwiftUI controls must still use meaningful localized labels; empty labels can create an empty string catalog key
- When adding new user-facing fields, status values, settings, alerts, errors, or labels, ask the maintainer whether to complete translations for all supported languages in the same change. If not, document the intended fallback/follow-up clearly.

### Global Singletons

- `AppLogger.shared` — structured logging with rotation, diagnostics export
- `LanguageManager.shared` — locale management, injected via `.environment(\.locale, ...)`
- `UpdateChecker.shared` — checks GitHub Releases for updates

### Real-time Monitoring

`FolderMonitor` uses `DispatchSource` (kqueue) to watch `/Applications`, the external drive, and custom local app scan dirs, triggering automatic re-scans on filesystem changes.

### Key Architectural Patterns

- **Actor isolation**: all file operations are actor-isolated for thread safety
- **MVVM-ish**: views directly call service/utility methods; no formal ViewModel layer (except `ContributorsViewModel` in AboutView)
- **@AppStorage**: persistent settings (allowAppStoreMigration, allowIOSAppMigration, autoResignEnabled, LogEnabled, MaxLogSizeBytes)
- **UserDefaults**: external drive path, language selection, log configuration, custom local app scan paths (`customLocalScanPaths`), custom directory migration configs (`customDirConfigs`)
- **Progress callbacks**: `FileCopier.ProgressHandler` is `@Sendable (Progress) async -> Void` for real-time UI updates from actor contexts
- **Managed link metadata**: `.appports-link-metadata.plist` sidecar files for authoritative link tracking
- **Real-path signing for linked apps**: signing operations (resign, backup, restore) always resolve to the real external app path via `resolveRealAppURL`/`resolveExternalRealApp`, never the local stub shell. This ensures signature changes take effect on the actual app package.
- **Stub Portal version sync**: when `FolderMonitor` or manual refresh detects an external app version change, `AppMigrationService.refreshStubPortal` updates the local stub portal's Info.plist, icon, and ad-hoc signature, then calls `lsregister -f` to refresh macOS Launch Services cache.
- **Structured error logging**: `AppLogger.shared.logError` accepts `errorCode` (e.g., `BACKUP-SIGNATURE-FAILED`, `RESIGN-FAILED`, `DATA-RESIGN-FAILED`) and `relatedURLs` for machine-parseable error tracking. Data directory operations include `appContextFields()` context (app_name, app_status, app_bundle_id, app_real_path, app_is_resigned).
- **Diagnostic export**: ZIP with redacted logs, operation summaries, system metadata
- **Zero external dependencies**: pure Apple frameworks only (SwiftUI, AppKit, Foundation, Combine)

## CI Configuration

- **PR smoke check** (`build.yml`): compilation-only build in Release mode. **Blocking.**
- **Data directory tests** (`build.yml`): runs `DataDirMoverTests` + `DataDirScannerTests`. Advisory (non-blocking).
- **Localization audit** (`build.yml`): runs `LocalizationAuditTests`. Advisory (non-blocking).
- **Post-merge** (`post-merge-validation.yml`): `DataDirMoverTests` + `DataDirScannerTests` + localization audit + Release build. Runs on push to `main`/`develop`.

## Branch Convention

- `main` — stable releases
- `develop` — integration branch; feature branches target `develop`
- Commit style: `feat: ...`, `fix: ...`, `docs: ...`, `refactor: ...`, `test: ...`

## Pull Request Workflow

When creating a Pull Request:

1. Ensure the branch is up-to-date with `develop`
2. Fill in all required fields in the PR template
3. Run `xcodebuild clean build` (Release) — this is mandatory
4. If the PR touches a tested module, run the corresponding test suite
5. For UI changes, attach screenshots

## Issue Workflow

When reporting a bug or proposing a feature:

1. Search existing Issues first to avoid duplicates
2. For bugs, attach a diagnostic package (Menu → 日志 → 导出诊断包)
3. For core feature proposals (migration strategy, data directory, code signing), detailed technical discussion is required before implementation

## Development Rules

- **Vibe Coding is accepted**, but code quality and correctness are the contributor's responsibility
- **Cross-validate AI-generated code** with multiple models when possible
- **Discuss large changes first**: major code churn, broad logic changes, migration strategy changes, data directory migration changes, and code signing changes should be discussed in GitHub Issues before implementation: https://github.com/wzh4869/AppPorts/issues
- **Never hardcode UI strings** — use i18n via `Localizable.xcstrings`
- **Never use `print`** — use `AppLogger.shared` for all logging
- **Always handle rollback** — migration operations must include rollback mechanisms on failure

---
> Source: [wzh4869/AppPorts](https://github.com/wzh4869/AppPorts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
