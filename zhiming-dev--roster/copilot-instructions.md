## roster

> Roster is a **human-in-the-loop, hierarchical multi-agent framework** for software-engineering and

# Copilot Instructions — Roster

Roster is a **human-in-the-loop, hierarchical multi-agent framework** for software-engineering and
workflow automation. The repo is **markdown-first**: agents and skills are plain markdown files with
YAML frontmatter, and runs are materialized on disk under `runs/<run-id>/`. An optional Python
**runtime** turns those markdown agents into live processes.

Read [AGENTS.md](../AGENTS.md) first — it is the canonical entry point. This file adds the
build/style conventions that AGENTS.md does not cover.

## Project Layout

- `planner-agent/`, `coder-agent/`, `qa-agent/`, `reviewer-agent/`, `researcher-agent/`,
  `e2e-agent/` — one folder per agent. Each holds a `<role>.agent.md` definition plus a
  `<role>-runner/SKILL.md` runbook.
- `shared/` — the contracts every agent depends on:
  - `schemas/` — JSON Schemas (`Plan`, `Task`, `ActionProposal`, `AgentMessage`,
    `ProvenanceEvent`, `CapabilityGrant`). Source of truth for every JSON artifact.
  - `approval-gate/`, `provenance/`, `council/` — cross-cutting skills.
  - `skills.registry.yaml` — the skill registry.
- `runtime/` — optional Python service (FastAPI) that runs the agents against Ollama / Azure AI
  Foundry and serves a dashboard at <http://localhost:8765/>.
- `runs/` — materialized runs (gitignored; principal- and machine-specific).
- `research/` — reproducibility harness, task suite, and pre-registration templates.
- `starter-packs/` — capability bundles and the portable Copilot core template.
- `examples/` — worked end-to-end examples.

## Build and Run

The runtime is the only executable component (Python 3.10+, FastAPI + uvicorn + httpx + pyyaml):

```powershell
cd runtime
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
python -m roster          # serves http://localhost:8765/
```

There is no automated test suite. The quality gate for JSON artifacts is **schema validation**
against `shared/schemas/*.schema.json`. Validate any `Plan`, `Task`, `ActionProposal`,
`AgentMessage`, or `ProvenanceEvent` you produce or edit.

## Repository-Specific Rules

- **Markdown-first.** Agents are `<role>.agent.md` with frontmatter `description` + `tools`.
  Skills are `SKILL.md` with frontmatter `name`, `description`, `argument-hint`. Keep this shape.
- **Schemas are authoritative.** Any schema change must update `shared/schemas/*.schema.json`,
  bump the schema `$id` version, and call out the breaking change in the PR description.
- **Never skip the HITL gate.** The Planner stops at `plan.draft.json` for human ratification.
  Do not plan-and-execute irreversible work directly — the approval gate is the point of the repo.
- **Least privilege.** Each task gets the minimum capability set and a conservative risk tier
  (T0–T4). When unsure, up-tier — the cost is one approval click.
- **Stable, prefixed ids.** `plan_…`, `task_…`, `msg_…`, `evt_…`, `prop_…`, `run_…`. URL-safe.
- **All times are ISO-8601 UTC.**
- **Provenance is append-only.** Never rewrite an in-progress run's `provenance.jsonl`.

## Read-Only Zones (do not edit unless explicitly asked)

- `e2e-agent/e2e-test/test-definitions/**` and `e2e-agent/e2e-test/suites/**` — user test fixtures.
  Let tests fail; never silently edit them to match the app.
- `examples/**/provenance.sample.jsonl` — immutable reference traces.
- `runs/<id>/provenance.jsonl` of an in-progress run — append only.

## Change Quality Bar

- Favor minimal, reviewable diffs. No unrelated refactors in the same change.
- Don't fix "bonus" bugs you spot — note them as followups instead. Silent scope expansion is
  exactly the surface area this framework bounds.
- Search the codebase before assuming; ground changes in existing patterns.

## Security and Privacy

- Never hardcode secrets, keys, tokens, or credentials. The live `runtime/agents.config.yaml`
  holds real endpoints/keys and is gitignored — only the `*.example.yaml` template is committed.
- Avoid logging sensitive user or production data.

---
> Source: [zhiming-dev/Roster](https://github.com/zhiming-dev/Roster) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-06 -->
