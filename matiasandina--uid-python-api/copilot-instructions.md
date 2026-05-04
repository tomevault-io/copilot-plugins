## uid-python-api

> - Production target: Windows.

# AGENTS.md

## Project Context
- Production target: Windows.
- UI is required for live runs.
- Dependencies are allowed (`uv` + venv).
- Prioritize stable behavior and small, focused branches.
- Date phrasing in docs/issues/responses: use `As of YYYY-MM-DD, ...` instead of relative wording like `Today, ...`.
- When printing terminal UI/errors with Rich, render file paths, config keys, literal values, and commands as visibly highlighted inline code-style spans; avoid dumping long plain-text paragraphs when structured panels or short labeled lines would make failures easier to scan.

## Current Runtime Map
- Entry point: `main.py`
- Core orchestration: `device_manager.py`
- TCP device client: `ip_device.py`
- Parsing: `data_parser.py`
- Logging: `datalogger_csv.py`
- Animal model/registry: `animal.py`, `animal_registry.py`
- Live console UI (Rich): `console_display.py`
- Trigger evaluation: `trigger_scheduler.py`
- Stimulation control: `stim_controller.py`
- Preflight UI: `preflight_ui.py`

Flow:
1. `main.py` loads/validates YAML config.
2. `preflight_ui.py` confirms safety-critical settings.
3. `DeviceManager` connects devices and starts loops.
4. Raw packets are parsed and written to CSV.
5. Readings update `AnimalRegistry` and UI panels.
6. Optional trigger scheduler evaluates windows and dispatches stimulation actions.

## Status (main)
Merged and operational:
- Config-driven runtime (`--config`) with validation.
- Rich-based live UI with trigger/stimulation status panels.
- Trigger scheduler with `pulse` and `window` modes.
- Classifier plugin loading from `module:function`.
- Trigger metadata grouped by animal (`triggers_by_animal`).
- Replay support for legacy (`DateTime/UID`) and raw (`Date/RFID`) CSV headers.
- Preflight confirmation gate before acquisition.
- Doric DLL UID-based access (port probing is fallback).
- Per-channel `current_ma` in `stimulus.channels` (as of 2026-04-09).

Pending validation:
- Stress test long sessions and high event volume.
- Decide whether train OFF should remain software-timed or move to Doric-native envelope.

## Priority Backlog
1. Per-channel full square pulse config (#8).
2. Data handling/performance tuning (window/history retention and CSV flush strategy).
3. Logging cleanup (structured logging to file + console, quiet UI mode preserved).
4. Clean exit UX beyond Ctrl+C.
5. UI/API decoupling: stop direct access to `DeviceManager` private fields from `ConsoleDisplay`.

## Branching Suggestions
- `feat/hardware-validation-notes`
- `feat/data-handling`
- `feat/logging`
- `feat/clean-exit`
- `refactor/device-stats-api`

## Classifier Contract
Classifier functions in `classifiers/` should expose:

```python
def evaluate(animal_id, window_readings, now, config) -> dict | None:
    ...
```

Expected decision shape:
- `{"trigger": True, "action": "start"|"stop"|"pulse", "stimulus_id": "...", "reason": "...", "meta": {...}}`

---
> Source: [matiasandina/uid_python_api](https://github.com/matiasandina/uid_python_api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
