## tamaclaude

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A desk device that shows live Claude Code session status via a blocky orange Claude mascot.
Hardware: ESP32-2432S028R ("Cheap Yellow Display"), ILI9341 320x240 landscape, BLE only.

`DESIGN.md` (Thai) records the **current** design and the reasons that are not visible from
the code — not a changelog. **Read the relevant section before changing visuals, protocol,
or layout.** Write to it only when a previous decision is *reversed*, or when you hit a
constraint the code cannot state on its own (a measured hardware value, a framework quirk, a
rule that must hold in two places at once). Ordinary fixes need no entry — git history is
the log. When an entry stops being true, rewrite it in place; do not append the correction
below it.

## Data flow

```
Claude Code hooks --> tamaclaude --hook --> Unix socket --> daemon --> BLE GATT --> board

Claude Code statusline --> ~/.tamaclaude/statusline.sh --.
                                                          >--> ~/.claude/.statusline-usage-cache
menu bar timer --> tamaclaude --usage-poll --> claude.ai --'                |
                                                        daemon reads --> "u" key --> board
```

The quota panel has **two** sources, not one. The statusline pipe needs no credential but is
event-driven, so it goes quiet exactly when the desk display is left alone; the poll pipe uses
the user's `sessionKey` and keeps the number moving with Claude Code closed. Neither replaces
the other and they are separate switches — see the reversal note in `DESIGN.md`.

The daemon owns all logic. Firmware only knows a fixed `VisualState` enum and draws it.
Tool-to-animation mapping is host-side and user-overridable at `~/.tamaclaude/tools.json`.

## Commands

### Host (Swift, macOS 14+)

```bash
cd host
swift build                        # debug
swift run tamatest                 # run the whole test suite
swift run tamaclaude --daemon --print --no-ble -v   # daemon without bluetooth, prints snapshots
swift run tamaclaude --send '<json>'                # inject one hand-written hook event
swift run tamaclaude --usage-poll                   # one quota fetch -> cache, then exit
swift run tamaclaude --usage-cache < statusline.json  # the statusline pipe, by hand
swift run tamaclaude --install-statusline           # take over statusLine.command
swift run tamaclaude --remove-statusline            # give the slot back
./Scripts/make-app.sh              # release .app -> host/dist/TamaClaude.app
./Scripts/make-app.sh --install    # install to /Applications and launch
```

`--usage-poll` reads the key from `~/.tamaclaude/session-key` (mode 600, never argv, never env)
and exits: `0` = wrote the cache · `2` = key rejected · `3` = key file unusable · `1` = anything else.
The menu bar app runs it on a timer; the key is set from its gear menu, not by hand.

There is **no `testTarget`** and no per-test filter — `swift run tamatest` runs everything
(`Sources/tamatest/Tests.swift`, grouped by `suite("...")`). A machine with only Command Line
Tools would build a `testTarget` and exit 0 without running it, which is worse than no tests.
To narrow the run, temporarily comment out `suite(...)` calls in `runAllTests()`.

Bluetooth only works from the `.app` launched via LaunchServices (`open`) — a bare binary run
from a shell gets `SIGABRT` from TCC, not a polite denial. Every rebuild changes the adhoc
cdhash, so macOS re-asks for Bluetooth permission.

### Firmware (ESP-IDF v5.5)

```bash
cd firmware
idf.py -p /dev/cu.usbserial-XX flash monitor
```

`firmware/probe/` is a separate throwaway IDF project that interrogates the real panel
(MADCTL, colour order, inversion). Its findings are recorded in `DESIGN.md`; the firmware
uses those constants, not a chip model number.

### Graphics / preview (Python + Pillow)

```bash
python3 tools/preview.py            # render every state + whole screens to out/ (PNG + GIF)
python3 tools/preview.py --sheet    # contact sheet only
python3 tools/export_layout.py      # tools/layout.toml -> firmware/main/layout.h
python3 tools/make_icon.py          # logo PNG (≥128px) + mascot (≤64px) -> host/Resources/AppIcon.icns
```

There is no SDL2 simulator. `tools/preview.py` is the visual dev loop: change a rect, render,
look at `out/`. It proves the *design*, not the C renderer.

## Architecture

### One source of truth per concern

- **`tools/layout.toml`** — every layout constant, palette colour, and randomised table
  (star/grass/cloud positions). Python reads it via `tools/gen/config.py`; C gets it through
  the generated `firmware/main/layout.h`. **Never edit `layout.h`** — edit the TOML and rerun
  `export_layout.py`. If preview and board disagree, that is a renderer bug, by construction.
- **`tools/gen/*.py` ↔ `firmware/main/ct_*.c`** — deliberate parallel ports, file for file:
  `props.py`↔`ct_props.c`, `mascot.py`↔`ct_mascot.c`, `rects.py`↔`ct_rects.c`,
  `screen.py`↔`ct_ui.c`, `sky.py` folds into `ct_ui.c`. A visual change means editing both
  sides; the Python side is where you iterate, the C side is the port.
- **Assets are rect lists**, `{x, y, w, h, color}` in mascot-relative *unit* coordinates —
  no bitmaps, no sprite pipeline. The preview and the board both come from `gen/mascot.py`.
  The **app icon is half an exception** — `.icns` carries per-size art, so ≥128 px is a
  hand-drawn PNG (`docs/images/tamaclaude-logo.png`) and ≤64 px is drawn from the same rect
  list. See the reversal note in `DESIGN.md`.

### Host layout (`host/Sources/`)

| File | Role |
|---|---|
| `TamaCore/Protocol.swift` | `HookEvent`, `VisualState` (+ `priority`), `Snapshot`, MTU squeeze |
| `TamaCore/SessionStore.swift` | all the logic: hook → per-session state → snapshot |
| `TamaCore/ToolMap.swift` | tool name → `VisualState`, overridable via `~/.tamaclaude/tools.json` |
| `TamaCore/Text.swift` | strip to the board font's charset, then truncate |
| `TamaCore/SocketServer.swift` / `HookClient.swift` | Unix socket between `--hook` and the daemon |
| `TamaCore/BLETransport.swift` | CoreBluetooth central + auto-reconnect |
| `TamaCore/Usage{Reader,Writer}.swift` | the `.statusline-usage-cache` contract |
| `TamaCore/UsagePoll.swift` | `--usage-poll`: one claude.ai quota fetch, then exit |
| `TamaCore/UsagePoller.swift` | when to poll and what the last poll said — fed `tick(now:)`, owns no timer |
| `TamaCore/SessionStarter.swift` | when the app may open a session of its own — same shape, plus the guards and what locks it |
| `TamaCore/ChildOutput.swift` | what a child process said, drained off its pipe without blocking it |
| `TamaCore/SessionKeyFile.swift` | writes `~/.tamaclaude/session-key` so it is mode 600 from birth |
| `TamaCore/{Hook,Statusline}Installer.swift` | writes into `~/.claude/settings.json` |
| `TamaCore/Paths.swift` | the `~/.tamaclaude` paths + `Log` (`settings.json` belongs to `HookInstaller`) |
| `TamaCore/Daemon.swift` | wires it together + 1 s tick |
| `TamaCore/MenuBadge.swift` | what the menu bar icon knows: percent + pace position |
| `TamaCore/PanelText.swift` | what the foot of the popover says (board link, session rows, figure age) |
| `TamaCore/QuotaCard.swift` | what a quota card says: colour level, pace tick, reset line |
| `TamaCore/RefreshControl.swift` | the refresh button's discipline: cooldown, and when opening the panel polls |
| `tamaclaude/MenuBarApp.swift` | the menu bar app **is** the daemon (Bluetooth TCC is per-`.app`) |
| `tamaclaude/PanelViewController.swift` | the popover: header + gear, the cards, the foot |
| `tamaclaude/QuotaCardView.swift` | how a quota card is drawn (bar, pace tick, palette) |
| `tamaclaude/MenuBadgeImage.swift` | how the menu bar icon is drawn (template vs red) |

### Invariants worth knowing before you touch things

- **The daemon must fit one MTU (500 bytes).** `Snapshot.encoded` shrinks `n` (cards) — body,
  then title, then drop cards — and never touches `s` (sessions). Cards dropped for any reason
  (2-card cap or MTU) are counted into the separate `m` key.
- **Encode with `sortedKeys` always.** Otherwise the "did it change?" comparison is always
  true and the board gets written every second.
- **Every pose stays ≥ 5 s (`Timings.minPose`).** Poses skipped during that hold are dropped,
  never queued — a queue makes the mascot narrate an ever-later past.
- **`--hook` must always exit 0**, daemon running or not. A broken hook breaks the user's
  session, which is far worse than a frozen screen.
- **Unknown ≠ zero** in the usage panel: `-1` means unknown, `0` is a real value. Within one
  window (same `resets_at`) the percentage only increases, so a lower value is stale — never
  overwrite a newer one. Foreign keys in the shared cache file (`PROFILE_NAME`, `COST_*`)
  must survive.
- **The `sessionKey` is a full-account credential.** File only (`~/.tamaclaude/session-key`,
  mode 600), never argv, never env, never logged, re-read every poll. Not the Keychain: the
  adhoc signature changes cdhash on every build, so the item would prompt on every upgrade.
- **`-v` and `-psn_*` are not modes.** LaunchServices appends `-psn_0_12345`; an app that
  rejects unknown args dies on double-click.
- **The `VisualState` enum is a contract with the firmware.** Adding or reordering it means
  changing `ct_model.c`/`ct_mascot.c` too. `tamatest` guards this.

### GATT

```
service  7A9B0001-4C1E-4B6D-9E2A-1D5C3F0A0001
state    ...0002   daemon writes the snapshot here (write with response)
config   ...0003   brightness etc.
event    ...0004   board -> host, reserved for v2
```

## Conventions

- Comments and design docs are in **Thai**; identifiers, commands, and error strings stay in
  English. Existing comments explain *why*, often citing what was tried and rejected — match
  that density rather than describing what the code does.
- Commit subjects are lowercase imperative with a conventional prefix (`feat:`, `fix:`) and
  read as a sentence about the visible effect, e.g. `feat: give the sky half the screen and
  cap cards at two`.
- The board font has no em dash and no Thai glyphs — anything crossing BLE goes through
  `Text.swift` first.

## Agent skills

### Issue tracker

Issues live as GitHub issues in `thaitop/tamaclaude`, managed with the `gh` CLI. External PRs
are not a triage surface. See `docs/agents/issue-tracker.md`.

### Triage labels

The five canonical roles use their default label strings (`needs-triage`, `needs-info`,
`ready-for-agent`, `ready-for-human`, `wontfix`). See `docs/agents/triage-labels.md`.

### Domain docs

Single-context: `CONTEXT.md` + `docs/adr/` at the repo root. See `docs/agents/domain.md`.

---
> Source: [thaitop/tamaclaude](https://github.com/thaitop/tamaclaude) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-01 -->
