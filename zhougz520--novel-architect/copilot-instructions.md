## novel-architect

> Repository-level instructions for AI coding agents working on Novel Architect.

# AGENTS.md

Repository-level instructions for AI coding agents working on Novel Architect.

This file is deliberately practical and short. Put always-on repo rules here; put user-facing explanations in `README.md` / `docs/`; put runtime skill behavior in `skill/novel-architect/SKILL.md` and `skill/novel-architect/references/`.

## 1. Scope and precedence

- This root `AGENTS.md` applies to the whole repository unless a deeper `AGENTS.md` / `AGENTS.override.md` provides more specific guidance.
- The user’s current explicit request takes precedence over this file when they conflict.
- Keep this file focused on durable repository behavior: layout, commands, invariants, tests, docs sync, and “done” criteria.

## 2. Project mission

Novel Architect is an agent skill for Chinese commercial serial web-novel production. The product direction is not “generate more text faster”; it is a hybrid system that helps an agent plan, write, review, repair, and continue long-form fiction without losing reader pull, visible payoff, character voice, logic, or continuity.

Protect these product goals in every change:

- reader pull and turn-page motivation;
- visible payoff and enemy loss;
- distinct character voice;
- coherent long-run state, promises, costs, foreshadowing, and enemy threads;
- strict gates before commit;
- correct division between deterministic scripts and agent literary judgment.

## 3. Source of truth map

- `skill/novel-architect/` is the release skill root.
- `skill/novel-architect/SKILL.md` is the runtime agent playbook.
- `skill/novel-architect/references/` contains runtime specifications.
- `skill/novel-architect/src/novel_architect/` is the Python deterministic tool layer.
- `tests/` contains the regression suite.
- `docs/` is for users and contributors.
- Root `README.md` is the English public project landing page.
- Root `README.zh-CN.md` is the Simplified Chinese public project landing page.
- Root `AGENTS.md` is for coding agents maintaining this repository.

If implementation, docs, and references disagree, inspect the code and tests, then update the stale document. Do not leave drift behind.

## 4. Architecture invariants

Do not weaken these without explicit user approval:

- Main chapter loop remains `PREPARE → WRITE → REVIEW → COMMIT`.
- Signal builders in `signals/` remain deterministic Python. Do not add LLM/API calls inside signal builders.
- Model-produced signal results are saved as model variants and merged with heuristic signals by strictest result; model `pass` must not override heuristic `warn` or `fail`.
- `data/` is derived state. Do not ask users or tests to manually edit `data/` to fix story state.
- Persistent story truth belongs in `src/` or validated backwrite from `manuscript/`.
- First-three review remains a production gate for chapter 4+: all required scores present, numeric, in range, high enough, with current `review_input_hash` and `bulk_continue_allowed=true`.
- Reader promise lifecycle remains automatic through chapter memory backwrite + recompute.
- `gate.passed=false` means do not commit in normal production workflows.

## 5. Setup and command policy

Run from repository root unless a task explicitly says otherwise.

Preferred commands:

```bash
uv sync --group dev
uv run python -m novel_architect --version
uv run python -m novel_architect --help
bash scripts/format.sh
bash scripts/lint.sh
bash scripts/test.sh
python tools/final_merge_probe.py .
```

Notes:

- This is a Python 3.12 project managed with `uv`.
- Do not introduce Node/npm/pnpm/yarn unless the user explicitly asks for a new frontend/tooling layer.
- Do not install global dependencies or modify user/system shell configuration.
- If `review` or `save-first3-review` returns non-zero, treat it as a gate failure unless stderr shows a real crash.

## 6. Change discipline

When changing Python behavior:

- Add or update the closest unit/integration test.
- Run targeted tests first, then the full suite when feasible.
- Check whether `SKILL.md`, `references/`, `README.md`, or `docs/` need updates.

When changing CLI commands, workflow files, runtime artifacts, or project schema:

- Update `docs/reference/cli.md` and `docs/reference/project-files.md`.
- Update `skill/novel-architect/SKILL.md` if runtime agent behavior changes.
- Update tests that assert paths, commands, gates, or schemas.

When changing signals, gate behavior, or model-signal parsing:

- Update relevant `tests/unit/test_signal_*`, `tests/unit/test_review_gate.py`, and integration tests.
- Update `docs/reference/signals-and-gates.md` and `skill/novel-architect/references/review-spec.md`.

When changing docs only:

- Verify command names, file paths, signal names, and gate semantics against code.
- Avoid copying outdated examples from old docs.

## 7. Documentation policy

Use progressive disclosure:

- README: public landing page, value proposition, quick start, links.
- docs/guides: task-oriented workflows for users.
- docs/reference: factual command/file/signal reference.
- docs/architecture: conceptual explanation.
- docs/development: contributor workflow.
- SKILL.md: runtime instructions for agents using the skill.
- AGENTS.md: repository maintenance rules for coding agents.

Do not duplicate long runtime specs in AGENTS.md. Link to the appropriate document instead.

## 8. Ask first

Ask the user before:

- replacing the core pipeline architecture;
- adding production dependencies;
- changing release packaging boundaries;
- weakening first3/gate/state_guard/commercial signal requirements;
- deleting tests or references;
- converting literary judgment that currently belongs to an agent into pure keyword scoring, or converting deterministic state checks into free-form generation.

## 9. Never

- Never use destructive git commands on user work unless explicitly requested.
- Never bypass gates by editing `gate.json`, `signals/`, or `data/` directly.
- Never claim a change is verified if tests or probes were not run; state what was and was not run.
- Never ship docs that instruct users to manually run routine skill CLI steps when the runtime agent should execute them.
- Never include font files in generated artifacts or release bundles.

## 10. Definition of done

A task is done when:

- requested behavior is implemented;
- relevant tests pass or unrun checks are clearly reported;
- docs and references are synchronized where behavior changed;
- no stale wording contradicts current gates, signal merge, first3 hash rules, or reader promise lifecycle;
- `git diff` contains only intentional changes;
- final response summarizes files changed, checks run, and remaining risks if any.

---
> Source: [zhougz520/novel-architect](https://github.com/zhougz520/novel-architect) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-09 -->
