## mjswan

> mjswan packages browser-based MuJoCo simulations with real-time policy control into an interactive static web app. It has two sides:

# AGENTS.md

## Project

mjswan packages browser-based MuJoCo simulations with real-time policy control into an interactive static web app. It has two sides:

- **Python** ([src/mjswan/](src/mjswan/)) — `Builder` / `Project` / `Scene` API that bundles models, policies, and UI config into a static site.
- **Frontend template** ([src/mjswan/template/](src/mjswan/template/)) — TypeScript + three.js + mujoco-wasm client that the Python build step bundles.

## Philosophy

- Write clean, readable, maintainable code.
- Don't reinvent what already exists upstream. Prefer mjlab ([GitHub](https://github.com/mujocolab/mjlab), [local](.venv/lib/python3.12/site-packages/mjlab)) or other dependencies over new boilerplate in mjswan.

## Layout

- [src/mjswan/](src/mjswan/) — package source (builder, project, scene, adapters, CLI, managers).
- [src/mjswan/template/](src/mjswan/template/) — frontend source (Vite + React + three.js + mujoco-wasm).
- [examples/](examples/) — `demo`, `mjlab`, `colab`, `tutorial` runnable examples.
- [tests/](tests/) — pytest suite. `slow`-marked tests are opt-out (see below).
- [docs/](docs/) — zensical site published to Read the Docs.

## Python workflow

Use `uv` instead of bare `python`/`pip`. Prefer the [Makefile](Makefile) targets.

Tests are configured with `slow` opt-out: `make test` runs everything, but pre-commit runs `pytest -m "not slow"` for speed.

---
> Source: [ttktjmt/mjswan](https://github.com/ttktjmt/mjswan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
