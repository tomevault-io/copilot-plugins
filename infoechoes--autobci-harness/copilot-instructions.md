## autobci-harness

> AutoBCI is a local, auditable research-loop harness. It helps a coding agent turn

# Project guidance for coding agents

## Mission

AutoBCI is a local, auditable research-loop harness. It helps a coding agent turn
a research task into a bounded Program, a direction queue, restricted code
execution, fixed evaluation, reviewable ledger records, and a Dashboard view.

This public harness export is deliberately narrow. It is meant to prove the
research-loop shape on a clean machine without carrying private strategy notes,
historical research trees, or raw scientific data.

## Hard Rules

1. Never modify files under `data/raw/`.
2. Do not invent channel names, joint names, target names, or time columns. Read
   them from config or inspect actual files.
3. Preserve strict causality:
   - model input windows may only use present or past samples;
   - do not use future samples in preprocessing, normalization, smoothing, or
     target construction.
4. Do not change alignment logic, data splits, or primary metrics unless the
   user explicitly asks.
5. Fail visibly when a provider key, dataset, runner, or command is missing.
   Do not use silent fake fallbacks.
6. Do not introduce default dataset downloads, profiler traces, tensor dumps, or
   unbounded artifacts. New runners must enforce a storage budget before
   training or repeated artifact writes.
7. Prefer small, reviewable changes.

## First Files

Read these before editing:

1. `README.md`
2. `DEMO_QUICKSTART.md`
3. `docs/storage_budget.md`
4. `scripts/serve_dashboard.py`
5. `.agents/skills/autobci-harness/SKILL.md`

## Public Alpha Path

The public alpha is deliberately narrow. It should prove that a new machine can
run:

```bash
bash scripts/install.sh
source .venv/bin/activate
autobci doctor --json
autobci status --json
autobci ask "现在进展如何？" --json
autobci demo onsite --skip-smoke
autobci dashboard
```

If a real model API key is configured, the live smoke path is:

```bash
autobci demo onsite --provider minimax-cn --model MiniMax-M3
```

If the key or model is unavailable, this must fail explicitly.

## Product Boundary

AutoBCI is not a general autonomous agent OS. It is the control plane around a
research loop:

- Program and task boundary
- research direction queue
- restricted execution sandbox
- fixed evaluator
- ledger, events, artifacts, and rollback evidence
- Dashboard projection

The Dashboard is not a second source of truth. The source of truth is the
machine-readable ledger, events, and artifact files.

## Product Entry

The public harness is headless by default. Do not reintroduce a TUI as the
primary entry. Claude Code, Codex, Cursor, Workbody, Hermes, ClawBot, or any
other agent should drive AutoBCI through stable CLI/JSON commands:

```bash
autobci doctor --json
autobci status --json
autobci ask "现在进展如何？" --json
autobci data set /absolute/path/to/dataset
```

Mobile gateways are transport only. They may call whitelisted CLI commands and
send reports to the user, but Program, ledger, events, and artifacts remain the
research source of truth.

## Owner Debug Requirements

Any completed research-loop feature should make these questions answerable:

- Who proposed the direction?
- What evidence or prior result motivated it?
- Why this track instead of the alternatives?
- Which files could be edited?
- What diff was actually produced?
- Which commands ran?
- Where are stdout, stderr, metrics, and artifacts?
- Was the score selected by the system, merely the highest observed candidate,
  or still unverified?
- Was any rollback applied?

High scores from parameter sweep, threshold tuning, lucky split, leakage, or
post-hoc cherry-picking must be labeled as risk, not presented as algorithmic
progress.

## Editable By Default

- `src/**`
- `scripts/**`
- `configs/**`
- `dashboard/**`
- `tests/**`
- `.agents/skills/**`

## Read-Only Unless Explicitly Approved

- `data/raw/**`
- Program files under `programs/**`
- split logic
- primary metrics
- fixed evaluators

## Validation Defaults

When touching CLI, provider, Dashboard, runner, or research-loop code, run
the narrowest relevant checks, for example:

```bash
PYTHONPATH=src pytest -q tests/test_headless_cli.py
PYTHONPATH=src python -m bci_autoresearch.product_shell.cli doctor --json
PYTHONPATH=src python -m bci_autoresearch.product_shell.cli status --json
PYTHONPATH=src python -m bci_autoresearch.product_shell.cli ask "现在进展如何？" --json
PYTHONPATH=src python -m bci_autoresearch.product_shell.cli demo onsite --skip-smoke --json
```

For research-loop changes, report the Program boundary, exact data split,
primary metric, artifact paths, and whether any result is only a smoke result,
a candidate, or a fixed-evaluator result.

## Style

- Python 3.10+
- Type hints where practical
- Terminal-runnable scripts
- Clear failures on ambiguous assumptions
- No raw-data edits

---
> Source: [infoechoes/AutoBCI-Harness](https://github.com/infoechoes/AutoBCI-Harness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
