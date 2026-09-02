## face-recognition-app

> This file defines repository-specific operating rules for Codex.

# AGENTS.md

This file defines repository-specific operating rules for Codex.

## Goal

- Improve this Windows-only local face recognition playground safely and deliberately.
- Preserve a clean main UI and avoid degrading the user flow.

## Technical Baseline

- Dependency management: `uv`
- GUI: `CustomTkinter`
- Face detection and embedding: `OpenCV DNN (YuNet + SFace)`
- Liveness check: `MediaPipe Face Landmarker`
- Persistence: `SQLite`
- Analytics: prefer self-contained `Plotly` HTML reports instead of embedding heavy web UI into the main window

## Mandatory Rules

- Add dependencies only with `uv add` or `uv add --dev`
- Do not introduce suspicious or low-trust libraries
- Do not use `from __future__ import annotations`
- Do not use `Any`
- Do not weaken type safety
- Do not ignore issues that are visible in `Pylance` or `Pyright`
- Keep the main UI simple
- Favor a restrained public-service-style visual language
- Avoid decorative excess, verbose explanatory UI, too many cards, or unnecessary new flows

## Implementation Rules

- Preserve the existing layer structure
  - `domain`
  - `gateways`
  - `infra`
  - `app`
  - `strategy`
  - `ui`
- Keep UI formatting logic in `view_model` or the UI layer when possible
- Put SQLite aggregation and translation logic in `gateways` or `app`
- For analytics features, prefer a separate report before adding complexity to the main window
- If static typing becomes awkward, improve the type design instead of muting the checker

## UI Rules

- The camera preview is the primary surface
- Favor the shortest usable flow
- Do not create states where the user presses a control and nothing appears to happen
- Keep status presentation compact
- Use a quiet white-based palette with a limited blue accent
- Do not overload the right column
- Before adding a new section, check whether the feature can fit into an existing card

## Before Changing Code

- Read the related files first
- Do not break the current responsibility boundaries
- Explicit user instructions override this file

## Verification

Before considering a change verified, always run:

```powershell
powershell -ExecutionPolicy Bypass -File .\scripts\verify.ps1
```

This script runs:

- `uv run pyright`
- `uv run ty check .`
- `uv run ruff check .`
- `uv run pytest`
- `uv run python -m compileall app main.py`

Do not treat a change as verified if any step fails.

## Notes

- Do not commit generated output in `data/reports/`
- Do not commit model files or SQLite database files
- For analytics work, prioritize charts that help operational judgment
- Avoid flashy features with weak practical value

---
> Source: [shuuuumai96/face-recognition-app](https://github.com/shuuuumai96/face-recognition-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-02 -->
