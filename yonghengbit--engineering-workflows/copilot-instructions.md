## engineering-workflows

> This repository is a workflow library, not an application.

# AGENTS.md

This repository is a workflow library, not an application.

## Editing Principles

- Keep `skills/` limited to the canonical, installable `engineering-workflow` skill.
- Keep incomplete or exploratory workflow designs under `proposals/` and never place a `SKILL.md`
  there.
- Put primary-intent workflow rules under `skills/engineering-workflow/references/<intent>/`.
- Keep the top-level `SKILL.md` limited to routing, progressive loading, transitions, and common
  execution principles.
- Keep repository-specific engineering conventions out of reusable workflow references.
- Avoid compatibility wrappers unless a real compatibility burden is documented; wrappers must not
  duplicate workflow rules or become alternate default entry points.

## Architecture Rules

Use these layers:

```text
repository constraints
    -> canonical engineering-workflow entry
    -> primary-intent routing
    -> one selected workflow
    -> workflow-specific scale/strategy
    -> execution and verification
    -> optional objective transition
```

Task type and task scale are separate concepts.

- `Development` is a primary intent; `LARGE` is a Development scale.
- `Testing` uses `QUICK / STRUCTURED / VALIDATION`.
- `Debugging` is evidence-driven and uses `DIRECT / SYSTEMATIC`.
- Investigation and Review do not need artificial scale taxonomies.

Supporting activity does not change workflow ownership. Transition only when the primary deliverable
changes.

## Adding a Workflow

Before adding `references/<intent>/workflow.md`:

1. define its primary intent and owning question;
2. define entry conditions and adjacent non-goals;
3. define strategy distinctions only when they change execution;
4. define proportional artifacts;
5. define re-evaluation and exit conditions;
6. define allowed objective transitions;
7. test representative and failure scenarios;
8. update the top-level routing table only if this is a new primary intent.

Use `docs/workflow-contract.md` as the authoring contract. Do not create another top-level skill for
a primary workflow.

## Router Rule

The routing policy in `skills/engineering-workflow/SKILL.md` may:

- classify current primary intent;
- load exactly one workflow reference;
- coordinate a transition when the primary objective changes.

It must not:

- contain detailed Development, Testing, Debugging, Performance, Investigation, or Review procedure;
- decide workflow-specific scale, artifacts, commands, tests, or subagents;
- spawn a router agent or pre-load all workflows;
- turn every mixed task into a heavyweight multi-workflow process.

## Documentation

- Keep `README.md` concise and user-oriented.
- Put full user instructions in `docs/usage.md`.
- Put architecture rationale in `docs/architecture.md`.
- Put workflow-author rules in `docs/workflow-contract.md`.
- When documentation and active skill content disagree, inspect code and observed behavior, then fix
  stale documentation.

---
> Source: [yonghengbit/engineering-workflows](https://github.com/yonghengbit/engineering-workflows) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
