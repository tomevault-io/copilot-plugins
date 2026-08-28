## fleet-widget

> **FleetWidget** — a macOS always-on-top 8-bit widget showing the live state of every

# claude_widget

**FleetWidget** — a macOS always-on-top 8-bit widget showing the live state of every
running Claude Code session, and alerting when one needs your attention.

## Commands

```bash
make build     # debug build
make test      # unit tests (FleetWidgetCore; no AppKit needed)
make selftest  # checks header control hit regions against the drawn glyphs
make verify    # test + selftest
make run       # run from build products, no bundle
make app       # assemble .build/FleetWidget.app
make install-run  # build the bundle and launch it
make icon      # regenerate AppIcon.icns from the pixel mark
make install   # copy the bundle to /Applications (survives `make clean`)
make uninstall # remove it from /Applications
make clean
```

Visual check without a screenshot or a live fleet:

```bash
./.build/debug/FleetWidget --render-preview /tmp/preview.png
# also writes /tmp/preview-empty.png
```

Installed location is `/Applications/FleetWidget.app`. After changing code, run
`make install` again to update it — the running copy is not hot-reloaded.

## Architecture

```
FleetWidgetCore  (pure logic, unit tested)
  SessionRecord     on-disk shape of ~/.claude/sessions/<pid>.json
  SessionStatus     busy | idle | waiting | shell | unknown
  ProcessLiveness   kill(pid,0) + sysctl start time, for pid-reuse rejection
  SessionRegistry   polls the registry directory, tolerates partial reads
  TurnAlertEngine   transition table + debounce + baseline + latch
  FleetMonitor      500ms poll loop joining the two

FleetWidget      (AppKit shell)
  FleetPanel        non-activating always-on-top NSPanel
  MainMenu          App menu, so a regular app has About/Hide/Quit and ⌘Q
  FleetContentView  all drawing, hit regions, prominence tiering
  SessionAppearance presentation state (busy/blocked/done/idle/shell/unknown)
  Sprites           string-literal pixel bitmaps, no asset pipeline
  ClaudeMark        the starburst, shared by the header and the app icon
  IconRenderer      writes an .iconset from that same bitmap for iconutil
  PixelRenderer     draws a bitmap as a grid of rects (top-left origin, flipped space)
  AlertSound        mute-aware system sounds
  PreviewRenderer   offscreen PNG render, dev aid
```

## Notes

- **Toolchain is Swift 5.6 / Xcode 13.4.1** (macOS 12.3 SDK). Avoid syntax added in
  5.7+: `if let x {` shorthand, regex literals, `Duration`/`Clock`, `@Observable`.
  AppKit is used rather than SwiftUI for the same reason.
- **The data source is an undocumented internal contract.** `~/.claude/sessions/<pid>.json`
  is written by Claude Code itself; field names and the status vocabulary may change in
  any release. Observed against **Claude Code 2.1.240**. The observed behaviour
  (status vocabulary, `waitingFor` values, precedence) is summarised in `README.md`
  under "How it works".
- The widget is **read-only**. It never writes to `~/.claude`, hooks, or `statusLine`,
  so it cannot break a Claude Code install.
- Token / rate-limit gauges are deliberately **not** in v1: utilization never touches
  disk and is only reachable via the `statusLine` hook payload, which would mean owning
  the user's `statusLine` setting.
- **No binary art in the repo.** Sprites, the header mark and the app icon all come
  from the same string-literal bitmaps, so the icon cannot drift from the widget.
- Local-only, untracked: `.claude/` (settings, skills, commands), `openspec/`,
  `CLAUDE.local.md`. They are working tooling, not part of the shipped project.

---
> Source: [nicodapta/fleet_widget](https://github.com/nicodapta/fleet_widget) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
