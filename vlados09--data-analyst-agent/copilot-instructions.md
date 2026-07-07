## data-analyst-agent

> Spec-driven, stripped-down [spec-kit](https://github.com/github/spec-kit) style:

# CLAUDE.md

## Workflow

Spec-driven, stripped-down [spec-kit](https://github.com/github/spec-kit) style:

1. `/specify <feature>` — write `specs/NNN-slug/spec.md` (requirements + acceptance
   criteria, no design or code).
2. `/plan` — write `specs/NNN-slug/plan.md` (approach, design decisions) against the
   spec and `.specify/memory/constitution.md`.
3. `/tasks` — write `specs/NNN-slug/tasks.md` (checkable units of work) and implement
   them one at a time.

Code changes should reference the spec/task they satisfy. If you deviate from the spec,
plan, or the constitution, write a new ADR in `docs/decisions/` explaining why (use
`docs/decisions/0001-record-architecture-decisions.md` as the template) — ADRs cover
cross-cutting technical decisions; `.specify/` covers feature-level requirements.

Before considering a task done, all quality gates must pass: `make check`.

## AI-usage logging

Every prompt is captured mechanically by a `UserPromptSubmit` hook into
`docs/ai/prompt-log.jsonl` — do not edit that file by hand.

After each substantive task (a feature, fix, or non-trivial refactor — not every single
tool call), append one entry to `docs/ai/ai-log.md`:

- Date
- One line: what was asked
- What you (the AI) produced
- What the user accepted, edited, or rejected

## Project structure

- `src/data_analyst_agent/graph/` — LangGraph graph and node implementations
- `src/data_analyst_agent/stores/` — golden-query bucket, user prefs, saved reports
- `src/data_analyst_agent/safety/` — guardrails, PII masking
- `src/data_analyst_agent/observability/` — logging/tracing
- `infra/` — minimal prototype Terraform (see `infra/README.md`)
- `specs/` — spec-kit-style feature specs, plans, and tasks
- `.specify/` — spec-kit constitution and templates
- `docs/` — design (HLD), ADRs, AI-usage logs

---
> Source: [Vlados09/data-analyst-agent](https://github.com/Vlados09/data-analyst-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
