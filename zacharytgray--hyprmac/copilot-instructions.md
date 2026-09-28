## hyprmac

> Tiling window manager for macOS, inspired by Hyprland. It is an unsandboxed Swift/SwiftUI menu

# HyprMac

Tiling window manager for macOS, inspired by Hyprland. It is an unsandboxed Swift/SwiftUI menu
bar app built on Accessibility plus private CGS/SkyLight APIs, so SIP stays on. The Hypr key is
Caps Lock by default (remapped to F18 through `hidutil`). It has BSP dwindle tiling, 10
workspaces with home monitors, a floating scratchpad layer, and multi-monitor support.

## Where work runs

- All repository work runs on the Mac mini hub (`zacharys-mac-mini`): edits, tests, builds,
  releases.
- The MacBook (`zachbook-pro`) only runs the signed Debug app for live-behavior tests. Build on
  the hub and copy the app over.

## Hard rules

- `HyprMac.xcodeproj` is generated from `project.yml`. Run `xcodegen generate` after adding or
  removing files. Never hand-edit `project.pbxproj`. On a PR, regenerate and compare; stale
  hand-edited copies have dropped tests before.
- `TilingEngine` owns the BSP trees. Nothing else mutates a tree.
- `Action` JSON wire keys are frozen, including legacy ones like `switchDesktop`. Never rename
  one. Config decoding drops only the bad keybind. Schema changes need a `ConfigMigration` step
  and `KeybindDecoderToleranceTests` updates.
- Ghost windows (closed while the app keeps running) keep their workspace. User-facing
  emptiness (menu bar, Hypr+F) ignores all hidden windows. Admission and move capacity ignore
  only hidden minus reserved. Suspect ghosts first on any occupancy bug.
- With Caps Lock as the Hypr key, macOS Modifier Keys settings must leave Caps Lock as
  "⇪ Caps Lock". Never advise "No Action"; it makes the key invisible to HyprMac.
- Keep the red shake error feedback (`FocusBorder.flashError`).
- Never launch through the Xcode debugger. Accessibility is not granted there.
- Intermittent bugs: no repro, no guess fix. Add `.notice` logs and confirm the cause first.
- Debug and release share `~/Library/Application Support/HyprMac/`. Back it up before any
  migration test.

## How to verify

```bash
git diff --check
plutil -lint HyprMac.xcodeproj/project.pbxproj
for s in scripts/*.sh; do bash -n "$s"; done
bash scripts/test-release-pipeline.sh
./scripts/test-isolated.sh --debug-variant                # full suite, ~5 min
./scripts/test-isolated.sh --debug-variant BSPTreeTests   # one class
```

The class argument must be a bare class name. A `HyprMacTests/` prefix runs 0 tests and exits
0, so check that `Executed N tests` is nonzero. Other traps, the Release universal build, and
the analyzer are in `docs/testing.md`. `swiftlint` is not installed; do not claim a lint pass.
Tests do not prove window behavior; say so if you did not check it live.

`xcodegen` and `gh` are in `/usr/local/bin` on the hub.

## Doc map

- `docs/testing.md`: test harness traps, headless tests, Release build, analyzer.
- `docs/laptop-debug-deployment.md`: `build-debug.sh` and the MacBook redeploy recipe.
- `docs/debugging.md`: log subsystems, `/usr/bin/log`, file log, debug recipes.
- `docs/release.md`: What's New step, `scripts/release.sh`, failure recovery.
- `docs/architecture.md`: component graph, ownership, threading, module layout.
- `docs/tiling-algorithm.md`: dwindle, smart insert, layout passes, drag classification.
- `docs/keybinds-and-actions.md`: Action wire format, frozen keys, decoder tolerance.
- `docs/coordinate-systems.md`: NSScreen vs CG origins. Read it before frame math.
- Other `docs/*.md` and `plans/`: dated audits and plans. Their facts go stale.

Top-level layout: `HyprMac/` app source, `HyprMacTests/` unit tests, `scripts/` build, test and
release scripts, `Casks/` Homebrew cask, `tools/` standalone SkyLight probe.

---
> Source: [zacharytgray/HyprMac](https://github.com/zacharytgray/HyprMac) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-27 -->
