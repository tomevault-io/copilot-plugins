## optical-sync-gui

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A PySide6 desktop wizard app for measuring IR/RGB timing sync on an Intel RealSense camera against an Image Engineering LED panel. It replaces three standalone scripts (in the sibling `optical_sync_poc_/` directory, which this repo ports/lifts logic from) with one guided flow: Device Select -> Stream Config -> ROI Select -> Calibration -> Live Session.

## Commands

```powershell
# Setup (Windows, Python 3.10+, developed against 3.13)
python -m venv .venv
.venv\Scripts\pip install -r requirements.txt

# Run the full test suite (works with no hardware connected)
.venv\Scripts\python.exe -m pytest -v

# Run a single test
.venv\Scripts\python.exe -m pytest tests/domain/test_running_stats.py::test_mean_of_single_value -v

# Run the app (requires a connected RealSense camera + LED-Panel.exe on PATH past Device Select)
.venv\Scripts\python.exe main.py
```

`pytest.ini` sets `pythonpath = .` and `testpaths = tests`, so `pytest -v` from the repo root works without installing the package. GUI/widget tests construct real Qt objects; on a machine/CI runner with no display, set `QT_QPA_PLATFORM=offscreen` first (the GitHub Actions workflow in `.github/workflows/tests.yml` does this on `windows-latest`). Widget tests share one `QApplication` instance via the session-scoped `qapp` fixture in `tests/conftest.py`.

## Architecture

### Layering: `domain` -> `engine` -> `gui`, plus `state`

- **`domain/`** - pure image/math/calibration/export logic. No Qt, no `pyrealsense2`. Fully unit-tested with plain numpy arrays.
- **`engine/`** - hardware and live-session orchestration. Splits into a pure-Python core and a thin hardware/Qt shell:
  - `engine/metrics.py` (`PairingGapMetric`, `PositionGapMetric` - the `Metric` ABC), `engine/test_session.py` (`TestSession` - start/stop/buffers rows), `engine/acquisition_loop.py` (`AcquisitionLoop` - drives one frame-pair at a time through the metrics) are all pure Python, unit-tested with fakes.
  - `engine/streams.py` (`ContinuousCapture`, device/sensor helpers), `engine/led_panel.py` (`LEDPanel`, a static-method wrapper around the `LED-Panel.exe` CLI), and `engine/session_engine.py` (`SessionEngineThread`, a `QThread`) are hardware-facing and have no automated tests by design - see the "Live Session pipeline" section below and the README's Project Structure note.
- **`gui/`** - PySide6 wizard pages (`gui/pages/`) and reusable widgets (`gui/widgets/`), wired together by `gui/main_window.py` (`MainWindow`, a `QStackedWidget` driving the 5-page flow and persisting choices to `state.gui_state`).
- **`state/`** - `GuiState`, the wizard's own persisted state (`gui_state.json`: last device, resolution/fps, ROI). Separate from `settings.yaml` on purpose - see Configuration files below.

When extending a metric or the live session's data flow, start from `engine/metrics.py`/`engine/test_session.py` (pure, testable) and only touch `engine/session_engine.py` for the hardware/Qt plumbing.

### Live Session pipeline (the core runtime loop)

`gui/pages/live_session_page.py`'s `start_session()` builds a `TestSession` (with `PairingGapMetric` + `PositionGapMetric`) and starts a `SessionEngineThread`. That thread's `run()` opens the RealSense sensors, puts the LED panel into scanning mode, then drives `engine.acquisition_loop.AcquisitionLoop.run_until_stopped()` in a plain Python loop - `AcquisitionLoop` calls `TestSession.process_pair()` per frame pair and invokes three callbacks (`on_frames`, `on_row`, `on_stats`), which `SessionEngineThread` re-emits as Qt signals (`frame_ready`, `row_ready`, `stats_ready`, `session_finished`, `error`) to cross into the GUI thread safely.

Two callback cadences matter and must stay separate:
- **`row_ready`/`on_row` fires on every single frame pair**, unthrottled. `LiveSessionPage._on_row_ready` must stay O(1) - only cheap counter/accumulator updates (frame-drop counts, `domain.running_stats.RunningStats`). It must NOT call `LivePlot.add_point()`/pyqtgraph `setData()` here - that was tried and caused a real GUI freeze (a continuously growing backlog of queued cross-thread Qt signal work that only became visible when the user tried to interact with the window).
- **`stats_ready`/`on_stats` fires only every `display_stride` pairs** (default 10, set in `AcquisitionLoop`/`SessionEngineThread`). This is where plot updates (`LivePlot.add_point`) and stat-tile pushes happen - the rate the GUI thread can actually sustain, matching the same cadence the video panels update on.

`SessionEngineThread.finished` (Qt's own built-in signal, fired only after `run()` fully returns including its `finally` block) - not `session_finished`/`error` - is what re-enables the Start button. Gating on `session_finished` instead would let a new session's camera/LED-panel calls race the old thread's still-in-progress hardware cleanup.

### Naming: UI labels vs. data keys are intentionally decoupled

The live session UI shows "HW TS Latency" and "Optical Sync" as user-facing names, but the underlying `Metric.name`/dict keys/CSV columns are still `pairing_gap_us` and `position_gap_ms` throughout `engine/`, `domain/csv_export.py`, and `gui/widgets/stats_panel.py`'s field keys. Only display text (checkbox labels, chart axis titles, `LivePlot.add_series`'s `display_name` param, stat tile labels) uses the renamed terms. Don't assume a UI label matches its data key when tracing a value back through the pipeline.

### `gui/widgets/live_plot.py` gotcha

`LivePlot` subclasses `pg.PlotWidget`. Its own `clear()`-style method must be called `clear_data()`, not `clear()` - `pg.PlotWidget.__init__` copies several of its own methods (including `clear`) onto the *instance* itself, which in Python takes priority over a same-named method defined on the subclass, silently shadowing it. `add_series(name, color, display_name=None)` keeps `name` as the lookup key used everywhere (`add_point`, `get_series_data`, `set_series_visible`) and `display_name` as an independent, optional legend label.

### Configuration files (three, different purposes)

- **`settings.yaml`** - the one hand-edited file (camera defaults, calibration tuning, live-test tuning). Nothing in the app writes to it.
- **`config.yaml`** - auto-generated by the Calibration wizard step, overwritten wholesale per camera model on every calibration run (`domain/calibration.py`'s `update_config_leds`/`load_led_positions`). Never hand-edit.
- **`gui_state.json`** - the wizard's own last-used choices (`state/gui_state.py`), gitignored, machine-specific.

### Output

Everything a live session or calibration produces lands under `output/` (created automatically): raw/frame-drop CSVs (`domain/csv_export.py`), a static end-of-session plot (`domain/plot_export.py`, matplotlib with the `Agg` backend), and LED on/off debug snapshot PNGs (both periodic-during-run and on-demand via "Save Debug Snapshot").

---
> Source: [guybaruch1/optical_sync_gui](https://github.com/guybaruch1/optical_sync_gui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-01 -->
