## battery-manager

> Before writing any code, read **all** spec files in this directory:

# CLAUDE.md — battery-manager@local

## Mandatory pre-task

Before writing any code, read **all** spec files in this directory:

```
00-overview.md
01-usage-history.md
02-battery-tab.md
03-power-adapter-tab.md
04-schedule-tab.md
05-notifications-and-icon.md
06-gschema.md
07-panel-menu.md
```

Do not proceed until all files have been read. They are the single source of
truth for every decision in this project.

---

## Project

GNOME Shell extension: `battery-manager@local`
Target: GNOME Shell 49–50, GJS, GTK4 + Adwaita, no root privileges.

---

## Rules

### Spec compliance
- Follow the spec files exactly. Do not add features not described. Do not
  omit features that are described.
- Every D-Bus interface, GSettings key, widget type, Cairo draw routine, file
  path, and systemd unit template is already defined. Use them as-is.
- Items marked `✅ Resolvido`: implement exactly as specified, no substitutions.
- Items marked `⚠️ Aberto`: implement the best-effort approach described in
  the spec. Do not invent alternatives.

### Decision-making
- Do not ask clarifying questions mid-task. If a section is ambiguous, re-read
  the relevant spec file before deciding. The answer is always there.
- Do not hallucinate D-Bus API signatures, GSettings key names, or GJS import
  paths. Use only what is listed in the spec or verifiable from GNOME docs.

### Code quality
- One file per module (matches the file tree in `00-overview.md §3`).
- No global state outside `extension.js`.
- All D-Bus proxies must be destroyed in `disable()`.
- All GSettings `connect()` calls must be disconnected in `disable()`.
- Cairo contexts (`cr`) must call `cr.$dispose()` after each `repaint`.
- Use `async/await` with `Gio.DBusProxy` async calls. No blocking main loop.

### Style
- ESM imports (`import ... from 'gi://...'`), not `const ... = imports...`.
- 2-space indentation.
- Private methods prefixed with `_`.
- No unused variables.

---

## Implementation order

Complete each step fully before starting the next.

| Step | File(s) | Spec reference |
|------|---------|----------------|
| 1 | `metadata.json` | `00-overview.md §2` |
| 2 | `schemas/org.gnome.shell.extensions.battery-manager.gschema.xml` | `06-gschema.md` |
| 3 | `lib/upower.js` | `00-overview.md §4–5`, `05-notifications-and-icon.md §5` |
| 4 | `lib/powerProfiles.js` | `00-overview.md §4`, `03-power-adapter-tab.md §5` |
| 5 | `lib/gsdPower.js` | `00-overview.md §11` — stub only, no callers yet |
| 6 | `lib/notifier.js` | `05-notifications-and-icon.md §3` |
| 6 | `lib/historyStore.js` | `01-usage-history.md §5.2`, `01-usage-history.md §6` |
| 7 | `lib/scheduler.js` | `04-schedule-tab.md §3.4`, `04-schedule-tab.md §4.1` |
| 8 | `ui/charts.js` | `01-usage-history.md §3–5` |
| 9 | `ui/batteryIndicator.js` | `05-notifications-and-icon.md §2`, `07-panel-menu.md §2–4` |
| 10 | `ui/usageHistoryTab.js` | `01-usage-history.md` |
| 11 | `ui/batteryTab.js` | `02-battery-tab.md` |
| 12 | `ui/powerAdapterTab.js` | `03-power-adapter-tab.md` |
| 13 | `ui/scheduleTab.js` | `04-schedule-tab.md` |
| 14 | `ui/panelWindow.js` | `00-overview.md §6` |
| 15 | `extension.js` | `00-overview.md §7`, `05-notifications-and-icon.md §6` |
| 16 | `prefs.js` | `00-overview.md §11` |

---

## Cross-cutting concerns

These must be kept consistent across all files at all times:

- **Energy Mode sync:** `batteryTab.js` (Low Power checkbox) ↔
  `powerAdapterTab.js` (Energy Mode dropdown) ↔ `batteryIndicator.js` (menu
  rows) all share `net.hadess.PowerProfiles` `ActiveProfile`. All three
  subscribe to `PropertiesChanged`. Use a `_syncing` boolean guard to prevent
  write feedback loops. Spec: `03-power-adapter-tab.md §9`.

- **Show battery in menu bar:** `batteryTab.js` checkbox writes to
  `org.gnome.desktop.interface` → `show-battery-percentage`. `extension.js`
  reads this on init and on change to show/hide the indicator.

- **Screen history file path:**
  `~/.local/share/gnome-shell/extensions/battery-manager@local/screen-history.json`
  Written by `lib/historyStore.js`. Read by `ui/usageHistoryTab.js`.
  Retention: 10 days. Spec: `01-usage-history.md §5.2`.

- **systemd unit files path:**
  `~/.config/systemd/user/battery-wake.timer` and `battery-sleep.timer` +
  their `.service` counterparts. Written by `lib/scheduler.js`.
  Spec: `04-schedule-tab.md §8`.

---

## Verification checklist (run after all steps complete)

- [ ] `glib-compile-schemas schemas/` exits 0
- [ ] `gnome-extensions pack` exits 0 with no missing files
- [ ] No `imports.*` syntax anywhere (must use ESM)
- [ ] All `Gio.DBusProxy` instances nulled in `disable()`
- [ ] All `GLib.Source` / timeouts cleared in `disable()`
- [ ] Cairo `cr.$dispose()` present in every `repaint` handler
- [ ] `screen-history.json` pruned to ≤10 days on every write
- [ ] systemd unit files written atomically (write to `.tmp`, then rename)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/amarqs182) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
