## bluetooth-keep-alive

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

macOS menu-bar app that keeps Bluetooth devices connected by periodically pinging them. Supports both BLE (CoreBluetooth) and classic paired devices (IOBluetooth). Built with SwiftUI + AppKit; persistence via GRDB/SQLite.

- Xcode project: `BluetoothKeepAlive/BluetoothKeepAlive.xcodeproj` (no SPM workspace at repo root — the `.vscode/launch.json` `swift run` configs are stale and don't work)
- Deployment target: macOS 15.5, Swift 5.0
- Bundle ID: `com.bluetooth-keep-alive`
- Only Swift package dependency: **GRDB** (resolved through Xcode SPM)

## Build & Run

Open `BluetoothKeepAlive/BluetoothKeepAlive.xcodeproj` in Xcode and ⌘R. Signing team must be set in target signing settings.

Command-line build:

```sh
xcodebuild -project BluetoothKeepAlive/BluetoothKeepAlive.xcodeproj \
  -scheme BluetoothKeepAlive -configuration Debug build
```

There is **no test target** — `xcodebuild test` will fail.

## Architecture

The directory `BluetoothKeepAlive/BluetoothKeepAlive/Souces/` is intentionally misspelled (do not rename — it would require pbxproj surgery).

Layered MVVM with a shared service locator:

- `Souces/Main.swift` — `@main` is a near-empty SwiftUI `App`. The real entry point is `AppDelegate` (`@NSApplicationDelegateAdaptor`), which:
  - calls `NSApp.setActivationPolicy(.accessory)` → no Dock icon, menu-bar only
  - builds the `NSStatusItem` menu and opens the Home / Settings windows as standalone `NSWindow`s hosting SwiftUI views via `NSHostingView`
  - applies the "start with system" preference at launch via `SMAppService.mainApp`
- `Souces/Core/Services/DIService.swift` — singleton (`DIService.shared`) that lazily owns every service. View models pull dependencies from here rather than through SwiftUI environment. Order of creation matters because services depend on each other (the lazy graph: `routineEventRepository` ← `routineStateStore` ← `timerRoutineService`; `pingerRegistry` is constructed eagerly inside its own lazy block and `RoutineStateStore` calls `startObserving` on every registered pinger in its init).
- `Souces/Data/Services/Interface/RepositoryService.swift` — generic base class. On first init it copies the bundled seed DB `Resources/RoutinesDatabase.sqlite3` into `~/Library/Application Support/<bundle-id>/RoutinesDatabase.sqlite3`, opens a GRDB `DatabaseQueue`, then runs `DatabaseMigrator` from `Souces/Data/Services/Migrations/DatabaseMigrations.swift`. **All repositories share this same SQLite file**. New tables go through a new migration step; never assume the seed has them.
- `RoutineRepository` / `SettingRepository` / `RoutineEventRepository` subclass `RepositoryService<T>` and override CRUD. They expose a Combine `PassthroughSubject<T, Never>` called `repositoryUpdated` that fires on every write (and, for `routineRepository.list()`, on each row read).
- `Souces/Core/Services/Pinger/` — `BluetoothDevicePinger` protocol abstracts the ping transport. `ClassicBluetoothPinger` is the only registered implementation today; it owns the `IOBluetoothDevice.register(forConnectNotifications:)` class-level observer and per-device disconnect observers (both retained as instance properties — they auto-unregister on dealloc). `PingerRegistry` keys pingers by `RoutineTransport`. To add BLE: implement the protocol and register the new pinger in `DIService.pingerRegistry` — no other code changes.
- `Souces/Core/Services/RoutineStateStore.swift` — single source of truth for runtime routine state (`.disabled` / `.dormant` / `.active`). Subscribes to `routineRepository.repositoryUpdated` to rebuild the address set the pinger filters against; receives connection callbacks from the pinger and publishes state via `@Published states` and a `transitions: PassthroughSubject<(String, RoutineRuntimeState), Never>`.
- `Souces/Core/Services/TimerRoutineService.swift` — owns one repeating `Timer` per enabled routine. **Each tick reads from `RoutineStateStore` and `SnoozeService` before pinging**: snooze active → log `.snoozeSkip` once and return; state `.dormant` → log `.dormantSkip` once and return; state `.active` → call `pinger.ping(...)` and log `.pingOk` / `.pingFailed`. It also subscribes to `stateStore.transitions` so that a `.dormant → .active` flip fires an immediate ping (don't wait for the next tick). To change a routine's behaviour, **write through the repository**; never poke the timer service directly.
- `Souces/Core/Services/SnoozeService.swift` — global pause. Persists the deadline in the `settings` table under id `"snooze-until"` (ISO timestamp in the `name` column). On launch it rehydrates and schedules a `DispatchSourceTimer` to clear itself at the deadline. `AppDelegate` builds a "Pause routines" submenu in the status item; the `NSMenuDelegate` updates the labels each time the menu opens.
- `Souces/Data/Models/RoutineEvent.swift` + `RoutineEventRepository` — per-routine event log (`pingOk`, `pingFailed`, `connected`, `disconnected`, `dormantSkip`, `snoozeSkip`). Driven from `TimerRoutineService` (ticks) and `RoutineStateStore` (connection callbacks). Retention is enforced by `RoutineEventRepository.prune(retainPerRoutine:)`, called once per launch from `AppDelegate.bootRoutineEngine`.
- `Souces/Core/Models/BluetoothModel.swift` — unifies BLE and Classic devices. `BTType` is an enum with associated values (`.ble(CBPeripheral)` / `.classic(IOBluetoothDevice)`); pattern-match when behaviour diverges by transport.
- `Souces/Ui/{Home,Device,Settings}/` — each screen has a `View` + `ViewModel` pair. `HomeViewModel` injects `RoutineStateStore` and mirrors `states` into a `@Published var routineStates` for the row badges (`RoutineStatusBadge`). `DeviceViewModel` reads `lastSuccess` and `disconnectsSince(sessionStart:)` from `RoutineEventRepository` for the stats panel and subscribes to `stateStore.transitions` to refresh on connection changes.

### Launch-at-boot is load-bearing

`AppDelegate.applicationDidFinishLaunching` calls `bootRoutineEngine()`, which touches `DIService.shared.timerRoutineService`. Without this touch, the service graph would only spin up when the user opens the Home window — meaning at headless boot (via `SMAppService.mainApp`), nothing would start. Don't remove that line.

## Conventions specific to this codebase

- `Bool` ↔ `Int` conversion uses the extensions in `Souces/Core/Extensions/` (`isEnabled.boolean`, `bool.integer`). The DB stores enabled flags as `Int` (0/1).
- The Xcode group `BluetoothKeepAlive` uses `PBXFileSystemSynchronizedRootGroup` — files added under `Souces/` are picked up automatically by the target; no `project.pbxproj` edit needed for new `.swift` files.
- Errors are surfaced through `ErrorHelpers` factory methods (e.g. `ErrorHelpers.updateFailed(reason:)`) and shown via `InfoAlert` / `showError(...)`. View-model methods that can fail are typically `throws` and the calling view presents the alert.
- `Settings` rows are keyed by string id. Currently used keys: `"start-with-system"` (mirrors `SMAppService.mainApp` registration) and `"snooze-until"` (ISO timestamp in `name` column, `isEnabled` flag toggles the row).
- The app is sandboxed (`com.apple.security.app-sandbox`) and uses IOBluetooth — both `.entitlements` files must keep `com.apple.security.device.bluetooth=true`. Without that entitlement, `IOBluetoothDevice.register(forConnectNotifications:)` silently returns no callbacks in release builds.
- MAC addresses are normalized to uppercase in `RoutineStateStore.normalizedToRoutineId` because `IOBluetoothDevice.addressString` casing isn't stable. The dict maps `normalized → canonical routine.id` so the rest of the code can keep using whatever case is stored in the DB.

---
> Source: [alan-pinho/bluetooth-keep-alive](https://github.com/alan-pinho/bluetooth-keep-alive) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
