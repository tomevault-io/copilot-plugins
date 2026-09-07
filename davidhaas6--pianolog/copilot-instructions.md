## pianolog

> This is a small Python MIDI recorder plus a rebuildable local analytics pipeline. Core source files live in `src/`.

# Repository Guidelines

## Project Structure & Module Organization

This is a small Python MIDI recorder plus a rebuildable local analytics pipeline. Core source files live in `src/`.

- `src/daemon.py` is the long-running capture entry point that waits for a MIDI device and appends events.
- `src/record.py` owns RtMidi input setup, callbacks, and queueing.
- `src/models.py` defines the serialized `MidiEvent` model; `src/constants.py` holds shared defaults.
- `src/sessionizer/` rebuilds normalized sessions, notes, raw events, rollups, metadata, and validation state into SQLite.
- `src/analyzer/` derives phrase attempts, tempo windows, and daily practice-quality metrics from normalized notes.
- `src/exporter.py` contains MIDI-file export support.

Examples live in `examples/`, project notes in `docs/`, deterministic tests in `tests/`, and captured/runtime artifacts under `data/`. Treat JSONL recordings and SQLite databases as generated data, not source.

## Build, Test, and Development Commands

- `uv sync`: install pinned dependencies from `pyproject.toml` and `uv.lock`.
- `uv run src/daemon.py`: run the passive MIDI recorder locally; connect a MIDI input device first.
- `./check-device.sh`: inspect whether the system can see MIDI devices.
- `uv run python src/sessionizer/sessionizer.py --input data/sessions --db data/practice.sqlite3`: rebuild the analytics database from captured JSONL.
- `uv run python src/sessionizer/sessionizer.py --watch --interval-sec 30`: continuously refresh the rebuilt database while developing consumers.
- `uv run pytest`: run the deterministic test suite.
- `uv run python examples/test_out.py`: run the example serialization/output script.

There is no package build step yet; treat this as a runnable application repository.

## Coding Style & Design Conventions

Use Python 3.11-compatible code and standard PEP 8 conventions: 4-space indentation, `snake_case` functions and variables, `PascalCase` classes, and uppercase module constants. Keep modules small and centered on one responsibility. Prefer dataclasses for simple data containers where they fit, as in `MidiEvent` and `RecorderCtx`. Use `pathlib.Path` for filesystem paths and the `logging` module rather than `print` in application code.

Preserve the pipeline boundary:

```text
raw JSONL capture -> normalized rebuild -> derived analytics -> validation
```

Raw capture is the durable source of truth. Prefer deriving new analytics from normalized tables instead of mutating capture behavior or coupling downstream logic to JSONL parsing. New serving or publishing code should rely on `validate_database()` rather than duplicating artifact-health checks.

## Testing Guidelines

Use `pytest`; tests belong under `tests/` and should be named `test_*.py`. Focus on deterministic behavior: event serialization, session splitting, rollups, analyzer outputs, rebuild metadata, validation, and candidate-database publishing semantics. Hardware-dependent MIDI behavior should stay isolated behind fakes or mocks so CI and local runs do not require a connected keyboard.

When changing rebuild behavior, cover both the happy path and artifact safety: malformed input handling, empty rebuilds, metadata counts, validation failures, and preservation of the last good published database.

## Commit & Pull Request Guidelines

Recent commits use short, imperative summaries such as `infra`, `robustify open func`, and `improve readme`. Keep commit subjects concise and action-oriented, for example `add analyzer tests` or `preserve last valid rebuild`.

Pull requests should include a brief description, the commands run for verification, and notes about any hardware-tested behavior. Link related issues when available. Include sample JSONL output only when event schema or capture behavior changes; include schema or validation notes when rebuilt database contracts change.

## Agent-Specific Instructions

Do not commit generated recordings, SQLite outputs, caches, or local environment files. Avoid broad refactors while changing recorder behavior; capture, sessionization, analytics, validation, and export are separate concerns in this repo. Prefer narrow changes that keep the append-only capture path simple and the rebuild path deterministic.

---
> Source: [davidhaas6/pianolog](https://github.com/davidhaas6/pianolog) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
