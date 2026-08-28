## win10x-linux-desktop

> These instructions apply to the entire repository rooted at

# Win10X Linux Desktop — Codex repository instructions

These instructions apply to the entire repository rooted at
`/mnt/data/win10x_linux_desktop`.

## Product direction

- Build a touch-first Linux desktop for phones, tablets, and touchscreen PCs,
  including an aarch64 target.
- The general shell and interaction baseline is the late, single-screen
  Windows 10X design. Use `research/windows10x/` for historical evidence and
  `docs/interaction-checklist.md` for acceptance work.
- The full-screen Windows 10 tablet-style Start screen and Live Tiles are an
  explicit user requirement. They are an intentional hybrid with the Win10X
  shell, not a defect to “correct” into the stock Win10X launcher.
- OOBE, lock screen, and login are deferred. Do not spend implementation or
  review time on them unless the user explicitly brings them back into scope.
- Dual-screen hardware, hinge-aware layouts, spanning, and Wonder Bar are
  deferred until after the single-screen version. Keep their research isolated
  under `research/windows10x/dual-screen-hardware/`.
- The previous acrylic implementation was deliberately removed so the material
  can be redesigned from a clean baseline. Do not restore the old wallpaper
  sampler, CPU blur cache, tint/noise stack, HostBackdrop element, or compositor
  GLES blur shader unless the user explicitly asks to begin the new acrylic
  implementation. The Start background currently crossfades to the wallpaper
  and adds a semi-transparent white veil; this is intentional simple alpha
  composition, not acrylic blur.
- Research screenshots and Microsoft marks are study references only. Do not
  redistribute them as product assets.

When documents disagree, current explicit user decisions above take precedence.
Update stale project documentation when the affected behavior is changed.

## Before changing code

1. Read `README.md` and the relevant sections of
   `docs/interaction-checklist.md`.
2. Read the latest relevant entries in `docs/lessons-learned.md` before
   debugging or repeating an earlier experiment.
3. Inspect `git status` and the affected code. The worktree may contain user or
   other-agent changes; preserve unrelated work.
4. For Win10X behavior questions, check the local research index and sources.
   Distinguish the 2019 concept, 2020 dual-screen emulator, and 2021
   single-screen build instead of treating them as one specification.
5. Do not assume a checked item in the interaction checklist has been visually
   accepted. Reproduce the current behavior when the task concerns UI.

## Codex command execution

- Use Codex's normal command/PTY tools with
  `/mnt/data/win10x_linux_desktop` as the working directory.
- Do not use or require `~/.qoder/skills/detached-shell/`. That path belongs to
  `.qoder/rules/agent.md` and is not available in Codex.
- Give builds and other long-running commands a bounded initial yield. Use the
  returned session for continued output or a persistent GUI instead of using a
  blocking foreground wait.
- Launching the graphical shell against the host Wayland compositor may require
  approved GUI access even when `WAYLAND_DISPLAY` is correct.
- When restarting a visual test, stop only the instance launched by the current
  task, rebuild/run the current worktree, and make sure the user is not looking
  at a stale shell instance.
- `cargo check` and test harness builds do not refresh an existing
  `target/debug/win10x-shell`. Before launching that path directly for visual
  acceptance, run `cargo build -p win10x-shell` after the final code edit.

## Implementation and verification loop

For a code change, use this sequence unless the task clearly needs less:

1. Reproduce or identify the exact code path before editing. For animated UI,
   compare both the in-animation and settled drawing branches.
2. Make a focused patch. Preserve unrelated changes and user-selected design
   choices.
3. Format only affected workspace packages. For the usual UI work:

   ```bash
   cargo fmt -p win10x-shell -p win10x-comp
   ```

   Do not use `cargo fmt --all` until the missing example files declared by the
   vendored `third_party/fluent-egui` manifest are restored.
4. Run checks proportional to the change. The normal baseline is:

   ```bash
   cargo check -p win10x-shell -p win10x-comp
   cargo test -p win10x-protocol
   ```

5. For visible or interactive changes, run the latest shell and inspect the
   real UI. A successful compile is not visual acceptance.
6. Watch runtime logs while exercising the changed path. In particular, treat
   egui `changed id between passes` warnings as real defects: debug builds draw
   a red 2 px diagnostic rectangle for them.
7. Keep the test instance running when the user has asked to inspect it. Report
   exactly what was verified and leave final visual judgment to the user.

Follow `docs/testing.md` for broader levels:

- L1: nested/standalone shell on the current Wayland desktop for daily work.
- L2: KVM/QEMU regression before releases.
- L3: real touch hardware and aarch64 build/smoke test.

If a normal system screenshot cannot capture the shell, the shell's one-shot
self-capture can be used after building:

```bash
WINUI_EGUI_SCREENSHOT=/tmp/win10x-shell.png target/debug/win10x-shell
```

## UI regression rules learned from this project

- Taskbar command buttons use a full-height slot whose painted and interactive
  rectangle reaches the screen/taskbar bottom edge at every scale.
- Give taskbar containers, buttons, clocks, and other repeated egui controls
  stable explicit IDs. Opening another panel must not shift their auto-ID
  sequence between layout passes.
- Start tile animation is user-selectable and persisted. Windows 10 Mobile's
  bottom-up scale/opacity wave is the default. Its tiles transform as joined
  row layers around one horizontal origin, begin at 91% scale, and settle
  downward while upper layers build toward the rows below; do not regress it
  to independent per-tile center scaling. Disco turnstile, Metro Launcher rise,
  Factor slide/scale, and flutter_metro_ui's joined group sweep are optional
  styles. Start tile groups and their names are persisted and shared by every
  layout. The flutter style transforms each real group about its own board-space
  center while translating 100 epx from the right, fading in, and scaling from
  80%; the first group uses the reference's 1.05 s interval and every later
  group begins 120 ms after the previous group.
  This Start setting is independent from the application-launch animation setting.
  Windows 8 layout is the exception to Start-style selection: it always uses
  this source entrance while preserving the user's saved choice for other layouts.
- Start layout is independently selectable and persisted. Windows 10 keeps the
  application list beside a vertically extensible/scrollable tile board.
  Windows Phone uses a vertically scrollable tiles-only page and a 340 ms
  `easeOutExpo` horizontal push-page into/out of the application list. Windows 8
  follows flutter_metro_ui's resizable 120 px normal / 250 px wide faces, 10 px gap,
  100 px start and 140 px top padding, at-most-six-row column-first groups,
  Metro palette, horizontal scrolling, per-group `easeOutExpo` entrances staggered
  by 120 ms after the 1.05 s first group, and 1.5 s delayed header. Its
  project-added horizontal application list is
  entered by upward swipe or the lower-left button with the same 340 ms nonlinear
  vertical push-page; downward swipe returns. Keep both pages alive during their
  transition so headers and contents move as joined pages rather than disappearing
  at the gesture boundary. Rendering this mode must not rewrite the saved freeform
  Windows 10/Phone tile coordinates.
- Windows 8 derives tile positions from source-list order, so its drag path must
  reorder that list (and update the target group) rather than writing freeform
  `col`/`row` cells that the Metro renderer ignores. Its interaction mirrors the
  Windows 10/Phone drag: the same 8 epx pending threshold activates the grab,
  the floating tile follows the pointer, and every live reorder preview is a
  pure function of the layout captured at drag start plus the pointer, so
  siblings ease out of the way without thrashing and the drop can never be
  silently cancelled between or below groups (nearest group always wins).
  Release settles the tile into its derived slot with the OutBack snap. Draw
  the dragged tile above its siblings. Group titles use a persistent inline
  editor: tapping/clicking a
  title or choosing Rename enters it, Enter/focus loss commits, and Escape
  cancels.
- Application-launch animation is separately user-selectable and persisted:
  Windows 10 Mobile wave, Windows 8 flying tile, Disco's source-specific
  turnstile, Factor's source-rectangle clip reveal, or Metro/system default.
  Spawn the application in the click frame so its cold start runs concurrently
  with any visible shell handoff; the animation must never be a pre-launch
  delay. Old coupled settings migrate to the equivalent launch style. The
  Windows 10 Mobile handoff starts
  from the fully settled 100% tile geometry—never the entrance's pre-shrunk
  state—then joined rows grow outward and fade in a top-to-bottom wave. The
  tapped tile may start 80 ms after its own row but still lifts and disappears;
  it must never expand into a full-screen tile proxy. Fade the application
  list, rail, and search while all tiles lift. Once Start content is gone, keep
  only the full wallpaper plus white veil for a 200 ms cold-start hold before
  revealing the application.
- Windows 8 launch is a distinct, optional style, not the old Win8.x trapezoid
  entrance applied to every tile. Non-selected Start tiles retreat together
  around one board origin to 72% and fade within 240 ms. The selected tile is
  repainted at its exact settled source rectangle, lifts and grows before the
  turn, reaches 90 degrees while still airborne, then changes to an unmirrored
  reverse face and expands across the full output as the 180-degree turn
  completes at 720 ms. Keep the Start wallpaper behind this handoff and keep
  application process startup concurrent with it. The implementation structure
  is informed by `flutter_metro_ui`, but its missing scene retreat and immediate
  source-to-fullscreen interpolation are deliberately not copied; see
  `research/windows-phone-launchers.md`.
- Animated Start entrances and ordinary closes finish the full-screen wallpaper
  plus white-veil crossfade with their final delayed tile. W10M application
  launch is the exception: keep that background fully present through its
  background-only cold-start hold. Use `research/windows-phone-launchers.md`
  when retuning source-derived styles.
- Geometry shared by animation and settled states must come from the same
  constants. Live Tile icons retain the same 45% vertical center while their
  tile rectangle scales.
- Tile drag avoidance is a reversible preview derived from the layout captured
  at drag start. Move colliding neighbors to the nearest local free slot—prefer
  the dragged tile's vacated position—and animate them back when the dragged
  tile moves away; never scan from the global top-left for every collision.
- Start and Task View are shell commands, not running app buttons; do not add an
  app-style bottom running indicator to them.
- Standalone shell runs as an undecorated fullscreen viewport with the
  `win10x-shell` app ID so host compositor chrome is not confused with shell
  pixels.
- Standalone `win10x-shell` is only a host-compositor UI preview. Test actual
  application containment with `win10x-comp`: child clients must inherit its
  nested Wayland socket, and every non-shell xdg-toplevel is configured as
  `Fullscreen` at the complete output geometry. Do not regress this to a
  host-controlled window or a maximized-minus-taskbar rectangle; shell chrome
  remains the transparent full-output overlay above the active app.
- Application processes must not share the shell/test PTY foreground process
  group. Keep launches detached so restarting a shell preview cannot propagate
  Ctrl+C into applications. In nested mode, prefer the desktop entry's direct
  `Exec` path so it inherits `win10x-comp`'s Wayland socket before falling back
  to desktop activation helpers.
- Taskbar application buttons include persisted pins and live unpinned apps.
  Primary click launches an unopened app, minimizes the current active app, or
  restores/switches to a minimized or inactive running app according to
  compositor state;
  secondary click or an upward icon swipe opens the application command menu.
  Keep close, pin/unpin, new-window, split, and floating actions wired to real
  session behavior rather than shell-only visual placeholders.
- Switching to an existing window uses flutter_metro_ui's snapshot handoff
  without delaying activation: Start/no-active restores grow the target snapshot
  from the taskbar icon to the app viewport in 200 ms with ease-out-quad;
  app-to-app switches use the 500 ms old-page retreat plus directional incoming
  slide/scale sequence. Keep the taskbar above the proxy. Full-screen proxies
  use a separate, correctly oriented capture at 25% of the output width and
  height; never enlarge the taskbar menu's bounded thumbnail. Capture
  single-window previews as well as multi-window menu previews. Spread multiple
  window readbacks round-robin across the refresh cycle, and keep thumbnail
  scaling, PNG encoding, and atomic writes on a single bounded background queue;
  never block the render thread or allow stale captures to accumulate. Use a
  neutral app-icon fallback if no snapshot is ready.
- Application-switch animation is independently persisted and may be disabled.
  When disabled, activate the real window immediately without a shell proxy and
  stop the compositor's background GPU readback/PNG capture work. Real
  multi-window taskbar previews remain available through bounded thumbnail-only
  captures requested while that menu is actually open.
- When an app has multiple xdg-toplevels, its taskbar menu shows one real
  compositor-rendered preview per window. Preserve stable window IDs across the
  runtime bridge, refresh thumbnails after clients finish their first paint,
  and make a preview selection raise/focus that exact window rather than an
  arbitrary window with the same app ID.
- Floating geometry alone is insufficient for borderless clients. Preserve the
  compositor-owned mouse/touch drag strip at the top of floating surfaces, and
  keep shell chrome above a moving app with `activate = false`.
- Shell chrome must be raised above app surfaces with `activate = false`.
  Rendering or hit-testing z-order must not make the overlay the active window;
  the newest fullscreen application owns activation and keyboard focus until
  the user explicitly interacts with shell chrome.
- A full-output shell surface cannot be rejected only after calling
  `Space::element_under`: that lookup has already stopped at the shell. For
  desktop-area input, iterate top-to-bottom while filtering out shell first so
  pointer and touch reach the application underneath.
- Taskbar and Task View running state comes from the compositor's real
  xdg-toplevel session bridge, including apps not launched by shell. Preserve
  app-id/title/active synchronization and make taskbar/Task View selections
  activate an existing compositor window rather than launching a duplicate.

## Lessons learned (mandatory)

When implementation, debugging, packaging, or testing hits a meaningful
blocker or a misleading failed diagnosis, append a dated entry to
`docs/lessons-learned.md`. Include:

1. **Problem** — observable failure or blocker
2. **Cause** — confirmed root cause or clearly labelled best hypothesis
3. **Resolution** — what fixed or worked around it
4. **Tried but failed** — failed approaches worth preventing others from
   repeating

Do not rewrite old entries to hide failed attempts.

## Development log (mandatory)

Every completed user-requested feature, behavior change, bug fix, build or
configuration change, and project-rule/documentation change must be appended to
`docs/development-log.md` before the task is reported complete. Make the entry
in the same task that makes the change; do not rely on chat history as the only
record.

One entry may cover a coherent group of related edits. Each entry must include:

1. **Request** — what the user wanted or what behavior had to change
2. **Change** — the resulting implementation or documentation change
3. **Files/components** — the important areas touched
4. **Verification** — commands run and visual/manual checks performed; write
   `not run` with a reason when no check was appropriate
5. **Status/follow-up** — complete, awaiting user visual confirmation, or any
   remaining scoped work

Use `docs/development-log.md` for the change history and
`docs/lessons-learned.md` for blockers, root causes, and failed approaches. A
change that also produced a reusable debugging lesson belongs in both files.
Pure read-only inspection or advice with no repository change does not need a
development-log entry. Keep the log append-only; corrections should be a new
entry rather than silently rewriting historical records.

## Recovering conversation context

The Qoder backup at `.qoder/session-context-backup.md` is not the Codex chat
history. If the user explicitly asks to reconstruct earlier Codex work, locate
the project thread in `~/.codex/session_index.jsonl`, then read the matching
`~/.codex/sessions/YYYY/MM/DD/rollout-*.jsonl` whose session ID and `cwd` match
this repository. Prefer durable project facts in this file and
`docs/lessons-learned.md` over depending on machine-local chat history during
ordinary work.

---
> Source: [ycrrongos/win10x-linux-desktop](https://github.com/ycrrongos/win10x-linux-desktop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
