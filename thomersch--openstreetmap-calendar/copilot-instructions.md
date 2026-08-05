## openstreetmap-calendar

> This project uses [devbox](https://www.jetify.com/devbox). `devbox shell` is

# Development environment

This project uses [devbox](https://www.jetify.com/devbox). `devbox shell` is
interactive-only and `devbox run --` doesn't reliably forward arbitrary
commands, so for non-interactive use (agents, scripts), load the devbox
environment into the current shell first:

eval "$(devbox shellenv)" && ./manage.py test osmcal.test_views

This puts `.venv/bin`, GDAL/GEOS, and postgres on PATH for the rest of the
command.

# Linting

Run ruff before considering Python changes done:

eval "$(devbox shellenv)" && ruff check . && ruff format --check .

---
> Source: [thomersch/openstreetmap-calendar](https://github.com/thomersch/openstreetmap-calendar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
