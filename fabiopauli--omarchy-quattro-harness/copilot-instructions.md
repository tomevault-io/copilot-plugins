## omarchy-quattro-harness

> `skills/omaharness/SKILL.md` documents the API. This file documents the things

# AGENTS.md — operating omaharness

`skills/omaharness/SKILL.md` documents the API. This file documents the things
that cost time, or a window, when you learn them by discovery. Everything here
was established against a real Omarchy session, not inferred from source.

Read this before your first mutating call.

---

## 0. You may be running inside something you can close

Before closing, killing, or force-quitting any window, find out where you live:

```bash
p=$$; while [ "$p" -ne 1 ]; do read -r ppid comm < <(ps -o ppid=,comm= -p "$p"); \
  echo "$p $comm"; p=$ppid; done
```

An agent hosted in VS Code's integrated terminal has `code` in that chain; one
in a terminal emulator has `foot`, `alacritty`, `ghostty`. Closing that window
kills the agent mid-task — the shell dies with signal 9 and the transcript ends
without an error you can read.

**Never match windows by class for a destructive operation.** `class == "foot"`
is every terminal on the desktop, including the user's, including possibly
yours. Match on the address you got back when you created the window.

---

## 1. `hyprctl dispatch` answers `ok` to calls that do the wrong thing

This is the single sharpest edge on the whole system.

Hyprland ≥ 0.56 takes selectors as **named table fields**. Passed positionally,
the selector is not an error — it is ignored, and the dispatcher runs against
**whatever is focused**:

```lua
hl.dsp.window.close("address:0x55…")      -- ok. Closes the FOCUSED window.
hl.dsp.window.close{ window = "address:0x55…" }  -- ok. Closes that window.
```

Both return `ok`. Nothing downstream can distinguish them. Verified by
experiment: with window A focused and B targeted, the positional form killed A.

Consequences to internalise:

- **Prefer the named intents.** `hyprctl.close_window(addr)`,
  `focus_window`, `focus_workspace`, `send_shortcut`,
  `move_window_to_workspace`, `move_workspace_to_monitor` encode the selector
  correctly for both grammars. `dispatch_raw` now refuses a positionally
  addressed selector, but it only recognises selector-shaped strings — it is a
  safety net, not a guarantee.
- **The stub audit checks names, not signatures.** `doctor` reporting
  `unknown_dispatchers: []` means every dispatcher *name* exists. Hyprland's
  stub declares most of them as `fun(...)`, so nothing validates argument
  shape. A correctly-named call with the wrong argument shape passes every
  check the harness makes.
- **Rehearse destructive dispatchers on throwaway windows.** Spawn two, focus
  one, target the other, and confirm the right one died. Two windows are what
  make the result unambiguous — with one window you cannot tell "honoured the
  address" from "closed the focused window".

Finding the real dispatcher names and namespaces:

```bash
sed -n '/---@class HL.DspWindowNamespace/,/^local /p' \
  /usr/share/hypr/stubs/hl.meta.lua | grep -oP '(?<=@field )\w+'
```

---

## 2. Trust ground truth over metadata

`background_safe`, `interference`, and `warnings` describe *what the harness
believes it did*. They are honest, and they are not proof of delivery.

A key call returning `backend: hyprland-sendshortcut, background_safe: True`
means the dispatch was accepted — not that the keystroke arrived.

Prove delivery with something outside the harness:

```python
# the window writes what it received; the file is the evidence
h.dispatch_raw('hl.dsp.exec_cmd("foot -T probe -- /path/reader.sh",'
               ' { workspace = "5 silent" })')
for k in ["h", "e", "l", "l", "o"]:
    desktop.key(k, app=addr)
assert Path("/tmp/typed.txt").read_text() == "hello"
```

Screenshot diffs are weaker evidence than they look. A `less` window whose file
fits on one screen shows `(END)` before and after a `G` keypress — two
different-looking captures, zero information about whether the key landed.

---

## 3. What each input route can and cannot reach

| route | reaches | background-safe | needs |
| --- | --- | --- | --- |
| `desktop.key` → `sendshortcut` | toplevel windows, by address | **yes** | chord with an XKB spelling |
| `desktop.ax.perform` | GTK/Qt apps exposing a tree | yes | PyGObject on the *right* Python |
| `desktop.type` → `wtype` | the focused window, as text | no | `wtype` (layout-safe; see below) |
| `desktop.click_screen` → `ydotool` | anything under the cursor, incl. layer shell | no | `ydotoold` + uinput access |
| Stage 2 native pointer | window surfaces directly | yes | extension built; **unported on ≥ 0.56** |

**Keys reach an unfocused window.** Verified: with a decoy focused, twelve
chords delivered to a different window's address; focus never moved, the cursor
never moved, and the target received all of them. This is the cheapest correct
route and should be the default.

**`sendshortcut` cannot press a compositor keybind.** This is the corollary
nobody expects. `desktop.key` defaults to `prefer_dispatch=True`, which hands
the chord to the *window's surface* — so the window sees it, and the
compositor's own bind never fires. Every Omarchy shortcut is a compositor bind:
`super+m` (minimize), workspace switches, the launcher. Sending them the
default way is accepted, reports `background_safe: True`, and does nothing.

To reach a bind you need a real key event, which means focus and a transaction:

```python
desktop.key("super+m", app=addr, prefer_dispatch=False)   # fires the bind
desktop.key("super+m", app=addr)                          # the window sees it; the bind does not
```

The receipt tells you which happened: `backend: hyprland-sendshortcut` went to
the window, `backend: ydotool` went through the compositor and carries
`interference`. Prove the bind fired from state the bind changes — for
`super+m`, the window's workspace becomes `special:minimized` — not from the
receipt.

**Check the binding before you send it.** Users rebind things, and stock
Hyprland's `SUPER+M` is `exit` — sending it blind on an unfamiliar machine ends
the session:

```bash
hyprctl binds -j | jq '.[] | select(.key=="M") | {modmask, description}'
```

`modmask` is a bitmask: 1 shift, 4 ctrl, 8 alt, 64 super.

**Layer-shell surfaces are not windows.** The bar, notifications, and the
wallpaper have no entry in `hyprctl clients`, no address, and therefore no
`sendshortcut` and no `ax.query(app=…)` target. Find them with
`desktop.layers()`. Only raw pointer input reaches them.

```bash
desktop.layers()   # namespace, bounds, level — the bar lives here, not in clients
```

Omarchy's bar is **quickshell**, not waybar. It registers on AT-SPI as an
application and exports **zero children** — QML sets no `Accessible`
properties, so there is no element to act on. Do not budget time for an AT-SPI
route to a bar button; there isn't one.

---

## 4. Coordinates

`desktop.click` takes `coordinate_space="screenshot"` by default and needs a
prior `see()`. To click something you never captured — a bar, a notification —
pass **logical layout coordinates** to `desktop.click_screen(x, y)`.
`desktop.capture_output()` captures the focused output and reports the scale
between its physical PNG and logical layout bounds.

Logical ≠ physical on a scaled display. `hyprctl monitors` reports physical
pixels and a scale; `grim` captures physical; the compositor and the harness
speak logical:

```
1920×1080 physical ÷ 1.3333 scale = 1440×810 logical
```

`click_screen` uses the active toplevel as a transaction-restoration anchor.
If no window is focused, pass `anchor_app=` explicitly. The receipt names that
window as `anchor_window`, separately from the actual screen-coordinate target.

**Text goes through `wtype`, chords go through `ydotool`.** This is not a
preference, and swapping them breaks in both directions.

`ydotool` sends key *positions*. It resolves each character to the position a
**US** keyboard puts it on, and the compositor then reads that position through
the layout the user actually has. On this machine (`br`) the slash position
carries `;`, so typing `nvim /tmp/notes.md` delivered `nvim ;tmp;notes.md` —
bash read it as three commands, `nvim` opened with no argument, and the
following prose ran as dashboard keybindings. `ydotool` exited 0 throughout.
Check the layout before you doubt this: `hyprctl devices -j | jq '.keyboards[].layout'`.

`wtype` uploads an XKB keymap containing the exact characters it was handed, so
no layout can reinterpret them — including accented Latin and arbitrary Unicode.
That synthetic keymap is also why it cannot replace `ydotool` for chords:
Hyprland matches keybinds against key positions, so `super+m` sent through
`wtype` reaches the focused surface and fires no bind. Verified both ways on
0.56.2.

`desktop.type` handles this for you. If `wtype` is missing it refuses on any
layout but `us`, rather than typing something else; `doctor` reports it as
`capabilities.text_entry` with `layout_safe` and `keyboard.layouts`.

**Do not drive `ydotool` directly.** Its absolute mapping is not layout pixels —
on this machine it lands at exactly **2×** the requested coordinate (a 15-bit
vs 16-bit `absmax` mismatch), silently, with a zero exit code. The harness
measures the mapping with two probes on first use and runs closed-loop: ask,
read `hyprctl cursorpos`, correct. Anything bypassing that will click the wrong
pixel. `doctor` reports a warning when the measured mapping is not identity;
the harness's own corrected raw-pointer capability remains available.

---

## 5. AT-SPI is a Python-version trap

`python-gobject` is a compiled extension installed for **one** Python minor
version — the system one. A `uv venv --python 3.12` on a host whose system
Python is 3.14 cannot import it, and no amount of `pacman -S python-gobject`
will change that. `--system-site-packages` only helps when the versions match.

```bash
python3 -VV                                  # what the system ships
ls -d /usr/lib/python3.*/site-packages/gi    # what gi was built for
```

If they differ from the venv, either rebuild the venv on the system version
with `--system-site-packages`, or `uv pip install 'omaharness[atspi]'` to
compile bindings into the environment. `doctor` now diagnoses this precisely
instead of prescribing a reinstall that cannot work.

**Chromium and Electron** — Chrome, VS Code, Slack — expose no tree unless
launched with `--force-renderer-accessibility=complete`. Never relaunch a
user's app to get one. For a browser, use CDP; it is better than AT-SPI would
have been.

---

## 6. Working without disturbing the user

The whole point. Do the work on a workspace the user is not looking at:

```python
h.dispatch_raw('hl.dsp.exec_cmd("foot -T scratch -- cmd",'
               ' { workspace = "5 silent" })')
```

`silent` is what keeps the user's view from following. Then capture and drive
that window by address — `see()` photographs a window on an inactive workspace
via `grim-foreign-toplevel` with `background_safe=True`, and `desktop.key`
reaches it without focus.

**The headless-output rung is the unreliable one.** When a window has no
foreign-toplevel identifier, capture tries a temporary headless output before
falling back to a visible focus switch. On Hyprland 0.56.2 that output
frequently renders the wallpaper and bar and nothing else - the window's
surface never composites onto it - and the window's geometry can take ~0.4s to
follow the workspace, roughly four times the harness's settle. Both cases are
now refused rather than returned: reading too early cropped a rectangle that
was still on the user's real screen, which produced a full-resolution
photograph of their desktop labelled `background_safe: True`. If you see

```
headless-output capture unavailable: The headless output rendered no content
```

in `warnings`, that is the guard working. The capture fell through to
`focused-region` and `background_safe` is `False`; report the disturbance
rather than retrying the same window.

Check capturability *before* capturing, so a focus switch never surprises the
user:

```python
desktop.toplevels()   # exported: true  -> invisible capture
                      # exported: false -> see() will focus it and say so
```

Clean up windows you created, by address, with `close_window`. `kill <pid>` is
equally precise and involves no dispatcher at all — a reasonable choice when
the process is yours. Verify the window is actually gone; a `kill` that races
the compositor leaves a stray window parked on `special:minimized` where
nobody, including you, will look for it.

**A minimized window stays focused.** Hyprland moves a minimized window to
`special:minimized` without moving focus, so `activewindow` can name a window
the user cannot see. That matters because the transaction restores focus at the
end of every disruptive burst, and focusing a window on a special workspace
drags it back into view: the restore becomes the disruption. The harness now
detects this and refuses that one restore, reporting

```
focus was not restored to 0x…: it sits on special:minimized, and focusing it
would pull a window the user had put away back onto a visible workspace
```

so a burst that follows a minimize leaves focus wherever the compositor put it
rather than un-minimizing the user's window. Read the warning; don't "fix" the
focus afterwards.

---

## 7. Speed

- **One CLI call per decision point, not per primitive.** Process startup and
  compositor detection dominate; ten primitives in one heredoc cost roughly
  what one costs.
- Poll inside the Python program, never by re-invoking the CLI.
- `desktop.layers()` and `desktop.script("hyprctl … -j")` beat a screenshot for
  anything the compositor already knows: geometry, focus, workspaces, layers,
  and window lists.
- `foot` is the cheapest throwaway window and is background-capturable.
- Reach for a capture only when the answer is genuinely pixels.

## 8. herdr is part of this desktop

Omarchy Quattro ships **herdr**, a terminal workspace manager that organises
panes, recognises coding agents inside them, and exposes the session over a
CLI. If you are running inside one of its panes, `HERDR_ENV=1` and
`HERDR_PANE_ID` are set, and you can create panes, run commands in them, and
coordinate other agents without touching the user's layout.

It composes well with this harness, and the division of labour is clean:

- **herdr** for terminal work — a sibling pane, a long-running command, another
  agent, reading back output.
- **omaharness** for everything that is not a terminal — GUI windows, the bar,
  the browser, captures, native input.

Two rules matter most from outside a pane: never drive the user's *focused*
session, and use a named session (`HERDR_SESSION=<name>`) for anything
experimental. See [HERDR.md](HERDR.md) for the details that only show up in
use — output formats that are not JSON, read sources that return empty until
scrollback exists, and pane geometry.

## 9. When to stop and ask

- Chrome's **"Allow remote debugging?"** prompt is a consent step. Ask the user
  to click it. Do not click it through `ax` or `click()`.
- `StateRestoreError` means the physical pointer was not where the harness left
  it. Something else moved it — possibly the user. Report it; do not re-run the
  same disruptive burst.
- A locked session refuses mutating input, and so does an *unknown* lock state.
  That is deliberate. Do not work around it.
- Anything that installs, compiles, or loads: `native build`, `native load`,
  package installs, udev rules, systemd units. Ask first.

---
> Source: [fabiopauli/omarchy-quattro-harness](https://github.com/fabiopauli/omarchy-quattro-harness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
