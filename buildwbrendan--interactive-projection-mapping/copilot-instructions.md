## interactive-projection-mapping

> Orientation for an AI agent (or a human) picking this repo up cold on an

# AGENTS.md

Orientation for an AI agent (or a human) picking this repo up cold on an
unfamiliar machine. Read this before changing anything.

Tool-agnostic on purpose — Claude Code, Cursor, Copilot, Aider and friends all
read this file.

## What this is, in three lines

A projector paints a black webpage on a wall. Polygons on that page sit over
real objects. Hovering or clicking a polygon sends a WebSocket event that fires
real hardware — a smart switch, a motor behind a bookshelf.

**There is no camera, no calibration, and no computer vision.** The editor
canvas *is* the projector rectangle, 1:1. A polygon drawn at (x, y) lands at
(x, y) on the wall. If you find yourself reaching for OpenCV, you've
misunderstood the design — read "Why there's no calibration" in the README.

## First contact: prove it runs before you change it

Nothing here needs hardware. Do this first, every time:

```bash
uv sync
uv run python -m runtime --devices data/devices.dev.yaml
```

(No uv on the machine? `docker compose up` runs the same thing sandboxed —
see the README's Docker section.)

`data/devices.dev.yaml` maps every device to a mock that logs each dispatch
to stdout as a `[mock] <object_id>: ...` line. **Clicks cannot reach the real
switch or the real motor.** Use it for all development.

Then, in another shell:

```bash
curl -s localhost:8000/api/editor/scene     # {"scene_available":false,"projector":[1920,1080]}
curl -s -o /dev/null -w '%{http_code}\n' localhost:8000/editor
curl -s localhost:8000/api/objects
```

If those three work, the server is healthy and anything else you're seeing is
a browser or a hardware problem, not a server problem.

## Setting up on a new machine

### 1. Projector resolution — the one thing you must get right

Everything hangs off this. The canvas is the projector rectangle, so a wrong
value puts every polygon somewhere else on the wall, and **nothing in the logs
will tell you.**

```bash
uv run python scripts/list_displays.py
```

It prints every display, flags the main one, guesses which is the projector,
and hands you the exact command:

```
2 display(s):

  Color LCD: 3024x1964  ← main (your laptop)
  Optoma HD: 1920x1080  ← candidate projector

Projector is probably 'Optoma HD'. Start the server with:

    uv run python -m runtime --projector-size 1920x1080
```

The script shells out to the OS (`system_profiler` / `xrandr` / `wmic`) — no
dependency, no camera. If your platform isn't handled it exits 1 and tells you
to read the number off OS display settings, which is the only thing the server
actually needs.

Default is `1920x1080`. A malformed value exits 2 with a clear message rather
than silently falling back — that's deliberate, see `parse_projector_size` in
`src/runtime/__main__.py`.

### 2. Getting the show onto the second monitor

There is **no programmatic display targeting** in this repo, by design — it was
a whole calibration subsystem and it earned nothing. The flow is manual:

1. Open <http://127.0.0.1:8000/> in a browser.
2. Drag that window onto the projector display.
3. Press F11 (or ⌃⌘F on macOS) for fullscreen.

The author's own pattern: keep `/` and `/editor/projection` as two tabs of
that one fullscreen window. The projection page is an editable 1:1 twin of
the show (drag vertices or bodies, ⌘-Z undoes), so shape alignment happens
at wall scale — and the show is one tab switch away.

### On first start, tell the human which windows to open

The first time you start the server for someone, don't just report "server
running" — hand them the three windows (swap `:8000` for whatever `--port`
was used):

1. **<http://127.0.0.1:8000/>** — the final scene. Fullscreen on the
   projector display (F11 / ⌃⌘F) so nothing but the scene is on the wall.
2. **<http://127.0.0.1:8000/editor>** — on their computer monitor. Create
   polygons, bind them to devices and logic, see the whole scene at once.
3. **<http://127.0.0.1:8000/editor/projection>** — on the projector, ideally
   a second tab of the same fullscreen window as `/`. The editable 1:1 twin:
   drag and ⌘-Z shapes at wall scale until they sit on the real objects.

The server prints this same list on boot; your job is to make sure the
human actually sees it.

**An agent cannot do steps 2 and 3.** No API places a window on a physical
display. If you're an agent and the task needs the projection actually on the
wall, do everything else and then tell the human to drag the window — don't
fake it, and don't add a window-management dependency to work around it.

### 3. Hardware credentials (only for the real demos)

```bash
cp .env.example .env
```

`SMART_SWITCH_HOST` plus `KASA_USERNAME` / `KASA_PASSWORD` for the lamp. Find the
switch with `uv run python scripts/probe_kasa_discover.py`. Never commit `.env`;
it's gitignored and must stay that way.

## How to test a change

```bash
uv run pytest              # the whole suite, no hardware, no network
```

Every test runs offline. If you add one that needs a real device on the LAN,
mark it `@pytest.mark.hardware` so it's skipped by default.

**What an agent can verify on its own:**

| Check | How |
|---|---|
| Server boots, routes serve | `curl` the endpoints above |
| Canvas geometry is right | `GET /api/editor/scene` returns your `--projector-size` |
| A polygon is bound to a device | boot banner prints `✓ <id>` — `✗ (no device)` means unbound |
| Editor/style pages render and behave | drive a headless browser against `/editor` |
| Devices fire | run with `--devices data/devices.dev.yaml` and watch stdout for `[mock] <id>: left-click ...` |

**What needs a human:** whether the outline actually lands on the lamp. That is
the only real acceptance test, it happens on a wall, and no amount of passing
tests substitutes for it. Say so plainly rather than implying you checked.

## Map of the code

```
src/runtime/
  __main__.py     CLI. --projector-size lives here, and the boot banner.
  server.py       FastAPI app, all routes, the WebSocket hub, object API.
  editor.py       polygon model + validation. Save path goes through here.
  devices/
    base.py       the Device interface: on_hover / on_click / on_context_menu / close
    registry.py   type-name → factory. Register new drivers here.
    kasa_device.py, book_pusher_device.py, udp_device.py, http_device.py, mock_device.py
  static/         one HTML file per page; no build step, no framework, no npm
data/
  devices.yaml       object_id → driver bindings (real hardware)
  devices.dev.yaml   the same ids, all mocks — use this
  objects.json       your polygons. Gitignored: it's per-wall, not portable.
  examples/          worked objects.json for both demos
```

### The two-window model

Every authoring page has a projector twin, live-synced over the draft API.
Confusing them is the most common way to get lost:

```
  laptop (you look at)      projector (the wall shows)
  ────────────────────      ──────────────────────────
  /editor              →    /editor/projection
  /style               →    /style/preview      (clicks log on the page, nothing fires)
                            /                   ← the actual show
```

## Invariants — don't break these

- **`--mock` and `devices.dev.yaml` must never reach real hardware.** This is
  the safety property the whole dev loop rests on. If you touch device
  construction, re-read `load_registry` in `src/runtime/config.py`.
- **`/style/preview` renders like `/` but fires nothing.** Someone is dragging
  colour sliders next to a real lamp. Keep the preview inert.
- **The editor canvas viewBox must stay `0 0 <projector_w> <projector_h>`.**
  That equality is what replaces calibration. Break it and every saved
  coordinate is silently wrong.
- **Saving must round-trip unknown keys.** `/editor` doesn't manage
  `appearance` / `animation` / `sound`, but it must preserve them or a save
  from the editor wipes everything set in `/style`.
- **`data/objects.json` is per-wall.** Never commit one; never overwrite a
  user's without asking. Examples belong in `data/examples/`.

## Things that will waste your time

- **Display sleep blanks the projector while the server reports healthy.** A
  "no signal" badge on the wall, nothing in the logs. `caffeinate -d` on macOS.
- **`/` reads `/api/objects` once, on page load.** A change isn't on the wall
  until you save *and* reload that tab. `/style/preview` polls, so it updates
  live — test there.
- **Browsers block audio until a real user gesture.** The first hover on a
  fresh page is silent. Any click unlocks the session. Not a bug.
- **Clicking `book_pusher` also turns on `lamp_switch` after 3s.** A deliberate
  hardcoded chain from the original video — search `HARDCODED CHAIN` in
  `server.py`. Documented in the README so it doesn't read as a bug.
- **Moving the projector invalidates every polygon.** There's no calibration to
  re-run and nothing compensates for a new aim. Redraw, or don't move it.

## Adding a device

One class, up to four methods, then register the type name:

1. New file in `src/runtime/devices/`, subclass `Device` from `base.py`.
2. Add a factory to `registry.py` under a type name.
3. Reference that type from `data/devices.yaml`, and add a `mock` entry with
   the same `object_id` to `data/devices.dev.yaml`.
4. Test it against the mock before you point it at anything real.

The `Device` base class is ~60 lines and is the best thing to read first.

---
> Source: [buildwbrendan/interactive-projection-mapping](https://github.com/buildwbrendan/interactive-projection-mapping) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-25 -->
