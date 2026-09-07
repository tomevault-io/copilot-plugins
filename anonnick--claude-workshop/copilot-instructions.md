## claude-workshop

> Tiny Python package (`wxpost`) for WACCM-X post-processing.

# claude-workshop

Tiny Python package (`wxpost`) for WACCM-X post-processing.

## Build / test

Install into a **dedicated** Python 3.10+ environment — never a base or
system interpreter (a venv, or a named conda env on Derecho/Casper).

```bash
pip install -e .[dev]
pytest
```

## Conventions

- Python 3.10+, type hints where useful, no Fortran.
- Coord order in our internal arrays: `(time, lev, lat, lon)`.
- Pressure stored in **Pa** internally, even though WACCM-X files use hPa
  on `lev`. Convert at the I/O boundary.

## Data

`data/sample.nc` ships with the repo — open it from there. It is
**synthetic**: shaped like a real WACCM-X CAM `h0` monthly mean (96 levels,
49 lats, 24 lons, one January 2020 step) but generated from analytic
profiles by `data/make_sample.py`. Plausible, not real. Never cite a number
from it.

## Known-failing tests

Two tests are expected to fail on a fresh clone. They are the workshop
exercises. Don't fix them before the workshop.

- `tests/test_io.py::test_lat_orientation` — warm-up exercise.
- `tests/test_ops.py::test_to_height_thermosphere_telec` — main exercise.

---
> Source: [AnonNick/claude-workshop](https://github.com/AnonNick/claude-workshop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
