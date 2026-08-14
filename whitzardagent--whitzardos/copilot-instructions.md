## whitzardos

> This file is the durable working agreement for AI coding agents operating in this repository.

# QitOS AGENTS.md

This file is the durable working agreement for AI coding agents operating in this repository.

Keep this file high-signal:
- put repository-wide rules here
- put directory-specific rules in nested `AGENTS.md` or `AGENTS.override.md`
- prefer concrete commands, constraints, and acceptance criteria over slogans

---

## Mission

You are the coding agent for QitOS, a research-first, builder-friendly agent framework centered on one canonical kernel:
- `AgentModule + Engine`
- explicit lifecycle: `observe -> decide -> act -> reduce -> check_stop`

Your job is not only to ship correct code, but also to make project progress visible, reviewable, and easy for users and contributors to follow.

The quality bar is not MVP. Changes should move QitOS toward world-class open-source framework quality in:
- architecture clarity
- modularity and extensibility
- reproducibility and observability
- developer ergonomics
- documentation quality

---

## Primary goals

Optimize for the following, in order:

1. Correctness
2. Clarity
3. Consistency with the existing codebase
4. Reproducibility and maintainability
5. Visible project momentum for users and contributors

Do not optimize for speed at the expense of quality.

---

## Default working style

- Be proactive and execution-oriented.
- Gather the necessary context from the repository before editing.
- Follow existing patterns, naming, abstractions, and conventions unless there is a strong reason to improve them.
- Prefer small, coherent, reviewable changes over scattered hacks.
- Solve the root problem, not just the immediate symptom.
- When changing behavior, make sure all related surfaces remain consistent: code, tests, docs, examples, changelog, and README-facing project updates.

Do not stop at "the code compiles".
A task is only complete when implementation, verification, and repository-facing communication are all complete.

---

## Architecture invariants

These are non-negotiable:

- Keep a single mainline architecture. Do not introduce parallel architecture tracks.
- Do not create `V1`, `V2`, `Legacy`, `Next`, or alias-based duplicate concepts in core APIs.
- Keep stable contracts in `qitos.core`; put replaceable concrete implementations in `qitos.kit`.
- Preserve the `AgentModule + Engine` story as the primary public mental model.
- Prefer explicit contracts and hook points over hidden magic.
- Do not reduce trace clarity, stop-reason clarity, or `qita` replay/export usefulness.

---

## Package boundaries

Use these boundaries strictly:

- `qitos.core`: abstract contracts, canonical data types, stable framework primitives
- `qitos.engine`: execution kernel, loop mechanics, hooks, validation, recovery, stop logic, action execution
- `qitos.kit`: concrete reusable implementations such as tools, memory, parser, planning, critic, env helpers, prompts
- `qitos.benchmark`: adapters that turn external benchmarks into canonical `Task` (deprecated, migrating to recipes/)
- `examples`: runnable reference agents and benchmark runners
- `docs`: educational and operational documentation

Rule of thumb:
- if it is concrete or swappable, prefer `qitos.kit`
- if it is a stable contract, keep it in `qitos.core`

---

## Planning rules

For simple changes, proceed directly after gathering enough context.

For larger tasks, create or update a written execution plan before major implementation work begins.

Use a plan when any of the following is true:
- the task spans multiple files or subsystems,
- the task will likely take more than 30 minutes,
- the task involves architecture, refactors, benchmarks, or public API changes,
- the task has non-trivial product or documentation implications.

When a plan is needed:
- create or update a task-specific plan document,
- make the plan concrete and executable,
- keep the plan updated as the work evolves,
- treat the plan as a living document, not a one-time sketch.

---

## Code quality rules

- Prefer existing helpers and patterns over introducing new abstractions.
- Do not duplicate logic if a reusable internal abstraction already exists.
- Keep functions and modules focused.
- Avoid speculative generalization.
- Avoid broad try/catch blocks and silent failures unless the repository already uses them intentionally.
- Surface errors clearly and follow existing error-handling patterns.
- Keep types strong; do not use unsafe casts unless absolutely necessary and justified.
- Avoid adding production dependencies unless clearly necessary.

When introducing a new abstraction, ensure it earns its complexity.

---

## Verification rules

For every meaningful code change, you must do the relevant verification work.

This includes, as applicable:
- updating or adding tests,
- running the relevant test suites,
- running lint / formatting / type checks,
- checking that behavior matches the request,
- reviewing your own diff for regressions, inconsistencies, or overreach.

Default project validations:

```bash
pytest -q
```

Stable-surface static checks:

```bash
flake8 qitos/core qitos/engine qitos/models qitos/trace
mypy qitos/core qitos/engine qitos/models qitos/trace
```

Packaging checks when changing packaging, distribution, or release-facing behavior:

```bash
python -m build
python -m twine check dist/*
```

Do not claim success without verification.
If you cannot run a check, explicitly say so and explain why.

---

## Tooling and contract rules

- Class-based tools should implement `execute(args, runtime_context)`.
- `run(...)` exists as a compatibility path, not as the preferred new contract.
- Function-style tools should continue to use the canonical decorator path.
- Tool behavior should remain composable through `ToolRegistry`.
- Env-backed operations should consume env ops rather than assuming host filesystem/process access directly.

---

## Observability and reproducibility

Do not ship changes that degrade:

- trace schema consistency
- hook payload usefulness
- `run_id`, `step_id`, and `phase` clarity
- replayability through `qita`
- final result and stop reason auditability

Every major feature should preserve or improve observability.

---

## Documentation and project-history rules

These rules are mandatory.

### 1. CHANGELOG discipline

For every meaningful change, update `CHANGELOG.md`.

Default behavior:
- add an entry under the appropriate `Unreleased` section,
- describe the change in user-facing language,
- mention the affected area clearly,
- keep entries concise but informative,
- prefer `Added`, `Changed`, `Fixed`, `Deprecated`, `Removed`, and `Breaking` categories.

You must update `CHANGELOG.md` for:
- new features, fixes, behavior changes, CLI changes,
- benchmark support changes, docs-visible workflow changes,
- developer-facing improvements, performance improvements, deprecations or removals.

Do not leave meaningful repository progress undocumented.

### 2. Docs discipline

Whenever behavior, APIs, workflows, architecture, examples, setup, benchmarks, or contributor expectations change, update `docs/` in the same task.

Default behavior:
- update the most relevant existing doc if one already exists,
- create a new doc only when the topic does not fit cleanly into existing docs,
- keep examples and commands accurate,
- keep terminology consistent with the codebase.

You must treat documentation updates as part of implementation, not as optional follow-up work.

### 3. README news discipline

The README must visibly communicate that the project is actively progressing.

For every meaningful user-visible, contributor-visible, or roadmap-relevant change:
- update the `News`, `What's New`, or equivalent section in `README.md`,
- add a short, high-signal entry describing the progress,
- prefer concise updates that help users immediately notice momentum.

### 4. Sync rule

Never finish a meaningful task without checking whether all three of the following need updates:
- `CHANGELOG.md`
- `docs/`
- `README.md` news / updates section

Default to **yes** unless the change is clearly too minor.

---

## Open-source maintenance rules

QitOS is an open-source project.
Work should leave behind signals that help external users and contributors understand project health and direction.

Whenever relevant:
- improve contributor clarity,
- improve discoverability of new functionality,
- improve tutorial quality,
- improve consistency between docs and code,
- improve release readability.

Think like a maintainer, not just an implementer.

---

## Safety and scope control

- Do not make unrelated drive-by changes unless they are necessary to complete the task safely.
- Do not rewrite large areas of the codebase without clear justification.
- Do not introduce hidden breaking changes.
- Call out migration or compatibility implications clearly.
- If the task reveals a larger issue, fix what is necessary now and note the broader follow-up separately.
- Never use destructive git commands such as `git reset --hard` or `git checkout --` unless explicitly requested.
- Do not amend commits unless explicitly requested.

---

## Codex / MCP best practices

When using Codex in this repository:

- Give the agent concrete context: target files, expected behavior, constraints, and validation commands.
- Prefer durable guidance in `AGENTS.md` over repeating the same instructions in every prompt.
- Keep repository instructions concise and operational; add nested overrides only near specialized subsystems.
- Validate real outcomes after edits instead of stopping at analysis or code generation.
- Turn repeated workflows into reusable skills, scripts, or automation only after the workflow is stable.

For OpenAI-, ChatGPT-, or Codex-related questions:
- Always use the OpenAI developer documentation MCP server first if available.
- If MCP is unavailable, fall back only to official OpenAI docs domains.
- Do not rely on memory alone for volatile OpenAI product guidance.

---

## Benchmarks, examples, and docs rules

Benchmark rules:
- convert benchmark inputs into canonical `Task`
- keep benchmark-specific hacks out of core
- preserve useful raw fields in metadata
- keep adapters in `qitos.benchmark` (deprecated, migrating to `qitos.recipes`)
- provide runnable examples where practical

Example rules:
- examples are product surface, not toy snippets
- each example should run end-to-end on a real path
- examples should teach one clear pattern
- credentials must come from environment variables

Docs rules:
- update docs when public behavior, contracts, or user workflows change
- keep English and Chinese docs reasonably aligned when both exist
- prefer constructive walkthroughs over command dumps

---

## Preferred decision heuristic

When uncertain, choose the option that:

1. keeps `AgentModule + Engine` simpler
2. improves researcher iteration speed
3. improves traceability and debuggability
4. preserves modular extension through `qitos.kit`
5. avoids architecture forks and surface-area sprawl

If a proposal violates this file, revise the design before coding.

---
> Source: [WhitzardAgent/WhitzardOS](https://github.com/WhitzardAgent/WhitzardOS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
