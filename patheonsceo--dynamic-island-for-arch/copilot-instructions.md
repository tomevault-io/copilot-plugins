## dynamic-island-for-arch

> **OpenAgentIsland** is an open-source macOS-style **Dynamic Island desktop** for Hyprland, built in **Quickshell/QML** on top of the end-4 (illogical-impulse) framework. The look: **three independent FLOATING islands, no bar** — wallpaper breathes through the gaps. The centerpiece is a morphing notch: minimal clock when idle, expanding for volume, media (with audio visualizer), and — the novel, headline feature — **live Claude Code agent status with permission approval directly from the notch**.

# CLAUDE.md — OpenAgentIsland

## What this is
**OpenAgentIsland** is an open-source macOS-style **Dynamic Island desktop** for Hyprland, built in **Quickshell/QML** on top of the end-4 (illogical-impulse) framework. The look: **three independent FLOATING islands, no bar** — wallpaper breathes through the gaps. The centerpiece is a morphing notch: minimal clock when idle, expanding for volume, media (with audio visualizer), and — the novel, headline feature — **live Claude Code agent status with permission approval directly from the notch**.

You are PORTING ideas from a reference project (Hyprfabricated, Fabric/Python+GTK) into our stack (Quickshell/QML). Study the reference for HOW; rebuild in Quickshell. Do not copy Python — translate the technique.

---

## REPO & PATHS (read carefully — this is unusual)

- **THIS REPO / YOUR WORKING DIRECTORY:** `~/Projects/openagentisland/`
  - `quickshell/` — the QML shell code (the actual desktop)
  - `bridge/` — (you create later) the Claude Code agent bridge: hook scripts + socket listener
  - `NOTES.md`, `PROGRESS.md` — design + progress logs (see below)
  - This is a git repo (branch `main`). Commit after each working phase.
- **RUNTIME SYMLINK (don't edit through it — edit the repo):** `~/.config/quickshell/openagentisland` is a SYMLINK → `~/Projects/openagentisland/quickshell`. Quickshell only loads configs from `~/.config/quickshell/<name>/`, so the symlink bridges repo→runtime. `qs -c openagentisland` follows it. **Always edit files in the repo (`~/Projects/openagentisland/quickshell/`).**
- **READ-ONLY REFERENCE (never edit):** `~/Projects/island-reference/hyprfabricated/` — the Fabric project we study.

### HARD RULES (never violate)
1. **NEVER touch `~/.config/quickshell/ii/`** — the user's LIVE desktop. Breaking it breaks their real machine.
2. **Edit ONLY inside `~/Projects/openagentisland/`.** The one exception: Claude Code hook entries in `~/.claude/settings.json` during the agent phase — done carefully and reversibly.
3. **DO NOT merge to `ii`.** The user merges to live config themselves at the end.
4. **User is on `fish`.** `<<EOF` heredocs DON'T work in fish. Write files with your tools, or `printf '%s\n' ...`, or `cat > file` only inside `bash -c '...'`.
5. **Bracketed paste mangles multiline pastes** — prefer writing files directly over asking the user to paste blocks.
6. **Ask before destructive actions; STOP and ask on any UX/design ambiguity.** Don't guess on look/behavior.

---

## NOTES.md — design doc (CREATE & MAINTAIN at repo root)

`~/Projects/openagentisland/NOTES.md` = the **design/architecture reference** ("how it works and why"). Relatively stable — update on architecture decisions, not every edit.

**Create in Phase 0. Must contain:**
- **Reference findings:** after reading `modules/notch.py` + `utils/animator.py`, a concise writeup of Hyprfabricated's notch state model, morph approach, how it composes left/notch/right, and a **Fabric→Quickshell mapping table** (their concept → our equivalent).
- **Architecture:** the three-floating-island design, what each island contains, the full notch state machine (every state, trigger, content), and state precedence.
- **Quickshell/end-4 facts** used (theme tokens, services, panel-family loader, Variants for multi-monitor).
- **Agent bridge design:** socket path, hook events, the JSON schema sent over the socket, the blocking-permission protocol + safety/timeout behavior.
- **Key decisions & rationale.**

NOTES.md is what a new contributor reads to understand the project. Keep it accurate.

## PROGRESS.md — running log (CREATE & MAINTAIN at repo root)

`~/Projects/openagentisland/PROGRESS.md` = the **chronological work log**, updated after every meaningful step.

**Create in Phase 0. Structure:**
- **Current phase & status** at the TOP, always current (which phase, what works, what doesn't).
- **Done:** dated bullets, newest first — what was built + that it was verified in the nested window.
- **Next:** immediate next steps.
- **Blockers / open questions:** anything stuck or needing the user.
- **Gotchas hit:** problems + how solved (so they aren't repeated) — QML errors, API quirks, fish/paste issues.

Update at the end of every session and after each phase. A resuming session (you or the user) should learn exactly where things stand from PROGRESS.md. Be concise but specific, e.g.: "Phase 3 done: volume state morphs cleanly, verified by changing volume in nested window; flicker fixed by watching Audio.volume value, not osdVolumeOpen flag."

---

## DEV WORKFLOW (your "dev server")

A **nested Hyprland-in-a-window** renders `openagentisland` live, isolated from the real desktop — like `npm run dev`.

- Nested config: `~/.config/hypr-nested/hyprland.conf` (minimal Hyprland that launches `qs -c openagentisland`).
- User launches it with:
  `WLR_BACKENDS=wayland WLR_NO_HARDWARE_CURSORS=1 HYPRLAND_INSTANCE_SIGNATURE= Hyprland --config ~/.config/hypr-nested/hyprland.conf`
- **Quickshell HOT-RELOADS on file save** — edit a `.qml`, save, nested window updates instantly. No restart for QML changes.
- **Monitor/resolution changes** in the nested hypr config do NOT hot-reload — restart the nested session.
- On a QML error, Quickshell shows a **red "Reload failed" panel** with exact `file:line` + reason — USE IT.
- **NEVER run `qs -c openagentisland &` from a terminal that will close** — it dies with the terminal. Use the nested autostart or `hyprctl dispatch exec`.
- After each change: tell the user "save's in, check the nested window" and describe what they should see.

---

## end-4 / QUICKSHELL FACTS (don't rediscover these)

- **Quickshell 0.2.1** (modern QML API). Components are `.qml`.
- **Panel families:** `quickshell/shell.qml` loads `IllogicalImpulseFamily` (`quickshell/panelFamilies/IllogicalImpulseFamily.qml`) — a `Scope {}` of `PanelLoader { component: X {} }`. Register/unregister panels here (disable old `Bar`, add the three islands).
- **Panel pattern:** each panel = folder `quickshell/modules/ii/<name>/`, imported `qs.modules.ii.<name>`. Study existing: `modules/ii/bar/`, `mediaControls/`, `onScreenDisplay/`, `dock/`.
- **Theme tokens (Material You, wallpaper-adaptive) — ALWAYS use, never hardcode colors:**
  - `Appearance.colors.colLayer0/1/2`, `colOnLayer0/1/2`, `colLayer0Border`
  - `Appearance.rounding.windowRounding` (=18), `Appearance.rounding.full`
  - `Appearance.sizes.baseBarHeight` (=40)
  - `Appearance.font.pixelSize.normal/large/larger`
  - `Appearance.animation.elementMoveFast.*` (reuse prebuilt animations)
- **Custom widgets** in `qs.modules.common.widgets`: `StyledText`, `MaterialSymbol`, `RippleButton`, `Revealer`. Must import `qs.modules.common.widgets` or "X is not a type".
- **Services (REUSE, don't rebuild):** `Audio` (`Audio.sink.audio.volume/.muted`), `Brightness` (`Brightness.getMonitorForScreen(screen)`), `MprisController` (media), `Notifications` (`.unread/.silent`), `Battery`, `Network`, `BluetoothStatus`, `ResourceUsage` (CPU/RAM/SWAP), `TimerService`, `DateTime`.
- **Multi-monitor:** wrap every floating island in `Variants { model: Quickshell.screens; PanelWindow { required property var modelData; screen: modelData; ... } }`. Renders on every monitor. (Hyprfabricated LACKS this — our advantage; all three islands must use it.)
- **GlobalStates** (`quickshell/GlobalStates.qml`): `sidebarLeftOpen`, `sidebarRightOpen`, `osdVolumeOpen`, `overviewOpen`. NOTE: `osdVolumeOpen` flickers during scroll — trigger the notch by watching `Audio.sink.audio.volume` *value changes* directly, not this flag.

---

## REFERENCE MAP (read in Hyprfabricated; Quickshell target)

Read READ-ONLY in `~/Projects/island-reference/hyprfabricated/`. Translate technique, don't copy Python.

| Reference file | Extract | Quickshell target |
|---|---|---|
| `modules/notch.py` | notch states, morph/switch logic, sizing, per-state content | `quickshell/modules/ii/island/IslandNotch.qml` — state machine + `Behavior` animations |
| `utils/animator.py` | bezier-curve tween technique | `Behavior on width/height/opacity { NumberAnimation { easing.bezierCurve: [...] } }` |
| `modules/cavalcade.py` | cava visualizer | Quickshell visualizer; also study end-4 cava in `modules/ii/mediaControls/` |
| `modules/player.py` | media art/title/controls | `MprisController`; ref end-4 `mediaControls` |
| `modules/bar.py` | left/right clusters as separate floating pieces | `IslandLeft.qml` / `IslandRight.qml` |
| `modules/metrics.py` | CPU/RAM/SWAP | `ResourceUsage` |
| `modules/controls.py` | volume/brightness OSD content | notch volume/brightness states |
| `utils/hyprland_monitor.py`, `utils/occlusion.py` | monitor handling / hide-behind-windows (note multi-monitor gap) | we do multi-monitor via `Variants` |
| `modules/corners.py` | screen-corner rounding | end-4 `modules/ii/screenCorners/` |

**Phase 0 first action:** read `modules/notch.py` fully, then write the reference-findings section in `NOTES.md`.

---

## TARGET DESIGN

**Three FLOATING islands, no full-width bar, always visible, on every monitor.** Wallpaper through gaps.

### Left island — `IslandLeft.qml`
Floating top-left, rounded, transparent bg, themed surface. Workspace indicators (dots expanding for active, Hyprfabricated-style) + window title. Left-click → toggle `sidebarLeftOpen`. Right-click workspaces → `overviewOpen`.

### Center notch — `IslandNotch.qml` (THE STAR)
Floating top-center, rounded, morphing. **State machine** (`property string islandState`):
- `idle` → minimal **clock / small info** (NOT invisible).
- `volume` → expand, volume icon + level bar. Trigger: `Audio` value change. Auto-return ~2s.
- `brightness` → same pattern.
- `media` → expand: track art + title + **cava visualizer** + controls. Active while media plays.
- `agent` → THE BIG ONE: Claude Code session status, live updates, permission prompts.
- `notification` → brief expand for incoming notifications.

**Morph** with goey spring easing (`Behavior on width/height`, OutBack-style or bezier from `animator.py`). `implicitWidth` reflects active state so layout reserves space.

**State precedence:** agent-permission > agent-status > media > volume/brightness > notification > idle.

### Right island — `IslandRight.qml`
Floating top-right, rounded. Left→right: Resources (CPU/RAM/SWAP via `ResourceUsage`), clock, battery, system tray, wifi/bt. Left-click → toggle `sidebarRightOpen`. Keep ONLY the performance toggle from UtilButtons (user removed keyboard/brightness/darkmode).

### Bar removal
In `IllogicalImpulseFamily.qml`, DISABLE the full-width `Bar` (comment its `PanelLoader`). Add the three island `PanelLoader`s. Keep ALL other panels (sidebars, overview, lock, notifications, dock, screenCorners, etc.).

---

## AGENT MONITOR (novel, highest value)

Claude Code sessions appear LIVE in the notch — status, activity, questions — with **Allow/Deny permission approval from the notch**.

### Transport: Unix socket (MANDATED)
- Claude Code **hooks** (`~/.claude/settings.json`) fire on events, send JSON to a **Unix domain socket** (`$XDG_RUNTIME_DIR/openagentisland.sock`, fall back `/tmp/openagentisland.sock`).
- A listener reads the socket → updates the notch agent state.
- **Permission round-trip:** for permission events the hook BLOCKS waiting on the socket; notch shows Allow/Deny; user's click sends the decision back; hook returns the matching exit code. Hardest piece — design carefully, test in isolation first.
- All bridge code (hook scripts + listener) in `~/Projects/openagentisland/bridge/`.

### Hook events
`SessionStart`, `UserPromptSubmit`, `PreToolUse`, `PostToolUse`, `Notification`, `Stop`, + permission/approval. Each sends `{event, session_id, cwd, tool, message, ...}`. Document the schema in NOTES.md.

### Quickshell side
- Use Quickshell `Socket`/`Process`/IO primitives (research the `Quickshell.Io` API in 0.2.1 — confirm it supports what's needed; if bidirectional blocking is awkward, flag to the user before proceeding).
- Per-session state: running / idle / waiting-for-input / waiting-for-permission.
- Render in notch `agent` state: project (cwd basename), status text; for permission: request + Allow/Deny.
- Multiple sessions: count, cycle/stack. Start with ONE, then add multiple.

### CRITICAL SAFETY
A blocking hook can HANG Claude Code if the notch isn't listening/crashes. MANDATORY: a timeout (hook falls back to default behavior if no response in N seconds) + graceful handling when the listener is down. **A broken island must NEVER make real Claude Code unusable.** Test this failure mode explicitly before the agent phase is "done".

### First-pass scope
Claude Code only (not Codex/Gemini). ONE session's status live → then permission approval → then multiple.

---

## BUILD ORDER (verify each in the nested window; commit after each)

**Phase 0 — Orient.** Read `modules/notch.py` + `utils/animator.py`. Create `NOTES.md` (reference findings + architecture + Fabric→Quickshell map) and `PROGRESS.md` (Phase 0 status). Confirm the nested dev window runs `openagentisland`.

**Phase 1 — Floating skeleton.** Disable full-width `Bar`. Three floating `PanelWindow`s (`IslandLeft`, `IslandNotch`, `IslandRight`), each in `Variants` over `Quickshell.screens`, transparent bg, rounded, anchored top-left/center/right with margins. Static themed pills. Register in family. Verify: three floating islands, wallpaper through gaps, no bar.

**Phase 2 — Populate left & right.** Left: workspaces + title, click→left sidebar. Right: resources + clock + battery + tray + wifi/bt, click→right sidebar. Verify against the user's layout.

**Phase 3 — Notch idle + volume.** Idle = minimal clock/info. `volume` state via `Audio` value change, goey morph, auto-hide ~2s. Verify by changing volume — clean, no flicker loop.

**Phase 4 — Brightness + notifications states.** Same morph pattern.

**Phase 5 — Media + visualizer.** `media` state: art + title + controls + cava visualizer. Verify with music playing.

**Phase 6 — Agent bridge (status only).** Build `bridge/` socket listener + Claude Code hooks for status events. Build the timeout/failure safety FIRST. Notch `agent` state shows one session's live status. Verify with a real Claude Code session.

**Phase 7 — Permission approval.** Blocking permission round-trip: notch Allow/Deny → decision back → hook respects it. Test failure/timeout path hard.

**Phase 8 — Multiple sessions + polish.** Concurrent sessions. Final animation/spacing polish, state precedence, multi-monitor verification.

---

## DEFINITION OF DONE (sandbox)
- Three floating islands, no bar, on every monitor, matching the user's layout.
- Notch morphs cleanly: idle(clock) ↔ volume ↔ brightness ↔ media(+visualizer) ↔ agent, correct precedence, goey animation.
- Claude Code agent status live in the notch; permission Allow/Deny works from the notch; a down/broken island never breaks real Claude Code.
- Runs in `openagentisland` via the nested window. NOTES.md + PROGRESS.md current. User merges to `ii` themselves.

## When stuck
- QML errors → read the red panel's `file:line` in the nested window.
- Unsure of a Quickshell API → check existing end-4 modules for a working example before inventing.
- Unsure of design/UX intent → STOP and ask the user.
- Always keep NOTES.md and PROGRESS.md current.

---
> Source: [patheonsceo/Dynamic-island-for-arch](https://github.com/patheonsceo/Dynamic-island-for-arch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-06 -->
