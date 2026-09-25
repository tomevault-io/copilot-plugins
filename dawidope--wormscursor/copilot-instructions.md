## wormscursor

> A Windows tray app that themes the system cursors into a *Worms 3D*-style set: the arrow

# WormsCursor

A Windows tray app that themes the system cursors into a *Worms 3D*-style set: the arrow
and hand **rotate to follow mouse-movement direction**, and the rest of the standard
cursors get matching animated treatments (springy pendulums, jelly wobble, taffy stretch).
It ships as a Velopack app (Setup.exe / Portable.zip) that auto-updates from GitHub
Releases. No main window — everything lives in the system tray.

## How it works (the core idea)

Theming is done globally via the Win32 `SetSystemCursor` API. That's a **persistent,
system-wide** change, so the engine is obsessive about restoring the user's real cursors
on exit / crash / re-launch (`SetSystemCursor` never touches the registry, so
`SystemParametersInfo(SPI_SETCURSORS)` always reloads the genuine scheme). The arrow/hand
are pre-rendered as N rotated frames and swapped per movement direction; the animated
cursors (busy, help, crosshair, I-beam, resize, move, unavailable) are re-rendered each
frame **only while actually on screen** (detected via `GetCursorInfo`) so an idle tray
burns no CPU.

## Projects (`src/`)

### `WormsCursor.Core` — the engine (UI-agnostic, GDI+ only, **no WinForms/WPF**)
The reusable heart. Hostable from a tray app, console, or tests.
- `CursorEngine.cs` — the background tracking + animation loop. Owns all `SetSystemCursor`
  calls, the per-direction frame swap, the on-screen animation re-render, and restore-on-
  teardown. Defines the `TestCursor` enum (the set of themed cursor kinds + `Off`).
- `ArrowRenderer.cs` / `HandRenderer.cs` / `HandShape.cs` — draw the arrow and hand
  silhouettes (fill + pen outline) at a given size/colour/thickness. `HandShape` is the
  baked geometry (its source SVG/bake script was removed; recover from git history).
- `ProgressRenderer.cs` — composes every *animated* cursor (busy/app-starting/help/
  crosshair/I-beam/resize/move/unavailable) and exposes `RenderRest(...)` for static
  "at rest" previews. Also holds shared layout maths (`Layout`).
- `CursorSettings.cs` — the persisted, tunable settings (size, colours, outline thickness,
  corner radius, engine tuning, and `DisabledCursors` = which cursors are left as the
  Windows default). `Clone`/`CopyFrom` deep-copy the list so a dialog's working copy never
  aliases the live settings.
- `SettingsStore.cs` — tolerant JSON load/atomic save in
  `%LocalAppData%\WormsCursor\settings.json` (survives app-folder updates).

### `WormsCursor.App` — the tray app (WinForms, `net8.0-windows`, AssemblyName `WormsCursor`)
- `Program.cs` — entry point. `VelopackApp.Build().Run()` **first** (handles install/update
  hooks), then a single-instance guard, then the tray context.
- `TrayApplicationContext.cs` — owns the tray icon, the `CursorEngine`, autostart + update
  menu items, and the restore-on-exit/crash safety nets. `ApplySettings` saves and
  stops+restarts the engine so edits take effect live.
- `PreferencesForm.cs` — the live appearance editor: a real-size cursor preview grid (7×2),
  size/colour/outline/radius controls, hover-to-try, a per-tile on/off checkbox, a
  *Test cursor* combo, and **Showtime** (a hands-free demo that cycles the enabled cursors
  for recording).
- `Services/UpdateService.cs` — Velopack `UpdateManager` against the GitHub repo; no-ops
  cleanly for dev builds run from `bin\`.
- `SingleInstance.cs`, `Autostart.cs` (per-user `HKCU\…\Run`), `Assets/` (tray icon).

### `WormsCursor.Preview` — the docs/showcase generator (console)
Renders a labelled sheet of every themed cursor (a dark card + a transparent variant) to
PNG for the README/docs. `dotnet run --project src/WormsCursor.Preview -- [outPath] [size]`.

## Build & run

```bash
dotnet build WormsCursor.sln -c Release      # build everything
dotnet run --project src/WormsCursor.App     # run the tray app (dev)
```

.NET 8 SDK, pinned by `global.json`. The app is `net8.0-windows`, PerMonitorV2 DPI.

## Releasing

Pushing a `vX.Y.Z` tag triggers `.github/workflows/release.yml` (Velopack pack → GitHub
Release). Use the **`release` skill** (`.claude/skills/release/`) for the step-by-step
(version bump + CHANGELOG roll + tag + push). `CHANGELOG.md` follows Keep a Changelog with
one section per tag; the release notes are extracted from the matching section. Auto-update
wiring lives in `Services/UpdateService.cs`.

## Conventions & notes

- **Core stays UI-free** — only `System.Drawing`. Keep WinForms out of it.
- Several patterns (settings store, Velopack packaging, delta-on-release) deliberately
  **mirror the sibling PowerLink project**; comments call this out.
- Settings live in `%LocalAppData%\WormsCursor\` (not next to the exe) so updates don't
  wipe them. JSON load is tolerant; additive fields don't need a schema bump.
- `tools/`: `generate-icon.py` (tray/app icon) and `RestoreCursor.ps1` (manual
  cursor-restore fallback).
- Known issue: animated cursors can flicker on mixed-DPI multi-monitor setups (see the
  CHANGELOG); a proper fix needs an owner-drawn overlay instead of `SetSystemCursor`.

---
> Source: [dawidope/WormsCursor](https://github.com/dawidope/WormsCursor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-25 -->
