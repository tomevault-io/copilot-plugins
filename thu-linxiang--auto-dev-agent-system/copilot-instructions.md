## auto-dev-agent-system

> This repository defines a file-based, validator-gated multi-agent development system. Keep each concern in its existing layer:

# Repository Guidelines

## Project Structure & Module Organization

This repository defines a file-based, validator-gated multi-agent development system. Keep each concern in its existing layer:

- `agents/`: agent roles, prompts, inputs, outputs, and shared contracts.
- `orchestrator/` and `workflows/`: Codebuddy stage control, mechanical execution rules, and YAML pipeline definitions.
- `schemas/`, `templates/`, `validators/`, and `gates/`: artifact contracts, rendering formats, deterministic checks, and semantic transition policies.
- `memory/`: persistent project state, decisions, issues, evaluations, and traces.
- `tests/fixtures/`: stable PRD inputs and expected artifacts; test categories live under `tests/`.
- `workspaces/`: per-project inputs, outputs, evidence, and state. The canonical workspace layout, ownership, and generated-data rules are defined in `docs/architecture/workspace-artifact-architecture.md`; do not introduce or assume a `sample-workspace/` directory.

Read `docs/architecture/README.md`, the relevant architecture owner document, and `PROJECT_PRINCIPLES.md` before changing a workflow boundary or state transition.

## Build, Test, and Development Commands

Install dependencies with:

```bash
pip install -r requirements.txt
```

Run setup and deterministic validation through Python (Windows-compatible):

```bash
python scripts/setup.py
python scripts/test.py
python scripts/validate.py
```

`test.py` runs deterministic repository tests. `validate.py` compiles Validator code and checks active schemas/workflows parse. Workflow start, stage execution, and resume are Codebuddy operation contracts in `commands/`, not automated shell runners. Legacy Bash scripts are not a required execution path.

## Coding Style & Naming Conventions

Preserve the existing file-oriented design. Use lowercase kebab-case for Markdown, YAML, and script filenames (for example, `requirement-breakdown-validator.md`); use `*.schema.json` for JSON Schemas and `*.template.md` for templates. Keep Python tooling Windows-compatible. `scripts/normalize_prd.py` is the active cross-platform converter; legacy Bash converters are not a required execution path. Use two-space indentation in YAML and JSON unless the surrounding file establishes a different convention. Keep requirement IDs and trace references stable (for example, `REQ-001`).

## Testing Guidelines

Add deterministic fixtures under `tests/fixtures/` and keep expected outputs reviewable as text. Organize tests by concern: `agent-contract-tests/`, `validator-tests/`, or `workflow-tests/`. Every new artifact type needs schema/validator coverage and traceability to its input requirements. Update `scripts/test.py` and `scripts/validate.py` as implementations become available.

## Commit & Pull Request Guidelines

Use concise imperative subjects such as `Add PRD breakdown validator`. PRs should state the affected pipeline stage, list changed artifacts/schemas/validators, link the related issue or requirement IDs, and include validation evidence. Call out any gate, state, or traceability impact explicitly.

## Configuration & Generated Data

Copy `.env.example` to `.env` for local configuration; never commit secrets. Generated logs and per-workspace output/state are ignored by design. Keep durable system knowledge in the tracked `memory/` documents, not chat-only notes.

---
> Source: [THU-linxiang/auto-dev-agent-system](https://github.com/THU-linxiang/auto-dev-agent-system) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-24 -->
