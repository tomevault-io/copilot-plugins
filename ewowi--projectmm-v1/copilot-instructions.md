## projectmm-v1

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development History

The project follows an Agile (Scrum-inspired) process adapted for AI-collaborative development. Development is organized into **Releases** (major milestones) containing **Sprints** (scope-boxed work cycles).

[docs/development/](docs/development/) is the development history section:

- [docs/development/index.md](docs/development/index.md) — development process, style, and the **Backlog** (items raised but explicitly deferred; each entry notes *when* it should be picked up).
- [docs/development/release-01.md](docs/development/release-01.md) — **Release 1: Proof of Concept** (complete). 10 sprints covering toolchain, module runtime, pixel pipeline, JSON UI, HTTP/WebSocket, and CI.

Each sprint has a **Scope** (goal + definition of done), a **Result** (test counts, timings, footprint), and a **Retrospective** (what was learned, what seeds the next sprint).

Read the latest release's final retrospective and the Backlog before starting any new work — they capture what is intentionally deferred and what the next cycle is expected to address.

---

## What This Project Is

A cross-platform module runtime for loop-driven processes (targeting ~20ms frame time). Modules run through a `setup()` / `loop()` / `teardown()` lifecycle. The primary domain is light control (LEDs via GPIO, Art-NET, DDP, E1.31), but the runtime is domain-agnostic. Targets: ESP32 (classic), Raspberry Pi, and PC (macOS/Linux/Windows) as peer builds from day one.

## First-time Setup (agentic harness)

This project uses uv for Python dependency management and pre-commit for local hooks. Run these once after cloning:

```sh
# Install uv (skip if already installed)
brew install uv        # macOS
# or: curl -LsSf https://astral.sh/uv/install.sh | sh

uv sync --extra dev    # installs ruff, pre-commit into the project venv
uv run pre-commit install  # wires clang-format + ruff into .git/hooks/pre-commit
```

After this, every `git commit` automatically checks C++ formatting (clang-format) and Python style (ruff). CI runs the same checks via `uv run pre-commit run --all-files`.

## Build Commands

All build, flash, test, and summary steps use the cross-platform Python scripts in `deploy/`.

**PC:**
```sh
uv run deploy/build.py -target pc     # CMake configure + build → deploy/build/pc/
uv run deploy/unittest.py             # run doctest suite → deploy/test/
cd deploy/build/pc && ./projectMM     # run server (state/ is relative to binary cwd)
```

**ESP32:**
```sh
uv run deploy/build.py -target esp32dev       # PlatformIO build
uv run deploy/flash.py                         # flash all test:true devices
uv run deploy/flashfs.py --wifi                # flash LittleFS + WiFi credentials
```

**Full pipeline (PC + all ESP32 devices):**
```sh
uv run deploy/all.py
```

CI runs on every push/PR (`.github/workflows/ci.yml`): PC CMake build + tests, ESP32 PlatformIO compile (no hardware flash in CI).

## Architecture

```
Frontend (HTML/CSS/JS)
  ↓ HTTP/WebSocket
Control Interfaces (REST, MQTT, WebSocket)
  ↓
Module Runtime (Core)
  ├─ Scheduler        — hot path: calls loop() on each module in sequence
  ├─ ModuleManager    — reads state/modulemanager.json, instantiates/wires/owns all modules
  ├─ TypeRegistry     — factory registry (type-name string → factory lambda)
  └─ StatefulModule   — base class with setProps/setInput/loadState/saveState
  ↓
Platform Abstraction (GPIO, network, filesystem)
  ↓
Hardware
```

**Data flow for lights:**
```
SineEffectModule → ProducerLayer (pixel buffer) → ConsumerLayer (blends) → PreviewModule / driver
```

**Key files:**
- `src/core/` — Scheduler, ModuleManager, TypeRegistry, Module, StatefulModule, Timing, MemoryStats, FileSystem
- `src/modules/` — Module implementations; `ModuleRegistrations.cpp` contains all `REGISTER_MODULE()` calls
- `src/modules/layers/` — EffectsLayer, DriverLayer, GridLayout, Channel, RGB
- `tests/test_*.cpp` — doctest cases, one file per feature area
- `state/modulemanager.json` — persistent module list: id, type, props, inputs, parent_id, core (auto-written on first structural change; gitignored)
- `state/<id>.json` — persisted controls per module (auto-written on teardown or control change; gitignored)
- `tests/test_modules.json` — test fixture: reference pipeline used by integration/HTTP/WS tests
- `docs/developer-guide/` — architecture.md (merged arch+design), api.md, deploy.md, standards.md, add-a-module.md
- `docs/user-guide/` — getting-started.md, ui.md, modules/index.md
- `docs/development/` — development history: process, backlog, release documents (release-01.md, ...)

## Module Development

**Required for every new Module:**
1. Subclass `StatefulModule`, implement `setup()` / `loop()` / `teardown()`
2. Default constructor only — no constructor arguments; config is injected before `setup()`
3. Override `setProps()`, `setInput()`, `loadState()`, `saveState()` as needed
4. Add `REGISTER_MODULE(TypeName)` in `src/modules/ModuleRegistrations.cpp`
5. Add at runtime via `POST /api/modules` or the frontend UI (persists to `state/modulemanager.json`)
6. Write tests in `tests/`
7. Add a doc page in `docs/modules/`

**Module registration macro** (must be in executable, not static lib):
```cpp
REGISTER_MODULE(MyModule)  // in ModuleRegistrations.cpp
```

**Runtime wiring pattern** (via REST):
```sh
curl -X POST http://localhost:80/api/modules \
  -H 'Content-Type: application/json' \
  -d '{"type":"SineEffectModule","id":"sine1","props":{"speed":1.5},"inputs":{"layer":"producer1"}}'
```

## Critical Constraints

- **No allocations in `loop()`** — allocate in `setup()`, free in `teardown()`
- **No blocking calls or info-level logging in `loop()`** — inflates measured frame timing
- **`teardown()` must be complete** — leaks break hot-reload; missing teardown is a bug
- **`healthReport()` required** — returns a string snapshot of module state; enables automated testing
- **Platform `#ifdef` belongs in platform abstraction layer**, not inside Module code

## Testing

Framework: doctest (single-header, C++17). Tests live in `tests/test_*.cpp`.

Run all tests: `uv run deploy/unittest.py` (or directly: `deploy/build/pc/tests/tests`)

Test rules from `standards-and-guidelines.md`:
- Non-trivial tests only — not compile-only; cover actual behavior (state transitions, output correctness, boundary conditions)
- Every Module change needs a test

## Coding Standards

- **C++17** on all platforms (ESP-IDF compatible)
- **Naming:** `CamelCase` (classes), `camelCase` (methods/vars), `UPPER_SNAKE` (constants/macros)
- **Comments:** explain *why*, not *what*; document classes and non-trivial functions
- **JSON schema keys are a public API** — renaming is a breaking change

**Writing style (docs, comments, commit messages):** follow the AI-specific rules in `docs/developer-guide/standards.md#ai-specific`. Key rule: no em dashes (`—`); use a comma, colon, or parentheses instead.

**Definition of done for a Module:** tests pass + doc page + JSON schema documented + footprint impact measured (`classSize()`)

## Key Design Decisions

See `docs/developer-guide/architecture.md` for rationale. Highlights:
- Modules are **default-constructible only** — dependency injection is data-driven via `setInput()`, not constructors
- `ModuleManager` owns all modules (`unique_ptr`); `Scheduler` only executes them
- `TypeRegistry` enables hot-reload and runtime instantiation without source modification
- `ProducerLayer` / `ConsumerLayer` decouple effects from output drivers via double-buffered pixel buffers
- The frontend is a generic JSON schema interpreter — no framework, no per-module HTML

---
> Source: [ewowi/projectMM-v1](https://github.com/ewowi/projectMM-v1) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-15 -->
