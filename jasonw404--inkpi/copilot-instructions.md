## inkpi

> This file is the authoritative coding guide for the redesigned InkPi project.

# InkPi Engineering Guidelines

## Source Of Truth

This file is the authoritative coding guide for the redesigned InkPi project.
`docs/architecture.md` explains the same boundaries for human readers.
Documentation is served via MkDocs: `uv run mkdocs serve`.

## Required Architecture

1. `inkpi-core` is the main orchestrator and configuration authority.
2. `inkpi-display` is the only process allowed to access SPI/GPIO or choose
   full, partial, or skipped refresh behavior.
3. Dashboard pages collect data and render complete 800x480 grayscale images.
   They must not import display drivers or request refresh modes.
4. Management owns system/network facts and future Pi configuration controls.
5. Cross-module and cross-process behavior uses typed contracts from
   `inkpi/contracts.py`.
6. Local service IPC uses versioned JSON over Unix sockets.
7. Slow page collection/rendering must not block control/status requests.

All orchestration, rendering, and ownership logic belongs under `inkpi/`.

## Display Rules

- Callers submit complete frames and semantic metadata only.
- Dirty-region analysis is a refresh-decision input, not a rectangular transfer.
- Page changes, grayscale changes, startup/recovery, and uncertain controller
  state require full refresh.
- Default maximum partial streak is five.
- Failed refreshes make the next attempt a full recovery refresh.
- The panel enters sleep during display-service shutdown.
- Do not enable automatic sleep between refreshes until validated on hardware.

## Dashboard And Management Rules

- Pages implement the native page contract and register explicitly.
- At least one page must remain enabled.
- Page configuration mutations are validated, idempotent, and atomically saved.
- Dashboard pages may consume management facts only through contracts.
- Management may control dashboard state only through dashboard-control
  contracts.
- NetworkManager integration and privileged changes belong in a future narrow
  helper, never in dashboard pages or the web portal process.

## Developer Workflow

- Use Python 3.12 and `uv`.
- Local development: `uv sync --extra dev`.
- Raspberry Pi environment: `uv sync --extra rpi`.
- Tests: `uv run pytest -q`.
- Compile check: `uv run python -m compileall -q inkpi tests`.
- Lint: `uv run ruff check inkpi tests`.
- Previews: `uv run inkpi-preview overview --mock-data` and
  `uv run inkpi-preview overview --mock-data --eink-preview`.
- Never use `pip install`, `python -m venv`, or `virtualenv`.

Pi-only packages must remain in the `rpi` optional dependency group so local
development and tests do not require GPIO/SPI build tooling.

## Font Policy

All rendering fonts are bundled in `inkpi/fonts/` and loaded via
`importlib.resources`. System font paths (`/usr/share/fonts`,
`/System/Library/Fonts`, etc.) are forbidden in source code. Architecture
tests in `tests/test_font_architecture.py` enforce this policy. New font
requirements must be bundled and registered in `inkpi/ui/drawing.py`.

---
> Source: [JasonW404/InkPi](https://github.com/JasonW404/InkPi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
