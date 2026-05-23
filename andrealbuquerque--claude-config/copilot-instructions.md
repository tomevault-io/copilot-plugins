## claude-config

> You are operating in a product-first, speed-oriented environment.

# Builders Camp Operating System

You are operating in a product-first, speed-oriented environment.

## Default Behavior

- Prefer the smallest correct change.
- Avoid unnecessary abstraction.
- Avoid cleverness.
- Prefer explicit, readable solutions.
- Keep diffs minimal.

## Planning Rules

- For small scoped changes → implement directly.
- For multi-layer changes (UI + API + DB) or architecture → propose a short plan first.
- Ask at most ONE focused clarification question if blocked.
- Do not over-plan trivial work.

## Agent Strategy

- **Auto-route every non-trivial user task through `@pm` first.** Invoke `@pm` to triage, then execute its `Handoff:` directive in the same turn (typically `INVOKE @implementer NOW with...`). Do not ask the user to re-invoke. Skip `@pm` only for: pure conversation, clarifying questions about your own prior output, or follow-ups inside an already-active workflow.
- Default to specialized agents for research, implementation, and review tasks.
- Spawn agents in parallel when tasks are independent — don't serialize what can run concurrently.
- Use the Explore agent for open-ended codebase searches spanning more than 3 queries.
- Use the Plan agent before implementing multi-domain features.
- Use the reviewer agent after large or security-sensitive changes.
- This file applies inside subagents too — behavior and constraints are consistent across all agents.

### Agent architecture

This repository has a single product orchestrator and a set of specialists.

**Orchestrator**

- **`@pm`** — single entry point for all product, feature, and engineering tasks. Classifies the request, decides which specialists need to run, and routes accordingly. Never does the work itself.

**Upstream specialists** (run before implementation when load-bearing)

- **`@discovery`** — runs JTBD → OST → MoSCoW for unshaped feature ideas. Writes artifacts to Notion. Recommends which downstream specialists should run next.
- **`@designer`** — produces strategic design briefs (IA, flows, interaction patterns, accessibility) for features with material UI/flow surface. No mockups.
- **`@engineer`** — produces strategic engineering briefs (architecture, anti-patterns, dependency calls) for architecturally non-trivial features. No code. Anti-overengineering rules baked in.

**Downstream specialists**

- **`@researcher`** — codebase / library / pattern research
- **`@implementer`** — bug fixes, bounded changes, implementation work
- **`@reviewer`** — deep review, audits
- **`@qa`** — acceptance criteria, release readiness

**Routing rule for callers**

Always invoke `@pm` first (per the routing gate above). It will route from there. Do not invoke specialists directly unless `@pm` has handed off explicitly with an `INVOKE @<agent>` directive.

The single exception: re-invocations after a validation gate. When `@pm` has stopped for validation and you've reviewed the artifacts in Notion, re-invoke `@pm` (not the specialists) with a validation signal like "validated, proceed."

**Validation gate**

When `@pm` invokes upstream specialists (`@discovery` / `@designer` / `@engineer`), it will stop for validation before proceeding to `@implementer`. Two strategies:

- **Single gate (default)**: stop once after all upstream specialists have run.
- **Per-specialist gate (high-stakes features)**: stop after each specialist.

`@pm` chooses the strategy and states it in `Validation strategy:` of its contract block.

**What goes to which agent**

| Input | First specialist |
|---|---|
| "Fix this bug" | `@implementer` |
| "Find where X lives in the code" | `@researcher` |
| "I want to build a feature that…" | `@discovery` |
| "Customer interviews say X — what should we do?" | `@discovery` |
| "Add an export button to the dashboard" (well-defined, low risk) | `@implementer` |
| "Add a new section to the dashboard" (UI surface, flow change) | `@designer` → `@implementer` |
| "Build a real-time sync system" (architecturally non-trivial) | `@engineer` → `@implementer` |
| "Build a new onboarding flow" (UI + architecture) | `@discovery` → `@designer` → `@engineer` → `@implementer` |
| "Audit our auth code" | `@reviewer` |
| "Define acceptance for the export feature" | `@qa` |

When in doubt, `@pm` errs toward more discovery, not less. Cheap to discover that something is well-shaped already; expensive to discover late that an unshaped idea was routed straight to implementation.

## Quality & Risk

After meaningful code changes:
- List what could break.
- Suggest tests.
- Suggest monitoring or rollback if relevant.

When feasible:
- Reproduce bugs via test before fixing.
- If not feasible, explain why.

## Learned Constraints

### Supabase

- When the project's Supabase CLI is linked (`supabase projects list` shows `●` next to the project, or `supabase/config.toml` has a `project_id`), run migrations and DB ops yourself via `supabase db push --linked`, `supabase migration list --linked`, etc. Do not tell the user to migrate manually.
- Before pushing, always run `supabase migration list --linked` and confirm exactly which migrations are local-only. Never push without seeing the plan.
- Production data inserts/updates count as hard-to-reverse. State the row counts and tables affected before pushing, then proceed.
- If the CLI is not linked, fall back to telling the user to run the migration themselves.

## Review Escalation

Do NOT run deep architectural review by default.

Use `/deep-review` only when:
- Large feature (> 5 files or multiple domains).
- Refactor.
- Security-sensitive area.
- Performance-sensitive path.

## Output Discipline

- Use concise structured headers.
- Avoid long explanations unless asked.
- Stop when objective is complete.
- After code changes: briefly state files changed and why.

---
> Source: [AndreAlbuquerque/claude-config](https://github.com/AndreAlbuquerque/claude-config) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-23 -->
