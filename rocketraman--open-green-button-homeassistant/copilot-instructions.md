## open-green-button-homeassistant

> This repo's toolchain is **mise-managed** (see `mise.toml`): Python 3.13 and **uv** are pinned,

# CLAUDE.md

## Python environment: ALWAYS use mise + uv (do not bypass them)

This repo's toolchain is **mise-managed** (see `mise.toml`): Python 3.13 and **uv** are pinned,
and mise creates the venv at `.venv/` using uv. Always go through mise; do **not** invoke
`.venv/bin/python`, a system `python`, or a bare `pytest`/`ruff` directly — those skip the pinned
toolchain and the activated venv.

### Clean-checkout setup (the correct sequence)

```sh
mise trust                                       # one-time, authorize this repo's mise.toml
mise install                                     # installs pinned Python 3.13 + uv, creates .venv via uv
mise exec -- uv pip install -r requirements_test.txt   # install test deps INTO .venv
```

### Day-to-day

```sh
mise exec -- python -m pytest        # tests
mise exec -- ruff check .            # lint
mise exec -- ruff format --check .   # format check
```

### Installing/adding dependencies — read this, it bites every time

The mise-created venv is a **bare uv venv with NO `pip` module**. So:

- ✅ Install with **`mise exec -- uv pip install ...`** (uv targets `.venv` via `VIRTUAL_ENV`).
- ❌ Do **not** run a bare `pip` — `mise exec -- pip` silently resolves to the *base* mise
  Python's pip and installs where `mise exec -- python` (the venv) can't see it. That's the
  "I installed it but `No module named pytest`" trap.
- ❌ Do **not** run `mise exec -- python -m pip ...` against a fresh venv — it has no pip
  (`No module named pip`). Only works after `mise exec -- python -m ensurepip`, which you should
  not need to do — just use `uv pip`.
- Always pair `python` and its installer for the *same* interpreter: `uv pip` (preferred) or, if
  you truly must use pip, `python -m ensurepip` first.

### uv pin gotcha (already handled, don't undo it)

In `[tools]`, **`uv` is listed before `python`** on purpose: the Python install ships a stray
`uv` binary that otherwise shadows the pinned uv on PATH. Also, if a `.venv` was created by an
*older* uv, it may contain a stale `.venv/bin/uv` that wins on PATH — if `mise exec -- uv
--version` disagrees with the pin, `rm -rf .venv && mise install` recreates it cleanly with the
pinned uv.

## What this repo is

Home Assistant **custom integration** (`custom_components/greenbutton/`) that pulls Green Button /
ESPI usage data from the Open Green Button **proxy server** and writes it into HA long-term
statistics. It owns no entities — it only appends to the Energy dashboard's statistics.

The proxy server is a **separate repo**: `../open-green-button` (Kotlin/Ktor, deployed on Fly as
app `open-green-button`). Anything about per-utility config, OAuth, the `/proxy/usage` /
`/claim` / `/utilities` endpoints, or `utilities.conf` lives there, not here.

- Server build/test (from `../open-green-button/server/`) uses **mise too** (JDK 21 + Gradle
  8.14.1); there is no `gradlew` wrapper — use the mise-provided `gradle`.

## Conventions worth keeping

- The **backfill window is configured server-side**, per-utility (`initialHistory` in the proxy's
  `utilities.conf`, e.g. `2y`). The client receives it in the claim response as
  `CONF_INITIAL_HISTORY_SECONDS` and only falls back to `INITIAL_FETCH_LOOKBACK` when absent. Do
  not reintroduce a hard-coded client-side history window as the source of truth.

---
> Source: [rocketraman/open-green-button-homeassistant](https://github.com/rocketraman/open-green-button-homeassistant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
