## miniwatts

> Sideload-only iPhone battery instrument. Private APIs; never App Store safe.

# MiniWatts

Sideload-only iPhone battery instrument. Private APIs; never App Store safe.

The method for reading the PMU — dlsym'd IOKit, `IOHIDEventSystemClient` on usage
pages `0xff08` and `0xff00` — and the first version of the *Verified blocked* list
below are taken from
[ios-charging-monitor](https://github.com/gregsramblings/ios-charging-monitor)
(ChargeSpeed, MIT). What is built on that is this project's own: the five screens,
the thermal zone mapping, the USB-PD inspector, the energy integration and session
history, and everything under *Sensor notes*, which the blocked list has since grown
by several entries.

## Build and run

- Plain Xcode project, no XcodeGen, no packages. `PBXFileSystemSynchronizedRootGroup`:
  new files under `MiniWatts/` join the target automatically, do not edit the pbxproj.
- iOS 17 deployment target, **Swift 6** language mode, Xcode 26+ required
  (`nonisolated` on type and extension declarations is Swift 6.2).
- `./scripts/build-ipa.sh` — unsigned ipa, the distributable one; runs
  `verify-clean.sh` on itself and fails if the artifact carries identifying data.
- `TEAM_ID=… ./scripts/build-ipa.sh signed` — for your own device. No default team
  lives in this repo; `DEVELOPMENT_TEAM` is empty in the pbxproj and Xcode will
  write yours back into it if you pick one in the UI. Do not commit that.
- `.github/workflows/build.yml` builds, verifies and (on a `v*` tag) releases.
- The simulator reads the **Mac's** battery through IOKit and has no HID sensors:
  fine for layout and for the adapter/PD panels, useless for anything sensor-driven.

## Layout

- `Core/Sensors/` — the probes. `IOKitBattery` (dlsym'd IOKit + powerd), `HIDSensors`
  (`IOHIDEventSystemClient`, one client per process, created once), `BatteryCenterBridge`,
  `ThermalMonitor` (`ProcessInfo.thermalState`, public API), `SensorCatalog`
  (name → zone/label by whole-word keyword, never exact).
- `Core/Model/` — `PowerSnapshot` merges all four sources and owns every derived value.
  Anything derived from the HID readings is resolved **once, in `init`, and stored** —
  it used to be computed per access, which meant a body asking for the battery
  temperature four times did four linear scans and called `SensorCatalog.zone(for:)`
  (which lowercases and splits) once per sensor per scan. Registry lookups stay
  computed: those are single hash hits. Also here:
  `EnergyAccumulator` integrates ∫V·I dt; `ChargeSession` + `SessionStore` persist charges
  as one JSON file in Application Support.
- `Core/PowerMonitor.swift` — `@Observable`, 1 s tick, drives everything and owns session
  lifecycle. Injected once in `MiniWattsApp`, read via `@Environment(PowerMonitor.self)`.
  `headline` lives here rather than on the snapshot: its last fallback is the %-rate
  estimate, which is derived across several snapshots and so is not a snapshot's to give.
- `Design/` — palette (`Color.mw(light:dark:)`, no asset catalog entries), `Panel`/
  `Metric`/`Pill`/`BarRow`, `PowerRing`, Swift Charts wrappers, `PhoneHeatMap`.
- `Features/` — one folder per tab, plus Settings. `DebugView` (Raw data) is
  `#if DEBUG` only and reached from the bottom of Settings, not the main toolbar.

## Swift 6 isolation

The project sets `SWIFT_DEFAULT_ACTOR_ISOLATION = MainActor`, so everything is
main-actor isolated unless it says otherwise. Consequences that have already bitten:

- **`Theme.swift` must stay `nonisolated`.** `Color.mw` builds a `UIColor` with a
  trait-resolution closure, and UIKit calls that from whatever thread is resolving a
  dynamic colour while rendering. Under Swift 6 the compiler inserts an executor
  check there, and the app **traps on the first frame that paints a gradient**. It
  compiles fine either way — this is a runtime crash, not a build error.
- The whole `Core/` layer is `nonisolated`: it has no UI in it, and nonisolated
  protocol requirements (`Shape`, `Layout`, `Identifiable`, `Codable`) cannot be
  witnessed by main-actor-isolated members. `PowerMonitor` and `ThermalMonitor`
  stay on the main actor — they are `@Observable` UI state.
- `deinit` is nonisolated in Swift 6 and cannot touch isolated stored properties.
  `ThermalMonitor` therefore has no notification observers to tear down; it is
  polled from the one-second tick instead, which costs nothing and covers the same
  ground (nothing observes a change made while the app is suspended anyway).

## Sessions and the app lifecycle

A charge session ends when the **charger is unplugged**, not when the app leaves the
foreground. Three things make that work and they are easy to undo by accident:

- `RootView` calls `monitor.pause()` on `.background` only. It used to call a `stop()`
  that closed the session on anything that was not `.active`, and `.inactive` fires for
  a pulled-down Control Center, the app switcher, an incoming call and the screen
  locking — so an overnight charge was recorded as a scatter of two-minute fragments.
- `closeSessionIfNeeded` dates the end from `lastConnectedObservation`, the last tick
  that actually saw a charger, not from `.now`. Unplug while the app is suspended and
  the first tick after it wakes is the first that knows; `.now` there would stretch the
  session across however long the app was away.
- Settings has *keep the screen on while charging*, default on, applied in `RootView`
  (`isIdleTimerDisabled`) and gated on the phone being plugged in. Sensors can only be
  read in the foreground, so without it the screen locks and a full charge can never be
  recorded. UIKit stays in the view layer; `Core` only holds the preference.

`SessionStore` encodes and writes on its own serial queue, coalescing bursts, and the
load in `PowerMonitor.init` is a `Task`. At the ceiling — 60 sessions × 1,500 samples —
the file is several megabytes, and doing that inline was a stall before the first frame
and again every thirty seconds during a charge. `persist()` is a no-op until `isLoaded`,
or a save landing before the load would truncate the history to nothing.

## Localization

UI copy is in a String Catalog. Slots are typed by what they actually carry:

| Kind | Type | Examples |
| --- | --- | --- |
| Always copy | `LocalizedStringResource` | `Panel.title`, `Metric.caption`, `EmptyNote.text`, `DetailRow.label` |
| Always a measurement | `String` | `Metric.value`/`unit`, `BarRow.detail` |
| Either, decided per call | `Text` | `Panel.trailing`, `Pill.text`, `BarRow.title`/`subtitle`, `Metric.footnote` |

**`Text(someString)` is the non-localising overload.** Passing a `String` variable
where copy was meant does not just skip translation at runtime — the extractor has
nothing to extract either, so the string never reaches the catalog and nothing ever
flags it. That is how 49 user-visible strings were silently English-only after the
project had supposedly been localised: every `DetailRow` label, the five sentences of
`AdapterView.headroomReason`, the live-rail names, the session fallback titles, and
the Left/Right/Case/Internal/Wired/Wireless strings that `BatteryCenterBridge` used to
build in English inside the model. If a slot carries copy, type it
`LocalizedStringResource` or `Text` — never `String`. `DetailRow` has two initialisers
for exactly this: `label:` for copy, `rawLabel:` for an IOKit key.

`Text("…")` is extracted, `Text(verbatim:)` is not. The mixed slots exist because a
panel header reads `12 sensors` on one screen and `iPhone17,2` on the next, and a
hardware name must never reach the catalog as a lookup key. `SensorCatalog.label(for:)`
returns `LocalizedStringResource?` — nil means "show the raw sensor name".

Watch for: `String(format:)` still hardcodes the decimal separator, and
`Formatting.duration` hardcodes h/m/s. `Formatting.timestamp`/`clock` were moved to
`Date.FormatStyle` and do follow the locale.

## Verified blocked (don't retry)

**Sandbox, iOS 26/27.** `IOPMPowerSource` registry keys beyond `BatteryInstalled`/
`ExternalConnected`; `IOReport`; `IOPMCopyBatteryInfo`; `IOPSCopyChargeStatus`,
`IOPSCopyBatteryLevelLimits`, `IOPSCopyPowerSourcesInfoPrecise` (all
`kIOReturnNotPrivileged`); PowerUI XPC (Optimized Charging, charge limit); powerd
`Time to Empty` (always 0).

**Accessory battery levels are gone.** `BatteryCenter.framework` still loads from a
normal sandbox and `BCBatteryDeviceController` still exists, but:

- `+sharedInstance` **no longer exists** on iOS 26+ — the selector is not even in the
  framework binary. The controller is allocated with `+new` now, and
  `-addBatteryDeviceObserver:queue:` (protocol `BCBatteryDeviceObserving`, callback
  `connectedDevicesDidChange:`) is what starts collection.
- Even then `connectedDevices` returns an empty array, while the system Batteries
  widget shows the same devices. The console says it plainly:
  `(<_BCPowerSourceController: …>) Failed to obtain power sources info`. The
  controller's XPC to powerd is denied; it fails quietly and returns nothing.
- There is no public API either. Apple Watch is reachable only by shipping a
  watchOS companion app and sending `WKInterfaceDevice.batteryLevel` over
  `WatchConnectivity`. AirPods have no route at all — CoreBluetooth's standard
  Battery Service is not exposed by them, and ExternalAccessory is MFi-only.

The Devices tab therefore shows this iPhone and a placeholder. The card-rendering
code is still there and lights up if BatteryCenter ever answers again.

**No wireless input current.** Enumerating all 75 HID services on an iPhone 17 finds
eleven sensors on usage page `0xff08` and no `IQ1u` — the charge IC exposes the coil
voltage (`Charger VQ1u`) and nothing to multiply it by. Wireless input power cannot
be measured. The dial falls back to the battery-side figure and says "into battery".

**The adapter's voltage and current are a ceiling, not a reading.** Two MagSafe
samples minutes apart both reported `AdapterVoltage 6800` × `Current 661` while the
current into the cell fell from 0.76 A to 0.51 A; over USB-C the same pair equals the
`UsbHvcMenu` profile's `MaxVoltage` × `MaxCurrent` exactly (5 V × 3 A) while the phone
drew 3.9 W. Use it for the rating, never for live draw.

**No discharge-current sensor exists.** Discharge power comes from the %-rate estimate
against a pack energy the user sets in Settings.

## Sensor notes (iPhone 17 / iPhone18,4)

- `PMU tcal` reads exactly 51.8 °C in every sample while its neighbours move several
  degrees. It is a calibration constant. Excluded from anything that ranks sensors by
  heat, still listed in its zone — and that exclusion has to happen at the **grouping**,
  not only at `hottestSensor`. `temperaturesByZone` used to sort a zone's readings by
  value and hand out `.first`, which made `tcal` the SoC zone's representative: the heat
  map showed a permanently red 52° SoC pin while the "Hottest" readout an inch below it,
  which did exclude `tcal`, said 44°. `ZoneTemperatures.hottest` is the live-only
  maximum; `readings` keeps everything, with constants sorted last.
- Four separate sensors are all named `gas gauge battery`. `HIDSensors.Reading.id`
  therefore includes the service index — name alone gave `ForEach` duplicate ids.
- `Charger QQ0u` (usage 2) and `Charger WQ0u` (usage 3) are **unidentified**. They sit
  on the USB-C port, so they cannot be the wireless input; and `WQ0u` is not
  instantaneous power (it read 0.726 while `VQ0u × IQ0u` was 3.91 W). `ALS` has the
  same `Q`/`W` pair, so the letters are a general convention, not charger-specific.
  They stay out of every derived value.
- Enumerating **all** HID services needs `IOHIDEventSystemClientSetMatching(client, NULL)`.
  An empty matching dictionary matches nothing, which made the debug button look dead.
- `PMU tdie14`–`tdie17` appear in the service list but return NaN; they are skipped.

## Distribution

Releases ship an **unsigned** ipa. A signed one carries a provisioning profile, and
that profile contains the team ID, every developer certificate and **the UDID of
every registered device** — five of them, in the build checked. Never publish one.

Two things leak build paths into the binary and need two different fixes:

1. Swift writes absolute source paths through debug info and `#file` metadata →
   `-file-prefix-map $PWD=/MiniWatts` (and `-ffile-prefix-map` for C).
2. The linker records every object file's absolute path in the symbol table as
   `N_OSO` debug-map entries, in `__LINKEDIT` → `xcrun strip -S -x`.

`scripts/verify-clean.sh` checks all of it. Verify with a raw byte scan
(`grep -a "/Users/"`, or `nm -pa … | grep " OSO "`), **not** with `strings` and **not**
with Hopper: the paths live in `__LINKEDIT`, which Hopper's Strings view does not show,
so it reports clean while they are still there.

Also note that Xcode 26+ Debug builds put the app's code in `MiniWatts.debug.dylib`
and leave a Previews launcher stub as the main executable — inspecting the wrong file
makes every symbol look absent.

## Performance

An Instruments trace (Core Animation + Time Profiler, Release, device) says the
probes are not the cost:

- IOKit — 45 HID sensors plus the registry, every second — is **0.6 % of CPU**.
- SwiftUI/UIKit/QuartzCore rendering is ~31 %.
- Core Animation commits run at **66/s** for data that changes once a second, and
  take 7.9 % of wall time. Something is always animating.

`Color.mw` builds a **new** `UIColor` with a trait-resolution closure on every call,
and a dynamic `UIColor` compares by identity, so a `Color` produced by calling it in a
view body was a different value every evaluation. `Color.mwTemperature` did exactly
that, which is why `Backdrop`'s `.animation(_:value: glow)` restarted an 0.8 s
full-screen `plusLighter` animation every second on the Thermal tab for a temperature
that had not moved. Every palette entry is now a `static let`; keep it that way.

There is **no `Info.plist` in the source tree** and there should not be one: the
bundle is built entirely from `GENERATE_INFOPLIST_FILE` plus the `INFOPLIST_KEY_*`
build settings. The file used to exist for a single key,
`CADisableMinimumFrameDurationOnPhone`, which opts the app into 120 Hz for data that
changes once a second; with that gone the file held nothing, and Xcode dropped both it
and the `INFOPLIST_FILE` setting on the next build. Do not re-add it — put new keys in
`INFOPLIST_KEY_*` instead.

`PageScaffold` uses a `LazyVStack`. History puts up to sixty session panels through it.

Already removed: `contentTransition(.numericText())` on every readout (it is opt-in
via `mwReadout(rolling:)` now, used only by the dial), and the heat map's 0.6 s
animation, which was retriggered every second on a `blur` + `plusLighter` layer.
The grid `Canvas` is `.drawingGroup()`-rasterised.

Deliberately kept despite the cost, as design decisions: the `Backdrop`'s full-screen
`plusLighter` glow, and `PowerRing`'s `.shadow` on a stroked arc (a non-rectangular
shadow is an offscreen pass per frame).

## Conventions

- English source copy. Sensor names stay raw (`Charger VQ0u`), with a human label
  above them when `SensorCatalog` recognises one.
- Follow the system appearance — every colour is defined for both schemes in
  `Theme.swift`. No `colorScheme` checks in view bodies.
- A missing reading renders as a muted dash or the words "no reading", never as 0.
- Nothing is claimed that the sandbox did not actually return. Panels say where their
  numbers come from and mark inferred values as inferred — charging holds are inferred
  from behaviour, and labelled that way.
- User-facing copy stays in the app's voice; selector-level detail belongs in Raw data.

---
> Source: [ResistanceTo/MiniWatts](https://github.com/ResistanceTo/MiniWatts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-10 -->
