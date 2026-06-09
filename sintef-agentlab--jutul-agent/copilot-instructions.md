## jutul-agent

> jutul-agent is a Deep Agents–based assistant for Julia scientific simulation

# jutul-agent

## Purpose

jutul-agent is a Deep Agents–based assistant for Julia scientific simulation
work on the Jutul ecosystem.

## Working rules

- Prefer the live Julia session and simulator source code over model memory.
- Use the persistent Julia REPL incrementally. Run the smallest useful probe
  first, inspect the result, then extend.
- Treat the installed simulator packages as the source of truth for APIs and
  examples.
- Keep Julia execution serialized unless the runtime explicitly supports
  concurrency.
- Prefer Deep Agents memory and skills for reusable guidance; keep custom
  Python focused on Julia execution, simulator inspection, and trace capture.
- Workspace memory layout and stock Deep Agents wiring:
  [docs/design/memory.md](docs/design/memory.md).

## Validation

- Default suite (no Julia): `uv run pytest -q -m "not integration"`
- Integration suite (requires Julia + an instantiated env): `uv run pytest -q -m integration`.
  Envs instantiate from `Project.toml` + `[sources]`.
  `tests/test_simulators_smoke.py` loads each simulator with `using <Sim>`.
- Live LLM (humans only, needs API key): `uv run pytest tests/live/`
- Update transcript / report snapshots after renderer changes:
  `uv run pytest --snapshot-update tests/test_transcript_html.py tests/test_transcript.py tests/test_report_renderer.py`
- Python style (matches CI): `uv run ruff check .` then `uv run ruff format .`
- After editing Python, run `uv run ruff format` on touched paths (or rely on
  format-on-save / pre-commit if installed).

See [docs/testing.md](docs/testing.md) for the full testing guide.

## Continuous integration

Two GitHub Actions workflows:

- `ci.yml` — every push/PR. `lint` (ruff); `test` across Linux/Windows/macOS
  (`pytest -m "not integration"`, no Julia); `julia-integration` (Linux only:
  runs `test_juliakernel.py` against base Julia — the kernel needs no env).
- `simulators.yml` — per-simulator smoke matrix (`jutuldarcy`, `battmo`,
  `fimbul`, `mocca`; Linux). On PR + push to `main`, a weekly Monday run
  (catches upstream breakage), and manual dispatch. Each job instantiates the
  env from `Project.toml` and runs `test_simulators_smoke.py`.

## Local git hooks

One-time per clone: `uv run pre-commit install`. Commits then run Ruff format
and check on staged `.py` files. To verify the whole tree like CI:
`uv run pre-commit run --all-files`.

## Important paths

- `src/jutul_agent/paths.py` — the three runtime anchors: `PACKAGE_ROOT`
  (install), `workspace_root()` (CWD), `state_home()` (sessions).
- `src/jutul_agent/workspace.py` — workspace config
  (`.jutul-agent/config.toml`), simulator auto-detect, Julia-env bootstrap.
- `src/jutul_agent/session.py` — `Session`, the unit of work for one
  invocation.
- `src/jutul_agent/agent/builder.py` — deepagents wiring: composite backend,
  HarnessProfile registration, `build_agent` entry point. Custom tools sit
  alongside (`tools.py`, `julia_plot.py` + `julia_plot.jl`, `memory.py`,
  `approval.py`, `turns.py`, `mounts.py` — the `/add-dir` route mounting).
- `src/jutul_agent/simulators/` — adapter dataclass, registry, env bootstrap,
  shared skills, and one folder per simulator (see below).
- `src/jutul_agent/julia/` — the `JuliaSession` Protocol (`session.py`) and the
  Julia toolchain checks (`requirements.py`).
- `src/jutul_agent/juliakernel/` — the backend: a self-contained, supervised
  Julia runtime (`kernel.py` + stdlib-only `server.jl`) with live output and
  SIGINT interrupt. Splittable into a standalone package.
- `src/jutul_agent/trace/` — SQLite event log and `TraceRecorder` middleware.
- `src/jutul_agent/transcript/` — renderers that consume a trace
  (HTML transcript, markdown transcript, investigation report).
- `src/jutul_agent/interfaces/tui/approval.py` — HITL decision policy and the
  approval card markdown.
- `src/jutul_agent/interfaces/` — `cli` and the Textual `tui` package.
- `tests/` — end-to-end, chat, CLI, and tool coverage. `tests/integration/`
  needs Julia; `tests/live/` needs an LLM provider key.

## Per-simulator layout

Each simulator lives under `src/jutul_agent/simulators/<name>/` and owns
exactly three things:

- `adapter.py` — constructs the `SimulatorAdapter`. Set
  `module_dir = Path(__file__).resolve().parent` so the base class can
  derive `julia_env_template_path`, `skills_dir`, and `plot_helpers_path`.
- `julia_env/` — `Project.toml` declaring the deps the agent can `using`, plus a
  per-simulator `JutulAgent<Sim>/` warm package (a local `[sources]` path dep whose
  `@recompile_invalidations` + `@compile_workload` bakes that simulator's
  GLMakie-aware solve/plot into the precompile cache, so the first solve is fast).
  The shared, sim-agnostic `JutulAgent` package (figure capture, ensemble helpers,
  generic-Makie warm-up) has a single source in `src/jutul_agent/julia_runtime/` and
  is copied into the env at bootstrap (also a relative `[sources]` dep). The
  `Manifest.toml` is generated on instantiate (gitignored). The whole `julia_env/`
  folder is copied into a workspace on bootstrap, so the relative `[sources]` paths
  keep resolving.
- `skills/<skill-name>/SKILL.md` — markdown skills surfaced via the
  deep-agents skill system.

Add a simulator by creating that folder and one entry in
`simulators/registry.py`. Custom subagent factories, when they land, go on
the adapter's `subagent_factories` tuple.

---
> Source: [SINTEF-agentlab/jutul-agent](https://github.com/SINTEF-agentlab/jutul-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-09 -->
