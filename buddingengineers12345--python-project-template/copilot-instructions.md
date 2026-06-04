## python-project-template

> This repository is a **Copier template repository** (a meta-project). Running Copier against

# python_project_template — Claude Project Knowledge

## What this repo is

This repository is a **Copier template repository** (a meta-project). Running Copier against
this repo **generates** a new Python project by rendering the `template/` directory into a
destination folder.

> [!WARNING]
> Copier can run **template tasks** during `copier copy`/`copier update`. Only use the
> `--trust` flag with templates you trust.

## Directory structure

```
.
├── template/                 # Jinja2 source files that Copier renders
│   ├── src/{{ package_name }}/   # Generated package source (common/, core.py, cli.py…)
│   ├── tests/                # Generated project test suite
│   ├── .claude/              # Claude hooks/commands/rules/skills for generated projects
│   ├── .github/workflows/    # Generated CI/CD workflows
│   └── …                    # pyproject.toml.jinja, justfile.jinja, CLAUDE.md.jinja, …
├── tests/                    # pytest suite for this meta-repo (see tests/CLAUDE.md)
│   ├── constants.py          # REPO_ROOT / TEMPLATE_ROOT / COPIER_YAML for nested test modules
│   ├── conftest.py           # top-level shared fixtures
│   ├── unit/                 # fast isolated script tests
│   ├── integration/          # Copier copy/update integration suite
│   └── e2e/                  # end-to-end tests (placeholder)
├── scripts/                  # Automation scripts for CI or local tasks (see scripts/CLAUDE.md)
│   ├── repo_file_freshness.py    # Git-based freshness dashboard (→ docs/ + assets/)
│   ├── bump_version.py           # PEP 440 version bumper (patch/minor/major)
│   ├── check_root_template_sync.py  # Root ↔ template parity (workflows, settings, recipes)
│   ├── pr_commit_policy.py       # PR title/body + commit message policy (CI)
│   └── sync_skip_if_exists.py    # Sync copier.yml _skip_if_exists with template paths
├── .claude/                  # Claude Code hooks, commands, and rules for THIS meta-repo
│   ├── settings.json         # Hook registrations and permission allow/deny lists
│   ├── hooks/                # Shell hook scripts (see hooks/README.md)
│   ├── commands/             # Slash command prompts (/review, /generate, /release, …)
│   └── rules/                # AI rules (common/, python/, jinja/, bash/, yaml/, copier/)
├── docs/                     # Markdown output folder (repo_file_status_report.md, etc.)
├── assets/                   # Freshness JSON artifacts (file_freshness.json, etc.)
├── .github/                  # Meta-repo GitHub Actions workflows
├── copier.yml                # Template prompts, computed vars, and post-gen tasks
├── justfile                  # Task runner (use `just` not raw commands)
├── pyproject.toml            # Dev deps for THIS repo (not for generated projects)
├── .pre-commit-config.yaml   # Pre-commit hooks for meta-repo
└── uv.lock                   # Committed lockfile — never delete
```

## How to set up the development environment

```bash
uv sync --frozen --extra dev   # install all dev deps from the lockfile
just precommit-install         # register git hooks
```

Prerequisites: Python 3.11+, `uv`, `just`, `git`.

## Common development commands

| Task | Command |
|---|---|
| List recipes (default) | `just` or `just default` |
| Run all tests | `just test` |
| Run tests in parallel | `just test-parallel` |
| Run slow tests only | `just slow` |
| Fast unit tests (no slow/integration) | `just test-fast` |
| Integration tests only | `just test-integration` |
| Tests for changed files only | `just test-changed` |
| Verbose tests | `just test-verbose` |
| Full debug test output | `just test-debug` |
| Re-run last failed tests | `just test-lf` |
| Re-run last failed tests (max verbosity) | `just test-failed-verbose` |
| Stop on first test failure | `just test-first-fail` |
| CI-style tests + coverage XML | `just test-ci` |
| Coverage report | `just coverage` |
| Lint | `just lint` |
| Lint changed files only | `just lint-changed` |
| Format | `just fmt` |
| Format check (read-only) | `just fmt-check` |
| Auto-fix lint issues | `just fix` |
| Type check | `just type` |
| Docstring check | `just docs-check` |
| MkDocs recipes (generated projects only) | `just docs-help` |
| Pre-merge review (fix + lint + type + docs) | `just review` |
| Full CI locally (fix → check) | `just ci` |
| Read-only CI check (no auto-fix) | `just check` |
| Run pre-commit on all files | `just precommit` |
| Register git hooks | `just precommit-install` |
| Interactive conventional commit (Commitizen) | `just cz-commit` |
| Sync deps after lockfile change | `just sync` |
| Upgrade all deps | `just update` |
| Check for outdated dependencies | `just deps-outdated` |
| Verify lockfile integrity | `just lock-check` |
| Dependency security audit | `just audit` |
| Install all deps + pre-commit | `just install` |
| One-command developer onboarding | `just bootstrap` |
| Diagnose environment | `just doctor` |
| Generate freshness dashboard | `just freshness` |
| Root ↔ template sync validation | `just sync-check` |
| Suggested PR title + body (PR policy) | `just pr-draft` |
| Clean build artifacts | `just clean` |
| Build distribution | `just build` |
| Validate built distribution | `just check-dist` |
| Publish package | `just publish` |

**Always use `just` recipes.** Do not call `uv run ruff`, `pytest`, etc. directly —
the justfile handles the correct flags and order.

## Running the full CI pipeline

```bash
just ci
```

This runs: `fix` → `check`.

`check` bundles: `uv sync --frozen`, `fmt-check`, `ruff check`, `basedpyright`,
`sync-check`, `docs-check` (D-only; redundant with `ruff check` for enforcement), `test-ci`
(pytest + coverage XML), `pre-commit run --all-files --verbose`, `audit` (pip-audit).

Together, `lint.yml` + `tests.yml` + `security.yml` mirror these checks on GitHub (CodeQL is
GHA-only). All steps must pass before a PR is mergeable.

## Generating a test project from the template

To manually generate a project and inspect the output:

```bash
copier copy . /tmp/test-output --trust --defaults
```

To pass specific answers non-interactively:

```bash
copier copy . /tmp/test-output --trust \
  --data project_name="My Library" \
  --data include_docs=false \
  --data include_pandas_support=false
```

Clean up afterward: `rm -rf /tmp/test-output`

When iterating on an **uncommitted** local template, use **`copier copy --vcs-ref HEAD . DESTINATION`**
so Copier uses the current tree (otherwise it may select the latest PEP 440 tag and skip dirty files).

## Copy vs update (important)

- **Generate** a new project: `copier copy TEMPLATE DESTINATION`.
- **Update** an existing generated project to the latest template version: `copier update`.
- **`copier recopy`**: reapplies the template and keeps answers; does **not** use the smart merge
  algorithm. Prefer `copier update` for normal sync.

For updates, Copier works best when:

1. The template includes a valid `.copier-answers.yml`.
2. The template is versioned with git tags (PEP 440 versions).
3. The destination folder is versioned with git.

Useful flags: **`copier update --defaults`**, **`--data` / `--data-file`** for selective answer changes,
**`--vcs-ref=:current:`** to re-record answers without changing template version, **`copier check-update`**
(JSON via **`--output-format json`**, **`--quiet`** exit code `2` when an update exists).

### Never edit `.copier-answers.yml` by hand

Do not manually change the answers file (`.copier-answers.yml`, controlled by `_answers_file` in
`copier.yml`). The update algorithm relies on it and manual edits can lead to unpredictable diffs.

### Handle update conflicts

Default **`--conflict inline`** produces merge-style markers in files; **`--conflict rej`** writes
`*.rej` sidecars. Generated projects ship pre-commit hooks for both patterns (`check-merge-conflict`
and a fail-on-`*.rej` hook). Review conflicts before committing.

### Template `_tasks` on copy vs update

Post-generation **`_tasks`** in `copier.yml` run after both **`copier copy`** and **`copier update`**
(intentionally, so `uv.lock` and the dev environment stay aligned). Use **`copier update --skip-tasks`**
when you need to skip them.

### Maintainer: releases and `_migrations`

Tag releases with **PEP 440**-compatible versions so consumers’ `copier update` picks the right template
revision. Introduce **`_migrations`** in `copier.yml` when you need scripted steps across template
versions (e.g. renaming generated paths). Prefer SSH or credential-free Git URLs so `_src_path` in
answers files stays clean. Shallow template clones in CI can increase Git work for Copier; use full
clones if you hit resource issues.

## How tests work

Tests live in `tests/`. They use pytest to call `copier copy` programmatically,
render the template into a temporary directory, then assert that expected files
exist and contain expected content. When adding a new Copier variable or template
file, add a corresponding test.

## Copier variable conventions (copier.yml)

- Questions are defined under top-level keys in `copier.yml`.
- Computed values (not shown to users) use **`when: false`** with a **`default`** so they are not
  prompted or stored in the answers file.
- Use **`secret: true`** (with a **`default`**) for sensitive answers so they are not written to
  `.copier-answers.yml`.
- Post-generation `_tasks` run shell commands after rendering. Keep them idempotent.
- `_skip_if_exists` prevents overwriting user edits when running `copier update`.

## Jinja2 template conventions (template/)

- Template files use `{{ variable_name }}` for substitution.
- Conditional blocks use `{% if condition %}...{% endif %}`.
- File names can themselves be templated: e.g., `src/{{ package_name }}/__init__.py.jinja`.
- The `jinja2_time.TimeExtension` is available for `{% now %}` expressions.
- The `jinja2.ext.do` and `jinja2.ext.loopcontrols` extensions are also enabled.

## Code style

- Line length: 100 characters (set in `pyproject.toml` under `[tool.ruff]`).
- Target Python version: 3.11.
- Active ruff rules: `E`, `F`, `I`, `UP`, `B`, `SIM`, `C4`, `RUF`, `TCH`, `PGH`, `PT`, `ARG`, `D`, `C90`, `PERF`, `T20`.
  Rule `E501` (line too long) is ignored (handled by the formatter).
- Docstring convention: **Google style** (`pydocstyle` via ruff `D` rules).
  In this meta-repo, `tests/**` and `scripts/**` enforce `D` like other Python; only `T20` (`print`) is ignored there.
  **Generated projects** (from `template/`) treat `src/**/common/bump_version.py` like other library
  code for ruff `D` (Google docstrings required); the release helper uses
  `logging_manager` (`configure_logging`, structlog events, `write_machine_stdout_line` for the
  version line consumed by release tooling). Rendered **`template/CLAUDE.md.jinja`** is the source
  of truth for generated apps: **all logging must go through `common.logging_manager` public APIs**,
  and code outside `common/` must **prefer** imports from that package's `common` subpackage (file
  I/O, decorators, utils, logging) instead of duplicating those concerns.
- McCabe complexity: max 10 per function (`C90`).
- Type annotations are required on all public functions and methods (basedpyright `standard` mode).
- BasedPyright is lenient with external packages (`reportMissingTypeStubs = false`).
- `T20` (flake8-print): `print()` is discouraged in non-test code; use structured logging instead.

## AI rules

Detailed coding standards are documented as plain Markdown files under `.claude/rules/`
and are readable by any AI assistant (Claude Code, Cursor, or any LLM):

```
.claude/rules/
├── README.md            ← how to read and write rules; dual-hierarchy explained
├── common/              ← language-agnostic: coding-style, git-workflow, testing, security,
│                           development-workflow, code-review
├── python/              ← Python: coding-style, testing, patterns, security, hooks
├── jinja/               ← Jinja2: coding-style, testing  (meta-repo only)
├── bash/                ← Bash: coding-style, security
├── markdown/            ← placement rules, authoring conventions
├── yaml/                ← YAML formatting for copier.yml and workflows  (meta-repo only)
└── copier/              ← Copier template conventions (meta-repo only)
    └── template-conventions.md
```

The `template/.claude/rules/` tree mirrors this structure for generated projects
(common, python, bash, markdown — no Jinja, yaml, or Copier-specific rules).

## Standards enforcement

Standards are enforced at four layers — during development, at commit, in review, and in CI.

### Tooling layer (always active)

| What | Tool | When |
|---|---|---|
| Lint + style | ruff | `just lint` / pre-commit / CI |
| Docstrings | ruff `D` rules (Google) | `just docs-check` / CI |
| Complexity | ruff `C90` (max 10) | `just lint` |
| Performance | ruff `PERF` | `just lint` |
| Types | basedpyright `standard` | `just type` / pre-commit / CI |
| Test coverage | pytest-cov (reported) | `just coverage` |

### Claude layer (automatic feedback during development)

Hooks are registered in `.claude/settings.json` and fire at each lifecycle event:

| Hook script | Event | Matcher | Purpose |
|---|---|---|---|
| `session-start-bootstrap.sh` | SessionStart | * | Show toolchain status + previous session snapshot |
| `pre-bash-block-no-verify.sh` | PreToolUse | Bash | Block `--no-verify` in git commands |
| `pre-bash-git-push-reminder.sh` | PreToolUse | Bash | Warn to run `just review` before push |
| `pre-bash-commit-quality.sh` | PreToolUse | Bash | Scan staged `.py` files for secrets/debug markers |
| `pre-config-protection.sh` | PreToolUse | Write\|Edit\|MultiEdit | Block weakening ruff/basedpyright config edits |
| `pre-protect-uv-lock.sh` | PreToolUse | Write\|Edit | Block direct edits to `uv.lock` |
| `pre-bash-coverage-gate.sh` | PreToolUse | Bash | Warn before `git commit` if coverage below 85% |
| `pre-write-src-require-test.sh` | PreToolUse | Write\|Edit | Block writing `src/<pkg>/<module>.py` if matching test module is missing (strict TDD; register this **or** `pre-write-src-test-reminder.sh`) |
| `pre-write-src-test-reminder.sh` | (optional) | Write\|Edit | Warn if `tests/<pkg>/test_<module>.py` missing (non-blocking alternative to strict TDD hook) |
| `pre-write-doc-file-warning.sh` | PreToolUse | Write | Block `.md` files outside `docs/` |
| `pre-write-jinja-syntax.sh` | PreToolUse | Write | Validate Jinja2 syntax before writing `.jinja` files |
| `pre-suggest-compact.sh` | PreToolUse | Edit\|Write | Suggest `/compact` every ~50 operations |
| `pre-compact-save-state.sh` | PreCompact | * | Snapshot git state before compaction |
| `post-edit-python.sh` | PostToolUse | Edit\|Write | Run ruff + basedpyright after every `.py` edit |
| `post-edit-jinja.sh` | PostToolUse | Edit\|Write | Validate Jinja2 syntax after every `.jinja` edit |
| `post-edit-markdown.sh` | PostToolUse | Edit | Warn if existing `.md` edited outside `docs/` |
| `post-edit-refactor-test-guard.sh` | PostToolUse | Edit\|Write | Remind to run tests after several `src/` or `scripts/` edits |
| `post-edit-copier-migration.sh` | PostToolUse | Edit\|Write | Remind about `_migrations` after `copier.yml` edits |
| `post-edit-template-mirror.sh` | PostToolUse | Edit\|Write | Remind to mirror `template/.claude/` ↔ root `.claude/` |
| `post-bash-pr-created.sh` | PostToolUse | Bash | Log PR URL after `gh pr create` succeeds |
| `stop-session-end.sh` | Stop | * | Persist session state JSON |
| `stop-evaluate-session.sh` | Stop | * | Extract reusable patterns from transcript |
| `stop-cost-tracker.sh` | Stop | * | Track and accumulate session token costs |
| `stop-desktop-notify.sh` | Stop | * | macOS desktop notification on completion |

See `.claude/hooks/README.md` for full details on exit codes, JSON input format, and adding hooks.

### Claude commands (on-demand workflows)

| Slash command | Purpose |
|---|---|
| `/review` | Full pre-merge checklist: lint + types + docstrings + test coverage + symbol scan |
| `/coverage` | Run coverage, identify gaps, write missing tests |
| `/docs-check` | Audit and repair Google-style docstrings across all source files |
| `/standards` | Consolidated pass/fail report across all checks — the "ready to merge?" gate |
| `/update-claude-md` | Sync CLAUDE.md against pyproject.toml + justfile to prevent drift |
| `/generate` | Generate a test project from the template into `/tmp/test-output` |
| `/release` | Orchestrate a new release: verify CI, bump version, tag, push |
| `/validate-release` | Verify release prerequisites (clean tree, passing CI, correct tag format) |
| `/ci` | Run `just ci` and report results |
| `/test` | Run `just test` and summarise failures |
| `/dependency-check` | Validate `uv.lock` is committed, in sync, and not stale |
| `/tdd-red` | Validate RED phase: confirm a test fails for the right reason |
| `/tdd-green` | Validate GREEN phase: confirm the test passes with no regressions |
| `/ci-fix` | Autonomous CI fixer: diagnose failures, apply fixes, re-run until green |

### Definition of done

A feature or fix is **done** when all of the following are true:

1. `just ci` passes with zero errors.
2. Every new public function/class/method has a Google-style docstring.
3. Every new function/method has complete type annotations (parameters + return type).
4. At least one test case covers each new public symbol.
5. `just coverage` does not show a new module below its previous coverage level.
6. No `TODO` or `FIXME` comments are left in modified files (unless tracked as issues).
7. CLAUDE.md is up to date (`/update-claude-md` shows no drift).

## Markdown file placement rule

Any Markdown (`.md`) file you create as part of a workflow, analysis, or investigation output
**must be written inside the `docs/` folder**.

**Allowed exceptions (may be placed anywhere):**
- `README.md`
- `CLAUDE.md`

Do **not** create free-standing `.md` files (e.g. `ANALYSIS.md`, `LOGGING_ANALYSIS.md`,
`NOTES.md`) at the repository root or inside `src/`, `tests/`, `scripts/`, or any directory
outside `docs/`. This rule is enforced by `.claude/hooks/post-edit-markdown.sh`, which fires
after every file write and surfaces a violation message if the rule is broken.

## Files you should never modify directly

- `uv.lock` — regenerated by `just update`. Never hand-edit.
- `.copier-answers.yml` (appears in generated projects, not this repo) — managed by Copier.

## Files that are safe to delete for a clean slate

```bash
just clean   # removes build/, dist/, .pytest_cache, .ruff_cache, __pycache__, *.pyc
```

## Recent improvements (April 2026)

### Standards enforcement (this PR)
- Added `D` (pydocstyle, Google convention), `C90` (McCabe complexity), `PERF` (perflint) to ruff rules
- Added `per-file-ignores` so `tests/**` and `scripts/**` ignore `T20` only (`print`); docstrings (`D`) apply;
  generated projects enforce Google docstrings on `src/**/common/bump_version.py` like the rest of `src/`
- Added `[tool.ruff.lint.mccabe]` max-complexity = 10
- Enhanced basedpyright config: `standard` mode, lenient with external stubs
- Added `pytest-cov` + `coverage[toml]` to dev dependencies
- Added `just docs-check` and `just review` recipes; `just ci` now includes docs-check
- Added `.claude/hooks/post-edit-python.sh` and `post-edit-jinja.sh` (PostToolUse hooks)
- Added five new Claude commands: `/review`, `/coverage`, `/docs-check`, `/standards`, `/update-claude-md`
- Added Standards Enforcement section to CLAUDE.md (definition of done, tooling table, command table)
- Propagated all of the above into `template/` so generated projects inherit the same enforcement

### Package structure
- Renamed `_support` package to `common` for clearer semantics
- Refactored support utilities into dedicated modules: `file_manager`, `logging_manager`, `decorators`, `utils`
- Enhanced logging manager with level-aware log functions for all severity levels

### Testing enhancements
- Added comprehensive test coverage for `common` utilities (`test_support.py`)
- Improved test organization with new `tests/` directory structure in generated projects
- Added parametrized tests for license rendering and feature combinations

### CI/CD improvements
- Updated GitHub Actions workflow versions for better compatibility
- Added `--frozen` flag to `uv sync/update` recipes for reproducible builds
- Added pre-commit hook to reject Copier rejection files (`*.rej`)
- Enhanced test matrix to include Python 3.13 testing

### Documentation scaffolding
- Added `docs/` directory template with MkDocs structure
- Added `index.md.jinja` documentation template
- Added `.claude/` commands for template-specific automation

### Release automation
- Added `release.yml` GitHub Actions workflow for automated version bumping and releases
- Integrated `scripts/bump_version.py` for PEP 440 version management
- Template now supports manual release triggering with configurable bump strategy

### Claude documentation update (April 2026)
- Updated root `CLAUDE.md`: accurate `just ci` pipeline description, complete justfile recipe table,
  added `T20` ruff rule, full hooks table, full slash-commands table, expanded directory structure
- Fixed `template/CLAUDE.md.jinja`: corrected basedpyright mode from "strict" to "standard",
  added `/release` slash command entry
- Added `CLAUDE.md` in `template/` — explains Jinja2 source layout, Copier variables, dual `.claude/` hierarchy
- Added `CLAUDE.md` in `tests/` — explains test patterns, helpers, categories, and how to add new tests
- Added `CLAUDE.md` in `scripts/` — documents each script, CLI flags, outputs, and CI integration
- Added `CLAUDE.md` in `.claude/` — orientation hub for hooks, commands, rules, and the dual-hierarchy
- Added `CLAUDE.md` in `.github/` — documents all meta-repo workflows and design principles

---
> Source: [buddingengineers12345/python_project_template](https://github.com/buddingengineers12345/python_project_template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-04 -->
