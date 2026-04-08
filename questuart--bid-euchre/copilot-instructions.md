## bid-euchre

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Bid Euchre AI Research Framework — a Python framework for deterministic simulation and strategy evaluation of the card game Bid Euchre (double-deck, 10-A variant with bowers).

## Communication

- For non-trivial requests (multi-step tasks, ambiguous scope, architectural decisions), start your response with a brief **Intent:** line restating what you understand the user's goal and intent to be (1-2 sentences). Then proceed with the work.
- Skip the intent restatement for single-step tasks, simple questions, and follow-up confirmations.
- If the intent restatement reveals uncertainty, ask before acting.

## Git & PR Workflow

- Always use `git worktree` for PR branches — never work directly on main.
- Pattern: `git worktree add ../worktree-<branch> -b <branch>`
- After merging a PR: update local main (`git pull`), clean up the worktree (ephemeral PR worktrees only; persistent role worktrees are never removed — see `docs/02_agent/AUTONOMOUS_OPERATOR_WORKFLOW.md` § Cleanup Policy), and update MEMORY.md.
- When merging stacked PRs, merge bottom-up and immediately recreate any auto-closed downstream PRs before proceeding.

### Stacked PRs

- When creating stacked/dependent PRs, document the dependency chain in each PR description.
- If a base branch PR is merged and GitHub auto-closes downstream PRs, recreate them targeting the new base (usually `main`).
- Expect rebase conflicts when working with stacked PRs — resolve them methodically, don't panic.

## Workflow

- Always create a **plan** before implementing. Never start coding without an explicit written plan unless the user says otherwise.
- When the user asks for a "plan", produce ONLY a written plan document — do NOT begin implementation.
- When asked to discuss, analyze, or explore, stay in discussion mode — do not start writing code unless explicitly asked.
- Before claiming data, files, or APIs don't exist, verify with Grep and Read. Never assume absence — always check the actual codebase.
- Always ask clarifying questions about scope before starting multi-PR plans.
- Save plans as markdown files in a `plans/` directory.
- Do not use EnterPlanMode as a substitute for file-based planning — always write plans as markdown files. Users may still invoke `/plan` for interactive exploration.
- **Governed initiatives:** Work belonging to a major initiative uses the governing plan framework. Plans live under `plans/<initiative>/`. See `docs/02_agent/AGENTS.md` section 12 for the plan hierarchy.
- **Session-scoped work:** For standalone tasks (one-off bugfixes, small features, isolated PRs) that do NOT belong to a governed initiative, save to `plans/sessions/YYYY-MM-DD_<slug>.md`.
- Every plan file should include an `## Outcome` section (filled after implementation) linking to resulting PR(s) or noting abandonment.
- Use `/review-plan [path]` for independent plan review with Codex CLI + Claude failsafe.

### Planning Rules

- When creating plans, read the actual source code and API signatures first — never guess.
- Plans must reference real file paths, real function names, and real parameter signatures from the codebase.
- If the user provides their own plan structure, adopt it rather than proposing an alternative.
- Before presenting a plan, verify ALL outstanding items (DoD, blockers, open work items) against the plan document. Never skip items that are explicitly listed.
- After drafting a plan, self-audit for internal contradictions (e.g., mandating something in one section while exempting it in another) and verify all file paths and output paths against the actual repo before presenting.
- **Do not invent ad hoc planning structures** once a governing plan exists. All implementation work for a governed initiative traces to a governing plan step or a registered sub-plan.

## Python

- **Requires Python 3.10+**
- Default to `uv run` for all Python commands (pytest, notebooks, scripts) — not raw `python` or `pip`.
- Always run `ruff check` and `ruff format` before committing.
- Watch for: unused imports after refactors, f-strings without placeholders, circular imports when modifying `__init__.py`.

### Pre-Commit Checklist

- Run `git fetch origin main && git rebase origin/main` before opening a PR to avoid stale-base conflicts.
- Run full `make check-gated` (or `make check`) before creating any PR.
- Verify no uncommitted notebook changes that would trigger git diff checks.
- Run linter (`ruff check --fix`) and formatter (`ruff format`) on all changed files.

## Memory Management

- Update MEMORY.md after **every** PR merge with: PR number, branch name, one-line summary.
- After completing major work (plan completion, experiment runs), also update with status and next steps.
- When resuming from a previous session, read MEMORY.md first to recover context.

## Essential Commands

### Install Dependencies
```bash
make sync               # Install dependencies (uses uv sync)
```

```bash
make check-gated        # Full validation with concurrency cap (default for fleet — run before PRs)
make check-quiet        # Same validation, minimal output, no concurrency cap (single-lane debugging)
make check              # Full validation, full output (interactive debugging)
make test               # Pytest fast suite only
make lint               # Ruff check only
make repo-lint          # Repo boundary linter only
make notebook-sync      # Sync paired .py ↔ .ipynb (Jupytext)
make notebook-check     # Verify sync + outputs cleared
make docs-check         # Verify docs freshness
make help               # Show all available targets
```

### Notebook Execution (Not in make check)
```bash
make notebook-run       # Execute notebooks (SMOKE mode, ~10s)
make notebook-run-full  # Execute notebooks (QUICK mode, ~2-5min)
```
These validate notebook execution but are **not** included in `make check`.

### Review Infrastructure Testing (Not in make check)
```bash
make review-smoke       # SMOKE test review infra (~30s)
make review-quick       # QUICK test review infra (~5min, needs Codex auth)
make review-full        # FULL test review infra (~15min, needs Codex auth)
```

### Running Experiments

```bash
# Canonical experiment runner (always use this)
uv run python experiments/run_experiment.py --seed 42 \
  --config experiments/configs/quick_test.yaml --n_per 10

# Run experiment suite
uv run python scripts/run_suite.py \
  --suite experiments/suites/baseline_tiny.yaml \
  --seed 42 \
  --n-per 20

# Dry-run config validation
uv run python experiments/run_experiment.py --seed 42 --dry-run \
  --config experiments/configs/quick_test.yaml
```

### Comparing Experiment Runs
```bash
# Compare two runs with bootstrap statistics
uv run python scripts/compare_runs.py \
  --baseline data/runs/<baseline_run_id> \
  --candidate data/runs/<candidate_run_id> \
  --seed 42 \
  --n-bootstrap 10000 \
  --format markdown  # For PR bodies
```

### Running Tests

```bash
uv run python -m pytest -m "not slow" tests/     # Fast suite
uv run python -m pytest tests/unit/              # Unit only
uv run python -m pytest tests/integration/       # Integration only
uv run python -m pytest tests/unit/test_rules.py::test_specific  # Single test
```

**Note:** All commands use `uv run` which handles the virtualenv automatically. If already in an activated venv, plain `python` works too.

**Fresh worktrees:** `make check` auto-bootstraps the venv via `ensure-venv` (runs `uv sync` if `.venv` is missing). No manual setup needed after `git worktree add`.

## Architecture

### Source Layout (`src/bid_euchre/`)

| Module | Purpose |
|--------|---------|
| `core/` | Card primitives, rules, legality, trick resolution (source of truth) |
| `sim/` | Simulation loop, deal generation, orchestration |
| `strategy/` | Bot policies and decision logic |
| `features/` | Hand evaluation and feature extraction |
| `experiments/` | Config parsing (StrategyConfig, ExperimentConfig) |
| `datasets/` | Dataset collectors (bidding, bidless) |
| `models/` | Model training/inference |
| `diagnostics/` | Visualization and analysis tools |
| `reporting/` | Report generation utilities |
| `logging/` | JSONL game logging |
| `analysis/` | Statistical analysis (stats, paired comparisons, models) |
| `arc_d_v2/` | Arc D v2 lineage orchestration, charts, tables, reports |
| `agent_ops/` | Governed follow-on plan for agentic orchestration platform work |
| `validation/` | Promotion validation and schemas |
| `ops/` | Operator tooling (internal): status, worktrees, events, watchdogs, scheduler |
| `scoring.py` | Top-level scoring module |

**Import boundary:** `src/` must NOT import from `experiments/` or `tests/`.

### Key Directories

- `experiments/` — Configs (`configs/`), suites (`suites/`), and canonical runner (`run_experiment.py`)
- `scripts/` — Blessed tooling (report generation, suite runner, drift detection)
- `tests/` — Unit, integration, performance, property tests
- `data/runs/` — Generated outputs (never committed)
- `notebooks/` — Jupytext-paired notebooks (edit `.py` files, not `.ipynb`)

## Critical Constraints

### Determinism
- **Seed required** for experiments: `--seed <int>` (use `--allow-nondeterministic` only for exploration)
- Same seed + config = identical results
- Strategies must use local `random.Random(seed)`, never global `random.*`

### Data Policy
- **Never commit** `data/runs/`, `data/reports/`, `data/models/`
- Only `data/fixtures/` may be committed (tiny test fixtures)

### Statistical Rigor
This repo prioritizes technical correctness over convenience. Key requirements:
- **Sample size minimums:** ≥2,000 deals for bias detection, ≥50,000 for production reports
- **Statistical validation required:** Hypothesis tests with p-values, confidence intervals, effect sizes
- **No visual-only validation:** Statistical tests must accompany visual inspection
- **Fail-fast gates:** Use assert-style sanity checks in notebooks and pipelines

See `.claude/rules/deferred/05_rigor.md` for complete standards.
See `.claude/rules/deferred/35_integrity.md` for methodology deferral analysis requirements.

### Worktree-Only Workflow
All code changes MUST happen in dedicated git worktrees, never on `main` in the shared checkout. Pre-commit hooks enforce this policy. See `.claude/CLAUDE.md` for detailed workflow.

### PR Requirements
- One concept per PR
- Run `make check-gated` before opening
- Include exact repro command with seed in PR description
- Use the PR template from `.github/pull_request_template.md`
- **Define test criteria** — every PR must include specific, verifiable pass/fail conditions (not just "tests pass"). State what observable outcome proves the feature works, with exact commands and expected results.
- After `reviewing-changes` passes, wait for Codex pre-merge review before merging (see `docs/02_agent/CODEX_GITHUB_REVIEW.md`)

### Post-Merge Review
- After every `gh pr merge`, a PostToolUse hook triggers a comprehensive review
- A background Explore agent reviews merged code for correctness, contract compliance,
  architecture, and test coverage
- CRITICAL findings trigger immediate fix PRs
- This is a safety net — pre-merge review catches most issues, post-merge catches the rest

## Game Rules Summary

- **Deck:** Double-deck (40 cards), ranks 10-A, 4 suits × 2 copies
- **Hand:** 10 cards per player, 10 tricks per hand
- **Partnerships:** Seats (0,2) vs (1,3)
- **Contract types:** `"suit"` (with trump/bowers), `"high"` (no-trump, A high), `"low"` (no-trump, 10 high)
- **Bowers (suit contracts only):** Right bower = J of trump, Left bower = J of same color
- **Scoring:** Declaring team gets tricks won if made, `-bid` if set; defending team always gets tricks won

See `docs/01_core/RULES.md` for complete rules specification.

## Adding a New Strategy

1. Implement in `src/bid_euchre/strategy/<name>.py`
2. Export in `src/bid_euchre/strategy/__init__.py`
3. Register in `src/bid_euchre/experiments/config.py` (`StrategyConfig.create_strategy`)
4. Add unit tests in `tests/unit/`
5. Add/update YAML config in `experiments/configs/`
6. Run seeded smoke experiment to validate

## Key Documentation

- `docs/01_core/RULES.md` — Authoritative game rules
- `docs/01_core/ARCHITECTURE.md` — System design and boundaries
- `docs/01_core/EXPERIMENTS.md` — Experiment runner and output structure
- `docs/01_core/REPRODUCIBILITY.md` — Seeding and determinism
- `docs/02_agent/AGENTS.md` — Development workflow for AI agents (including governing plan framework, section 12)
- `docs/02_agent/AGENT_EXECUTION_PROTOCOL.md` — Full agent execution protocol for governed initiatives
- `docs/TESTING_STRATEGY.md` — Test layers, commands, and two-tier policy
- `docs/STYLEGUIDE.md` — Coding conventions, placement rules, and hard constraints

## Active Governing Plans

| Initiative | Governing Plan | Status |
|-----------|---------------|--------|
| Arc D v2 (multi-model lineage) | `plans/arc_d_v2/lineage_plan.md` | COMPLETE |
| Browser Game Hosting and Human Data Capture | `plans/browser_game/governing_plan.md` | ACTIVE |
| Agentic Orchestration Platform | `plans/agent_ops/governing_plan.md` | PHASE 4 COMPLETE — POSTPONED |

When starting work on a governed initiative, follow the Agent Execution Protocol.

## Agent Execution Protocol

See `docs/02_agent/AGENT_EXECUTION_PROTOCOL.md` for the full protocol. Summary:

1. **Discovery:** Read `CLAUDE.md` → governing plan → `checkpoints.md` → `sub_plan_registry.md` → resume
2. **Next work:** Find first `PENDING`/`IN_PROGRESS` step in checkpoints; if all `COMPLETE`, advance to next phase
3. **Blockers:** Mark `BLOCKED` in checkpoints, log in `qa_log.md`, escalate — never silently work around
4. **Completion:** Update checkpoints, verify `Validates` conditions, update `MEMORY.md`
5. **Session handoff:** Update checkpoints + MEMORY.md, record partial progress, note uncommitted artifacts

**Key constraints:**
- Spawned agents die silently at ~15 min / ~700KB — keep tasks small (see `.claude/rules/70_agent_reliability.md`)
- `checkpoints.md` is authoritative for human handoff; `state.json` for orchestrator decisions
- Create sub-plans for steps with >3 files changed or design choices not in the governing plan

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Questuart)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Questuart)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
