## absentia

> A code-hygiene tool that mines patterns a codebase already follows and surfaces

# CLAUDE.md — guidance for AI assistants in this repo

## What absentia is

A code-hygiene tool that mines patterns a codebase already follows and surfaces
places that don't follow them. **No LLM in the engine** — purely classical
(tree-sitter + frequent itemset mining + statistics). User-facing tagline:
*"Find the holes your code already drew."*

See `README.md` for the public-facing pitch.

## Architecture in one screen

Four-layer storage model:

```
User layer       suppressions, annotations, config       sticky across runs
Pattern layer    groups, rules, gaps                     derived per run
Entity layer     entities, features, relations           incremental on file change
Raw layer        files, content hashes, parse cache      incremental on disk change
```

Two consumers of the engine library:

- **TUI** — `absentia` (default invocation), Textual-based, primary UX
- **Batch CLI** — `absentia check` for CI, scripts, editor integrations

A future third consumer: a Dev-Dashboard panel that imports the engine as a
Python library or shells out to `absentia check --json`.

## Locked-in decisions

These were debated and chosen deliberately. Don't reverse them silently — if
a reversal seems warranted, surface the reasoning explicitly.

1. **No LLM in the engine.** Determinism, free explanations as a byproduct of
   rule mining, sub-second feedback, and differentiation from saturated AI
   tooling. Embeddings are reserved for an eventual personal-knowledge variant
   if it materializes; LLM only as an optional natural-language query shell,
   never the core.
2. **TUI is the primary UX.** Built with Textual. Batch CLI is the secondary
   scriptable mode. Number keys for view switching; lowercase for actions.
3. **Standalone repo, not a host-app panel.** Designed for any host
   (a developer dashboard, an editor extension, a CI bot) to embed
   via library import or `--json` shellout. The engine has zero
   coupling to the host.
4. **Python + SQLite for the MVP.** Rust + alternate stores reserved for the
   enterprise tier. Don't pre-build that infrastructure.
5. **Stable IDs everywhere.** Entities, rules, gaps, and groups all have
   deterministic IDs derived from their identity (not sequence numbers), so
   suppressions persist across rebuilds.
6. **Architectural seams designed for worst-case; implementation built for
   current case.** Storage interface, extractor plugin shape, group selector
   polymorphism, content-hash-driven incremental — all baked in from day 1.
   Rust port, columnar store, parallelism, etc. are deferred until needed.

## Repo layout

```
src/absentia/        Engine package
  cli.py           Argparse dispatch + subcommand bodies
  tui/             Textual TUI (interactive front-end)
  extractors/      Per-language tree-sitter extractors (17)
  selectors.py     directory / decorator / parent_class group builders
  mining.py        Frequency mining over (group, feature) pairs
  symmetry.py      Symmetry-pair + call-pair detection
  series.py        Numeric-series-gap detection
  enrichment.py    Corpus-level features (sibling_test)
  storage.py       SQLite-backed entity + suppression store
  parallel.py      ProcessPool helpers, default_jobs() policy
  estimator.py     Cold-scan cost model + `absentia est` renderer
  calibration.py   First-run calibration + ~/.absentia/calibration.json
  runs_log.py      Machine-wide ~/.absentia/runs.jsonl accumulator
  settings.py      ~/.absentia/settings.json (jobs_default, etc.)
  progress.py      ProgressBar / Spinner / StepIndicator
  output.py        Gap rendering (text + JSON)
  _color.py        ANSI escape constants for in-place progress UI
  _console.py      rich Console proxy (pytest-capsys-compatible)
tests/             Unit + integration tests
scripts/           Maintenance + diagnostic scripts
  update_ts.py     Tree-sitter grammar version sweep
  scan_remote.py   Sanity-check against public corpora
  diagnose_scan.py Per-stage scan timing for cross-machine debugging
  profile_scan.py  cProfile harness for opt-pass hotspot validation
  release.sh       Interactive bump + tag + push (triggers release-checks.yml)
  local_ci.sh      Mirror of CI checks (ruff + mypy + pytest+cov + mkdocs --strict)
docs/              Mkdocs-material site (Diátaxis structure)
  tutorial/        Learn-by-doing
  how-to/          Task-oriented recipes
  reference/       Look-up authoritative
  explanation/     Concepts + ADRs (in decisions/)
pyproject.toml     Package config
absentia.toml.example   Sample per-project config
README.md          Public-facing pitch
DEFERRALS.md       Publication-blocking items intentionally deferred
CHANGELOG.md       Per-release notes
```

In **user projects** (not this repo):
- `absentia.toml` — committed per-project config + project-wide suppressions
- `.absentia/` — gitignored runtime state directory (entity DB, parse cache, etc.)

## Dev scripts

`scripts/update_ts.py` discovers every installed `tree-sitter*` package and
checks pip for updates. Run periodically as new absentia extractors are added —
keeps grammars current without hardcoding the list.

Three modes:

- **Interactive** (default, when run from a TTY): numbered list + menu.
  Choices: 1) apply outdated · 2) apply all · 3) apply specific packages
  by number (e.g. `3 2,4`) · q) quit.
- **Non-interactive apply** (`--apply`, `--apply --all`): runs the upgrade
  without prompting, suitable for CI / cron.
- **Non-interactive info** (`--dry-run`, or non-TTY stdin): prints the
  status and exits.

```bash
python scripts/update_ts.py            # interactive
python scripts/update_ts.py --apply    # upgrade outdated, no prompt
python scripts/update_ts.py --apply --all   # upgrade everything, no prompt
python scripts/update_ts.py --dry-run  # print status only
```

The script is *deliberately* dynamic: it discovers tree-sitter packages by
name prefix rather than reading from a hardcoded list or `pyproject.toml`.
Adding a new language means installing its grammar; the script picks it up
on the next run.

### `scripts/scan_remote.py` — sanity-check against real codebases

Clones a public repo into a temp dir, runs `absentia check`, and cleans up.
The default is `--depth 1` shallow clone, so even large repos use modest
disk space. Use it to verify a freshly added extractor actually works on
real-world code.

```bash
python scripts/scan_remote.py --list                       # show known corpora
python scripts/scan_remote.py --language python            # pick a Python corpus
python scripts/scan_remote.py URL                          # scan an arbitrary URL
python scripts/scan_remote.py URL --keep                   # leave the clone in place
python scripts/scan_remote.py URL --languages python,go    # restrict the scan
```

**Convention: when adding a new language extractor, add at least one entry
to `KNOWN_CORPORA` in `scripts/scan_remote.py`.** Pick a public repo that's
idiomatic for the language, small-to-medium sized, convention-rich, and
well-maintained. The `KNOWN_CORPORA` dict is *the* sanity-check resource —
if a language ships without an entry, we have no quick way to verify the
extractor works on real code as the codebase evolves.

### `scripts/release.sh` — interactive release CLI

Bumps `pyproject.toml`, promotes the CHANGELOG's `[Unreleased]` heading
to a versioned `[X.Y.Z] - YYYY-MM-DD`, commits, annotated-tags, and
pushes — which triggers the heavy CI gates in `release-checks.yml`
(test matrix · mypy · mkdocs --strict) plus the wheels build in
`wheels.yml`. Mirrors the Dev-Dashboard `build.sh` style.

```bash
bash scripts/release.sh                # interactive: validate / patch / minor / major / set
bash scripts/release.sh --patch        # 0.1.0 → 0.1.1
bash scripts/release.sh --minor        # 0.1.0 → 0.2.0
bash scripts/release.sh --major        # 0.1.0 → 1.0.0
bash scripts/release.sh --set=1.0.0    # explicit
bash scripts/release.sh --validate     # dispatch release-checks.yml on current branch (no bump)
```

`--validate` requires the GitHub CLI (`gh`) authenticated. Each step
of a real release (commit / tag / push / tag-push) rolls back on
failure. See `CONTRIBUTING.md` §9 for the CI split rationale (cheap
gates on every push, heavy gates on tag).

## Conventions

The full project-wide convention list lives in
[`CONTRIBUTING.md`](CONTRIBUTING.md) at the repo root. Read it before
making non-trivial code changes. Topics covered there:

- Progress UI (`ProgressBar` / `StepIndicator` / `Spinner`) — when to
  use which; never let an operation feel hung.
- Commit message format (subject ≤72, `Authored-by:` trailer required,
  enforced by `.githooks/commit-msg`).
- Destructive operations (disclaimer + `[y/N]` default-no + non-TTY
  refusal + `--yes` bypass).
- Local CI before commit (`pytest && ruff && mypy && mkdocs --strict`).
- Editable install for development.
- Doc-with-feature — which docs to update for which kind of change.
- Release flow — when to run `scripts/release.sh`, what the CI split
  (cheap on push, heavy on tag) means for which gate catches what.
- Defensive UI hooks (progress callbacks must never break the work).
- Stable IDs across runs (suppressions and external integrations
  depend on it).

**When to update `CONTRIBUTING.md`:** when a new project-wide
convention emerges — typically during code review or while shipping
a feature whose pattern future contributors should follow. Add a
new section in the same commit that establishes the convention.

**When to read `CONTRIBUTING.md`:** before adding code that touches
shared infrastructure (CLI surface, progress UI, destructive flags,
ID generation, etc.) or before introducing patterns other
contributors will copy.

A few project-level rules that aren't in `CONTRIBUTING.md`:

- Docs live in this repo. PRs that need docs are caught in review.
- ADRs go in `docs/explanation/decisions/` and are written **when
  the decision is made**, not retroactively.
- Tutorial code blocks must be runnable and tested in CI.
- Deferred publication-blockers go in `DEFERRALS.md`.
- Resolved deferrals get struck through (not deleted) and moved to
  the Resolved section at the bottom of `DEFERRALS.md`.

## Commit format

Enforced by `.githooks/commit-msg`. Full rules + example + hook
installation steps live in [`CONTRIBUTING.md`](CONTRIBUTING.md#2-commit-messages).

Quick recap: subject ≤72 chars, `Authored-by:` trailer required,
`Co-Authored-By:` when AI-assisted.

---
> Source: [skbays03/absentia](https://github.com/skbays03/absentia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-16 -->
