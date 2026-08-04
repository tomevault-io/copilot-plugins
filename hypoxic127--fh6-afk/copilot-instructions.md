## fh6-afk

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A Windows-only, fully automated AFK farming bot for **Forza Horizon 6**. It drives the game with
no human input by combining **computer vision** (OpenCV + Tesseract OCR + color histograms) for
perception and a **virtual Xbox 360 controller** (vgamepad/ViGEmBus) for input. A Flask + SocketIO
**web dashboard** provides monitoring and control. Runtime requires Windows + Tesseract OCR +
ViGEmBus installed, and the game running in English, windowed/borderless.

## Commands

```bash
python setup.py                      # one-time: create venv + install deps
python main_bot.py --web             # run web UI (default http://localhost:6800)
python main_bot.py --web --port 8080 # custom port
python main_bot.py                    # terminal mode (interactive lang/mode/phase menu)
python main_bot.py --lang zh --skip-update --web   # non-interactive flags
python main_bot.py --debug --web                   # enable OCR debug-image dumps (see below)

python -m pytest                              # run tests (hardware tests auto-excluded, see below)
python -m pytest tests/test_ocr.py            # single file
python -m pytest tests/test_ocr.py -k name    # single test by keyword
python -m pytest -m hardware                  # run ONLY the hardware tests (needs Windows + game)

python -m ruff check .                 # lint
python -m ruff format --check .        # format check (line-length 120, double quotes)

python packaging/build.py              # build single-file dist/FH6AutoBot.exe (PyInstaller --onefile)
```

- `pytest.ini` sets `addopts = -m "not hardware"`, so tests touching vgamepad/MSS/the game window
  are skipped by default. CI (GitHub Actions, ubuntu) runs only this non-hardware subset — most
  perception/input code cannot actually run there.
- A git hook (`core.hooksPath=.githooks`) runs `ruff format` on staged `.py` files at commit time
  and re-stages them, so commits are always formatted. Commit style: Conventional Commits.
- When searching, ignore `dist/` and `build/` — they bundle full copies of site-packages (cv2,
  numpy, …) and a stale copy of the source that pollute grep/glob results.
- `--debug` flips on `engine.ocr.DEBUG_WRITE_FILES`, which dumps annotated OCR crops for
  diagnosing misreads. **Read this flag live as `module_ocr.DEBUG_WRITE_FILES`** (import the
  module, e.g. `from engine import ocr as module_ocr`) — `from engine.ocr import DEBUG_WRITE_FILES`
  binds a copy at import time (frozen `False`) and silently ignores `--debug` (the bug fixed in
  `a66902a`).
- `tools/` holds standalone, **un-packaged** dev scripts — ROI calibration, state calibration, and
  OCR capture/debug (e.g. `tools/capture_ocr_debug.py`, `tools/tool_calibrate_states.py`) — used to
  build/verify the `custom_roi` skill-points region. Not shipped in the `.exe`.

## Architecture

Four layers, with a strict dependency direction (`web` → `macro`/`farm` → `engine`; never the
reverse):

- **`engine/`** — perception + infrastructure. `ocr.py` (skill-points + car-name OCR, multi-PSM
  voting), `state_detect.py` (histogram + OCR hybrid game-state detection), `utils.py` (logging,
  Win32 window ops, gamepad `press_button`, MSS screenshot singleton), `control.py` (cooperative-stop
  primitives — see below), `event_bus.py`, `i18n.py`, `runtime.py`, `updater.py`, `version.py`.
- **`macro/`** — the automation. `master_loop.py` holds `run_master_bot_loop`, the master state
  machine. `core.py` has shared infra + the four `STATE_*` constants + screenshot helpers.
  `navigation.py` / `purchase.py` / `garage.py` / `upgrade.py` implement per-stage menu macros.
  `__init__.py` is pure re-exports (hence the ruff `F401`/`F403` ignores there).
- **`farm/skills.py`** — a self-contained visual sub-state-machine that auto-drives an EventLab race
  to completion (the `STATE_FARM_POINTS` stage delegates here).
- **`web/`** — `server.py` (Flask + Flask-SocketIO), `state_manager.py` (current-state cache),
  `static/` (vanilla-JS dashboard).

### The state machine

`run_master_bot_loop(initial_state, skip_buy, loop)` cycles through four stages:
`STATE_FARM_POINTS → STATE_BUY_CARS → STATE_UPGRADE_CARS → STATE_TRASH_CARS → (loop)`. Each step
captures a screenshot, detects state, and issues gamepad inputs. The car kept vs deleted is decided
purely by the **S2 blue PI badge** on the main car.

### Event bus decouples the bot from the UI

`engine/event_bus.py` is a thread-safe pub/sub singleton (`get_bus()`). Engine/macro code **emits**
events (`"log"`, `"state_change"`, `"bot_started"`, `"bot_stopped"`, `"match_archived"`) and never
imports `web`. `web/server.py::_bridge_events()` subscribes and forwards them to SocketIO clients.
**To surface anything new in the UI, emit a bus event from the bot side and bridge it in
`_bridge_events`** — do not reach into `web` from `engine`/`macro`.

### Web threading model (a real source of past bugs)

- SocketIO runs in `async_mode="threading"` on the Werkzeug dev server. The bot runs in a daemon
  worker thread (`_run_bot`). The stop primitives live in **`engine/control.py`** (kept in the lowest
  layer so `press_button` can check them without breaking the dependency direction): `request_stop()`
  sets a global `_stop_event`, and cooperative checkpoints (`check_stop()` / `interruptible_sleep()`,
  plus the loop's `_check_stop()`) bail out at safe boundaries. **Instant stop** is the fallback for
  when the worker is blocked in a native call: it injects `BotStoppedError` via
  `ctypes.pythonapi.PyThreadState_SetAsyncExc` (`_kill_thread`). `BotStoppedError` subclasses
  `BaseException` (not `Exception`) so the many broad `except Exception:` handlers on the hot path
  can't accidentally swallow a stop request.
- **Long-running SocketIO event handlers must offload to `_socketio.start_background_task(...)` and
  emit results to `request.sid`** — blocking the dispatch thread (e.g. with `force_foreground` +
  sleeps) starves the connection.
- **MSS screenshots are not thread-safe.** There is one global `mss` instance (`engine.utils
  .get_mss()`); call `reset_mss()` when capturing from a different thread or after a GDI failure.
  Bot start/stop and the ROI-capture handler all do this — forgetting it causes silent black/empty
  captures.

### Runtime paths & config (dev vs frozen .exe)

`engine/runtime.py` separates read-only code from writable data: `get_base_dir()` returns the source
root (or PyInstaller `_MEIPASS` when frozen), while `get_user_dir()`/`get_data_dir()` return the
writable location (the `.exe`'s folder when frozen). **All runtime files live in `data/`**
(gitignored): `bot_config.json`, `play_archive.jsonl`, `race_state.json`, `roi_temp.jpg`. Read/write
config only through `load_bot_config()` / `save_bot_config()`; keys are `points_per_match`,
`target_points`, and `custom_roi` (the user-calibrated skill-points OCR region, as
`[top, bottom, left, right]` fractions of the raw client-area screenshot).

### i18n

Backend user-facing strings go through `t("key")` from `engine/i18n.py` (en/zh; `set_lang()` once at
startup). The **web frontend has its own separate** `I18N` table inside `web/static/app.js`.

### Frontend gotchas

- `web/static/app.js` is a **classic script** (not a module): a single uncaught error during
  top-level execution aborts everything after it, while hoisted `function` declarations stay defined
  (so an `onclick` may resolve but then operate on uninitialized state). Keep top-level code
  defensive.
- Static assets are cache-busted manually in `index.html` (`app.js?v=N`, `style.css?v=N`). **Bump
  `N` whenever you edit those files**, or browsers serve stale code.

---
> Source: [hypoxic127/FH6-AFK](https://github.com/hypoxic127/FH6-AFK) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-29 -->
