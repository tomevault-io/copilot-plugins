## claude-md-compiler

> `cldc` compiles repository policy from `CLAUDE.md` and related YAML files into `.claude/policy.lock.json`, then enforces that policy over explicit runtime evidence, Claude Code hook session state, and git-derived diffs.

# claude-md-compiler Agent Context

## Project Identity

`cldc` compiles repository policy from `CLAUDE.md` and related YAML files into `.claude/policy.lock.json`, then enforces that policy over explicit runtime evidence, Claude Code hook session state, and git-derived diffs.

## Tech Stack

- Language: Python 3.11+
- Packaging: `uv` with `uv_build`
- Runtime dependencies: `PyYAML`, `textual`
- Test runner: `pytest` (+ `pytest-asyncio`, `hypothesis`, `pytest-cov`, `pytest-benchmark`)
- Entry point: `cldc`

## Repository Map

- `src/cldc/__init__.py`: typed exception re-exports and the package version resolver.
- `src/cldc/_logging.py`: library logging setup (`NullHandler` by default, CLI stderr handler on demand).
- `src/cldc/errors.py`: typed exception hierarchy rooted at `CldcError(ValueError)`.
- `src/cldc/cli/main.py`: argparse CLI, command routing, exit codes, JSON/text output.
- `src/cldc/ingest/discovery.py`: repo-root discovery and source inventory.
- `src/cldc/ingest/source_loader.py`: source loading, inline `cldc` block extraction, include-pattern validation, and `extends:` preset resolution.
- `src/cldc/parser/rule_parser.py`: rule validation and normalized policy model.
- `src/cldc/compiler/policy_compiler.py`: lockfile generation and `doctor` diagnostics.
- `src/cldc/presets/`: bundled opinionated policy packs (`default`, `strict`, `docs-sync`) and the preset loader API.
- `src/cldc/runtime/evaluator.py`: policy evaluation against reads, writes, commands, command outcomes, claims, and git-derived write sets.
- `src/cldc/runtime/events.py`: machine-readable runtime evidence ingestion.
- `src/cldc/runtime/reporting.py`: saved report validation and rendering.
- `src/cldc/runtime/remediation.py`: deterministic fix-plan generation and rendering.
- `src/cldc/runtime/git.py`: staged and base/head diff collection.
- `src/cldc/runtime/hooks.py`: hook artifact generation for git and Claude Code integration.
- `src/cldc/runtime/claude_code_adapter.py`: stateful Claude Code session adapter used by generated hooks.
- `src/cldc/runtime/report_schema.py`: versioned `policy-report/v1` constants.
- `src/cldc/scaffold.py`: `cldc init` scaffolding for a fresh repo.
- `src/cldc/tui/`: Textual-based interactive TUI (`cldc tui`). `app.py` hosts `CldcApp`, `state.py` owns the `TuiState` dataclass + loaders, `widgets.py` defines the custom panes, and `styles.tcss` is the dark theme stylesheet.
- `tests/fixtures/repo_a/`: canonical fixture repo used by compile/runtime/CLI tests.
- `tests/e2e/`: narrated langchain demo runner plus the raw pytest e2e suite (`make e2e` / `make e2e-test`).
- `docs/library-usage.md`: full library reference for embedders.
- `docs/rfcs/`: frozen implementation contracts.

## Build, Test, and Run

```bash
uv sync --locked
uv run pytest -q
uv build
uv run cldc --help
uv run cldc init /path/to/new/repo --preset default
uv run cldc compile tests/fixtures/repo_a
uv run cldc check tests/fixtures/repo_a --write src/main.py --json
uv run cldc ci tests/fixtures/repo_a --base HEAD --head HEAD --json
uv run cldc explain tests/fixtures/repo_a --write src/main.py --format markdown
uv run cldc fix tests/fixtures/repo_a --write src/main.py --json
uv run cldc preset list
uv run cldc preset show default
uv run cldc tui tests/fixtures/repo_a
uv run cldc hook generate claude-code --json
uv run cldc hook claim . ci-green
```

## Conventions

- Use explicit `encoding="utf-8"` for repository file I/O.
- Keep JSON artifacts deterministic: sorted keys, stable ordering, explicit schema/version fields.
- Fail closed on malformed or tampered artifacts; do not silently ignore unsupported rule kinds.
- Keep the CLI shell thin. Put behavior in ingest/parser/compiler/runtime modules, then expose it in `cli/main.py`.
- Add or update tests with every behavior change, especially for stale lockfiles, malformed inputs, and boundary cases.

## Critical Constraints

- Do not change `docs/rfcs/` unless the specification itself is changing.
- Treat `.claude/policy.lock.json`, policy report JSON, and fix-plan JSON as versioned contracts.
- `cldc check` must refuse stale or schema-drifted lockfiles instead of guessing.
- Rule kinds that are not supported by the runtime must raise explicit errors, not degrade to a silent pass.
- Keep include patterns repo-local; do not allow config globs to escape the repo root.

## Gotchas

- This repository has a top-level `AGENTS.md` for agent context, but it intentionally contains no `cldc` rules.
- `require_claim` is a path-scoped rule: writes matching `when_paths` fail the rule unless at least one of the listed `claims` is asserted via `--claim`, an events file, stdin JSON, or an event payload.
- `couple_change` means a write matching `paths` requires an additional write matching `when_paths`; the same path does not satisfy both sides of the rule.
- `compile` updates the lockfile state in its returned metadata; missing-lockfile warnings from discovery should not survive a successful compile.

## Current State

As of April 8, 2026, the shipped CLI surface is `init`, `compile`, `doctor`, `check`, `ci`, `explain`, `fix`, `preset`, `tui`, and `hook`.

Implemented:

- deterministic source discovery and lockfile generation
- `cldc init` onboarding scaffolder (`.claude-compiler.yaml` + stub `CLAUDE.md`, repeatable `--preset`, safe by default)
- doctor diagnostics for malformed, stale, and drifted artifacts
- runtime enforcement for `deny_write`, `require_read`, `require_command`, `require_command_success`, `forbid_command`, `couple_change`, and `require_claim`
- claim ingestion via `--claim`, events file, stdin JSON, and event payloads
- bundled preset policy packs (`default`, `strict`, `docs-sync`) reachable via `extends:` in `.claude-compiler.yaml` and inspectable with `cldc preset list`/`cldc preset show`
- saved policy report rendering and deterministic fix-plan generation
- git-aware CI entrypoints for staged and base/head diffs
- stateful Claude Code hook adapter with `SessionStart`, `PreToolUse`, `PostToolUse`, `PostToolUseFailure`, `Stop`, and `SessionEnd`
- explicit Claude Code session claims via `cldc hook claim`
- git pre-commit and Claude Code hook generation via `cldc hook`
- `cldc tui` interactive policy explorer

Known limitations:

- no semantic extraction of free-form prose into rules yet; only structured policy is compiled today
- no automatic repo mutation or autofix execution
- no separate lint/type/coverage enforcement in CI yet

---
> Source: [AbdelStark/claude-md-compiler](https://github.com/AbdelStark/claude-md-compiler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
