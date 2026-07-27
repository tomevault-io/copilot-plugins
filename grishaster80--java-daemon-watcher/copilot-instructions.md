## java-daemon-watcher

> macOS app (SwiftUI) that detects and kills idle Gradle/Kotlin daemons spawned by Android development tools and AI agents. No sandbox — uses `jps`, `ps`, and POSIX `kill()` directly.

# CLAUDE.md — Java Daemon Watcher

## What is this project?

macOS app (SwiftUI) that detects and kills idle Gradle/Kotlin daemons spawned by Android development tools and AI agents. No sandbox — uses `jps`, `ps`, and POSIX `kill()` directly.

## Build & Run

```bash
# Build
cd JavaDaemonWatcher
xcodebuild -scheme JavaDaemonWatcher -configuration Debug build

# Install to /Applications (required for Spotlight indexing)
cp -R ~/Library/Developer/Xcode/DerivedData/JavaDaemonWatcher-*/Build/Products/Debug/JavaDaemonWatcher.app /Applications/

# Run
open /Applications/JavaDaemonWatcher.app

# Or just open in Xcode
open JavaDaemonWatcher.xcodeproj
```

Requires: macOS 14.0+, Xcode 15+, JDK installed (for `jps` binary).

### Release Build

```bash
cd JavaDaemonWatcher
xcodebuild -scheme JavaDaemonWatcher -configuration Release -archivePath build/JavaDaemonWatcher.xcarchive archive
xcodebuild -exportArchive -archivePath build/JavaDaemonWatcher.xcarchive -exportPath build/ -exportOptionsPlist ExportOptions.plist
```

## Project Structure

```
JavaDaemonWatcher/JavaDaemonWatcher/
├── App/JavaDaemonWatcherApp.swift          # @main entry, WindowGroup, AppDelegate
├── Assets.xcassets/                        # App icon (generated, all macOS sizes)
├── Models/
│   ├── DaemonType.swift                    # .gradle / .kotlin enum
│   ├── JavaDaemon.swift                    # Daemon data model + CPU ring buffer
│   └── Statistics.swift                    # KillRecord + Statistics structs
├── Services/
│   ├── JpsLocator.swift                    # Finds jps: JAVA_HOME → SDKMAN → java_home → PATH
│   ├── DaemonDetector.swift                # jps -lv → parse → allowlist filter → ps stats
│   ├── ProcessMonitor.swift                # CPU sampling → idle detection (6 samples < 1%)
│   └── DaemonKiller.swift                  # SIGTERM → 5s wait → SIGKILL
├── Persistence/
│   ├── StatisticsStore.swift               # actor, JSON in ~/Library/Application Support/
│   └── SettingsStore.swift                 # @Observable, UserDefaults, SMAppService
├── ViewModels/
│   └── DaemonWatcherViewModel.swift        # @Observable @MainActor, scan loop, orchestration
├── Views/
│   ├── MenuBarView.swift                   # Main window: daemon list, inline stats, inline settings
│   ├── DaemonRowView.swift                 # Daemon row with status dot and kill button
│   ├── StatisticsView.swift                # Today + all-time kill stats (standalone)
│   └── SettingsView.swift                  # Inline settings with back navigation
└── Utilities/
    ├── ShellExecutor.swift                 # async Process() wrapper with timeout
    └── Formatters.swift                    # Memory (MB/GB) and duration formatting
```

## Architecture

- **UI:** SwiftUI WindowGroup (regular macOS window)
- **State:** Swift Observation (`@Observable`), strict concurrency (`Sendable`, `actor`)
- **Concurrency:** structured concurrency, `Task`, `TaskGroup`, `actor`
- **Detection:** `jps -lv` for daemon discovery, `ps -o pid=,rss=,%cpu=` for resource stats
- **Kill flow:** `kill(pid, 0)` pre-check → SIGTERM → poll 5s → SIGKILL fallback
- **Persistence:** JSON file for kill stats, UserDefaults for settings

## Key Design Decisions

1. **Allowlist-only kill policy.** Only GradleDaemon and KotlinCompileDaemon class names pass `DaemonDetector.classifyDaemon()`. Everything else (Android Studio, IntelliJ, etc.) is ignored.
2. **No sandbox.** App needs `kill()`, `jps`, `ps` — cannot run sandboxed.
3. **Regular window app.** Standard macOS window with dock icon, visible in Spotlight.
4. **Idle = all 6 CPU samples < 1%.** Conservative: any activity resets idle timer.
5. **Stats pruned to 90 days.** `StatisticsStore` removes old records on every write.
6. **Inline settings.** Settings view slides in within the same window — no modals or secondary windows.

## Settings (UserDefaults keys)

| Key | Type | Default | Description |
|---|---|---|---|
| `autoKillEnabled` | Bool | `true` | Auto-kill idle daemons |
| `idleThresholdMinutes` | Int | `10` | Minutes idle before auto-clear |
| `scanIntervalSeconds` | Int | `10` | Scan loop interval |
| `launchAtLogin` | Bool | `false` | SMAppService registration |

## Data Files

- Stats: `~/Library/Application Support/JavaDaemonWatcher/statistics.json`
- Settings: standard UserDefaults (`com.javadaemonwatcher.app`)

## Common Tasks

- **Add a new daemon type:** add pattern to `DaemonDetector.classifyDaemon()`, add case to `DaemonType` enum
- **Change idle detection logic:** edit `ProcessMonitor.updateIdleState()` and `JavaDaemon.maxSamples`/`cpuIdleThreshold`
- **Change kill behavior:** edit `DaemonKiller.kill()` (timeouts, signal sequence)
- **Add a new setting:** add property to `SettingsStore`, add UI to `SettingsView`

## Code Style

- Stateless services are `enum` with `static` methods (not classes)
- Models are `struct` + `Sendable`
- Thread-safe persistence via `actor` (`StatisticsStore`)
- UI state is `@Observable` + `@MainActor` (`DaemonWatcherViewModel`)
- No third-party dependencies

---
> Source: [grishaster80/java-daemon-watcher](https://github.com/grishaster80/java-daemon-watcher) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
