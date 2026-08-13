## clearagent

> ClearAgent is an installable Python 3.14 library. Use `uv`, preserve the

# Agent Instructions

## Fast Orientation

ClearAgent is an installable Python 3.14 library. Use `uv`, preserve the
local-first SQLite default, and keep the default test path offline and
deterministic.

Bootstrap a checkout without changing the lockfile:

```bash
uv sync --locked --all-extras --dev
```

While iterating, run the smallest checks that cover the change:

```bash
uv run pytest tests/unit/test_tool_schema.py
uv run pytest tests/integration/test_agent_tracing.py
uv run ruff check src/clearagent/tool.py tests/unit/test_tool_schema.py
uv run python -m mypy src
uv run python scripts/check_docs_links.py
```

Before handoff, run the full offline gate when practical:

```bash
./scripts/check.sh
```

The gate runs offline tests with at least 90% package line coverage, Ruff lint
and formatting checks, mypy, and the documentation checker. Run `uv build` as
an additional check for package metadata, bundled files, entry points, or
release workflow changes.

## Repository Map

Use this map to find the implementation, its focused verification, and the
reader-facing docs that normally move with it.

| Task | Implementation | Focused tests | Public docs |
| --- | --- | --- | --- |
| Agent runtime, tools, messages, and results | `src/clearagent/agent.py`, `create.py`, `tool.py`, `messages.py`, `types.py` | `tests/unit/test_create_agent.py`, `test_tool_schema.py`, `test_structured_output.py`; `tests/integration/test_agent_tracing.py` | `docs/getting-started.md`, `core-concepts.md`, `reference.md` |
| Provider adapters and model URIs | `src/clearagent/providers/` | `tests/unit/test_*provider.py`, `test_model_uri.py`, `test_native_providers.py`, `test_live_provider_compatibility.py` | `docs/providers.md`, `live-provider-compatibility.md`, `reference.md`, `status.md` |
| Trace storage, replay, and reports | `src/clearagent/storage/`, `trace_lifecycle.py`, `replay.py`, `reports.py` | `tests/unit/test_sqlite_trace_store.py`, `test_replay.py`, `test_reports.py`; `tests/integration/test_trace_cli.py`, `test_custom_trace_store.py` | `docs/tracing.md`, `database.md`, `architecture.md`, `reference.md` |
| Eval suites, checks, baselines, and Promptfoo | `src/clearagent/evals/` | `tests/unit/test_eval_*.py`, `test_promptfoo_export.py`; `tests/integration/test_eval_*.py`, `test_baselines.py` | `docs/evals.md`, `pytest.md`, `promptfoo.md`, `reference.md` |
| Graph execution | `src/clearagent/graph/` | `tests/integration/test_agent_graph.py` | `docs/core-concepts.md`, `flows.md`, `architecture.md`, `reference.md` |
| Chat backend and browser assets | `src/clearagent/chat/` | `tests/unit/test_chat_store.py`; `tests/integration/test_chat_app.py` | `docs/chat.md`, `database.md`, `flows.md`, `reference.md` |
| CLI and project config | `src/clearagent/cli.py`, `config.py` | `tests/unit/test_config.py`; `tests/integration/test_cli_smoke.py`, `test_cli_json.py`, `test_trace_cli.py`, `test_baselines.py` | `docs/reference.md` and the relevant workflow guide |
| Pytest integration | `src/clearagent/pytest_plugin/` | `tests/integration/test_pytest_integration.py` | `docs/pytest.md`, `core-concepts.md`, `reference.md` |
| Packaging, examples, and docs | `pyproject.toml`, `examples/`, `scripts/`, `.github/workflows/`, `README.md`, `docs/` | `tests/unit/test_public_api.py`, `test_docs_links.py`; `tests/integration/test_examples.py`; `uv build` | `docs/install.md`, `publishing.md`, `deployment.md`, `contributing-docs.md` |

## Core Invariants

- A provider-shaped request is built and redacted before it is persisted, and
  it is persisted before the provider call. Preserve this order so failures can
  still be inspected and requests can be replayed exactly.
- `TraceStore` is the runtime, eval, trace-check, graph, and agent-backed chat
  persistence boundary. An injected store must not be silently replaced by a
  newly opened SQLite store. Standalone file-oriented CLI inspection remains
  explicitly SQLite-backed through `--trace-db`.
- SQLite is the default, not an assumption for injected-store code. Keep trace
  and chat persistence separate.
- Agent tool loops are bounded by `max_turns`; graph runs are linear, reject
  cycles and unknown targets, and are bounded by `max_nodes`.
- The normal test gate never calls a live model. The bounded compatibility
  runner requires `CLEARAGENT_LIVE_TESTS=1` and the relevant provider
  credential; default tests consume sanitized recordings.
- Public examples import authoring helpers from `clearagent`, provider values
  from `clearagent.providers`, storage values from `clearagent.storage`, and
  other documented package entry points. Treat deeper implementation modules
  as internal unless `docs/reference.md` says otherwise.
- Chat binds to loopback by default. Runtime settings mutation stays disabled
  unless the caller explicitly enables it.

## Files And Commands That Write

Hand-authored source includes `src/`, `tests/`, `examples/`, curated Markdown,
browser assets, project metadata, the lockfile, scripts, and workflows. Treat
these outputs as generated until a person deliberately reviews and promotes
them:

- `.clearagent/*.sqlite`, plus SQLite `-wal` and `-shm` sidecars
- `.clearagent/promptfoo_target.py`, exported Promptfoo configs, replayed
  request JSON, generated eval YAML, and generated trace reports
- `dist/`, `.coverage`, HTML coverage, and tool caches

`.clearagent/config.toml` is a special case: `clearagent init` creates it as a
starter project configuration. It may be committed when shared tracing settings
are intentional, but it is not required for contributor bootstrap.

Know the write boundary before running these commands:

- `clearagent init` creates `.clearagent/config.toml` if it does not exist.
- Agent runs, evals, and chat can create SQLite data at the configured paths.
- `trace-to-eval`, `trace-report`, `replay-request`, and Promptfoo export/target
  commands write their explicit output paths.
- The live compatibility runner writes checked-in fixtures only when passed
  its explicit `--record` flag.
- `uv build` writes `dist/`; the full check may update tool caches.

For tests and exploratory runs, use `tmp_path`, a temporary directory, or
explicit temporary database/output paths. Check `git status --short` before and
after broad commands. Remove only artifacts created by your own work; never
discard an existing untracked file merely because it looks generated.

## Live-Test Safety

Do not opt into live tests merely because a credential is present. Run one only
when the task explicitly needs network-backed verification, and target it
directly:

```bash
CLEARAGENT_LIVE_TESTS=1 uv run python scripts/live_provider_compatibility.py --provider openrouter
```

Set `OPENROUTER_API_KEY` in the environment separately. Never print, document,
or commit its value. Follow `docs/live-provider-compatibility.md` for request
caps and fixture review. Live-model output is not a substitute for
deterministic coverage.

## Documentation

ClearAgent is an open source project, so behavior changes must keep the
reader-facing docs current.

When changing public behavior, update documentation in the same change. This
includes changes to:

- public Python APIs such as `create_agent`, `@tool`, providers, evals, tracing,
  graph flows, chat, pytest helpers, or structured outputs
- CLI commands, flags, output shape, or config files
- example agents, eval suites, provider setup, or trace storage behavior
- contributor workflows, test commands, or project structure

Use curated Markdown as the source of truth for public docs. Do not dump
docstrings into the docs as a substitute for explaining concepts and workflows.
Docstrings can inform reference material, but docs should be written for people
learning the repo.

Before finishing a change, check whether `docs/site.md` still points readers to
the right pages. If a new concept or workflow is added, either update an
existing page or add a focused page and link it from `docs/site.md`.

## Testing And Main-Branch Safety

Treat `main` as releasable. Every change requires verification proportionate to
what it can break. Changes that can affect runtime behavior, persistence,
provider wire formats, public APIs, CLI or HTTP output, browser assets,
examples, packaging, installation, or documented commands must include
automated tests that exercise the changed contract. A focused test passing is
not completion; the full repository gate must pass.

For every changed behavior:

- add or update a test that would detect the previous or broken behavior
- cover the successful path and each relevant failure, rejection, boundary,
  and persisted-state path introduced or changed
- add a regression test that reproduces the exact failure for every bug fix
- assert observable contracts such as return values, exceptions, exit status,
  HTTP payloads, provider request bodies, traces, database rows, files, or
  rendered interactions; merely asserting that a helper was called or that
  source text exists is not enough
- preserve and test existing public behavior, including malformed, legacy, and
  backward-compatible inputs when applicable
- put isolated logic tests in `tests/unit/` and cross-component behavior in
  `tests/integration/`; use temporary directories and deterministic fake or
  mocked providers

Provider changes require credential-free tests for the applicable exact
request shape, response and usage parsing, tool and structured-output round
trips, streaming, non-success responses, malformed payloads, and normalized
errors. Sanitized live recordings may supplement those tests. Paid live tests
must remain explicitly opt-in, bounded, and secret-safe; they never replace
required offline CI coverage.

Changes to `src/clearagent/chat/static/` must be exercised by an executable
browser or DOM test that covers the changed interaction and its failure state.
Serving an asset or searching its source for function names is not sufficient.

Changes to package metadata, dependencies, package data, static assets, public
imports, or entry points must build both the sdist and wheel and test the built
wheel from a temporary environment outside the repository. The smoke test must
verify public imports, `clearagent --help`, bundled chat assets, and a fake
provider run that writes a SQLite trace.

Documentation changes must keep public commands and examples executable and
must pass the documentation checker. Public behavior changes require matching
reader-facing documentation in the same change.

Tests must be deterministic and must not depend on real credentials, external
network access, user home-directory state, wall-clock timing, or test order
unless those dependencies are explicitly isolated. Do not delete or weaken a
test, lower a coverage threshold, add a skip, xfail, deselection, collection
override, suppression comment, or analysis ignore merely to make a change pass.
Any justified replacement must retain or strengthen the same contract coverage.

Changes to CI, test configuration, fixtures, or `scripts/check.sh` must include
evidence that the gate still rejects a deliberately failing case; a successful
run alone does not prove that a gate works.

Before finishing any change, run:

```bash
uv run bash scripts/check.sh
```

Focused checks are useful during development but never replace the full gate.
If the full gate cannot run or a required behavior cannot be tested, report the
specific blocker and do not claim the change is safe to merge.

## Library Consumer Experience

Treat ClearAgent as an installable library, not only as this repository. Public
docs should make the external-project path obvious:

- show how to install the package as a dependency before showing contributor
  setup commands
- keep examples copy-pastable in a fresh project, with imports from
  `clearagent`
- label repo-only commands such as `uv sync --locked --all-extras --dev` as
  contributor setup
- document provider API keys, optional extras, local runtime files, and the
  expected Python version near the first install instructions
- keep `README.md` useful on PyPI by avoiding docs links that only work from a
  checked-out repository
- keep `docs/reference.md` aligned with the public API and CLI surfaces

When changing package metadata, bundled files, entry points, optional
dependencies, or release workflow, update the installation/publishing docs in
the same change and verify the package can build.

## Packaging And Release Readiness

Before claiming the project is ready to publish or consume as a package:

- run `uv build` and confirm both sdist and wheel are produced
- inspect the wheel when package data changes, especially chat static assets
- run `./scripts/check.sh` and require it to pass
- keep `pyproject.toml` project URLs valid for PyPI
- keep release instructions token-safe; never document or commit real publish
  tokens

## Project Conventions

- Use `uv` for setup, tests, and command examples.
- Target Python 3.14.
- Keep tracing local-first and SQLite-backed unless a change explicitly alters
  that design.
- Avoid documenting commands that are not implemented in this repo.
- Prefer runnable examples backed by `examples/` or tests.
- Run `./scripts/check.sh` and require it to pass before declaring a
  change complete or safe to merge.

---
> Source: [kyle-mirich/clearagent](https://github.com/kyle-mirich/clearagent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
