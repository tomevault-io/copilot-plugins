## privacy-diff

> Default routing — Superpowers (workflow) + Agency (execution) for Privacy Diff


# Routing

- **First substantive turn** of a new development thread (before planning or editing files): load **using-superpowers** from the Superpowers plugin (or, if the repo has it, `.claude/skills/using-superpowers/`) so subsequent skills are invoked in the right order.
- **Default for every task** (feature, bugfix, idea, doc, refactor, review): use **Superpowers** skills for workflow phases; use an **Agency** specialist role for execution. Override only when the user says so.
- Bug, error, or failing test → debugging (Superpowers) then implementation/testing roles (Agency).
- Idea or brainstorm → brainstorming + product/planning before implementation.
- Add a feature → plan (e.g. writing-plans) then implement and verify (TDD when a harness exists).
- Review or ship → review/verification skills plus reviewer (or security) role as fits.

## PLANNING RULE (non-negotiable)

**Every plan task MUST include an Agency role before the plan is emitted.** Do not output a plan that has tasks without an assigned `agency` field.

Required task schema — every row must carry all five fields:

| Field | Example |
|-------|---------|
| `path` | `src/modules/reports/ReportService.ts` |
| `intent` | Add monthly profit aggregation method |
| `verify` | `npm test` passes / observable outcome |
| `agency` | `@agency-backend-architect.mdc` |
| `docs` | `docs/CONVENTIONS.md`, `docs/API-PATTERNS.md` |

The `docs` field lists which reference files the implementer must read before starting the task. Map using: `src/` work → `docs/CONVENTIONS.md`; HTTP/API → `docs/API-PATTERNS.md`; errors → `docs/ERROR-HANDLING.md`; auth/secrets → `docs/SECURITY.md`; tests → `docs/TESTING-STRATEGY.md`; architectural change → `docs/ARCHITECTURE.md`. A task with no `docs` field is incomplete.

**Superpowers template override for planning:** When Superpowers `writing-plans` skill is active it does not include an `agency` field — you MUST add it to every task before emitting the plan. Map using `.ai/agents.md` tables (API/server → `@agency-backend-architect.mdc`, UI → `@agency-frontend-developer.mdc`, tests → `@agency-api-tester.mdc`, infra → `@agency-devops-automator.mdc`, etc.). A plan with tasks that have no `agency` field is incomplete — do not present it to the user.

## DISPATCH RULE (non-negotiable)

**Before dispatching any implementation Task/subagent:** prepend the matching `@agency-<slug>.mdc` reference at the top of the task prompt. Do **not** use a generic "implementer" persona. The `@` reference is what loads the full specialist rule (persona, constraints, anti-patterns, deliverable templates) — it is not cosmetic.

| Task focus | Cursor rule |
|------------|-------------|
| API / backend / server logic | `@agency-backend-architect.mdc` |
| UI / frontend / components | `@agency-frontend-developer.mdc` |
| Tests / API testing | `@agency-api-tester.mdc` |
| Performance / benchmarking | `@agency-performance-benchmarker.mdc` |
| Infra / CI / scripts | `@agency-devops-automator.mdc` |
| System architecture | `@agency-software-architect.mdc` |
| Senior generalist dev | `@agency-senior-developer.mdc` |
| Code review | `@agency-code-reviewer.mdc` |
| Security review | `@agency-security-engineer.mdc` |
| Documentation | `@agency-technical-writer.mdc` |
| Product / requirements | `@agency-product-manager.mdc` |

> **Naming:** Cursor slugs come from each agent's frontmatter `name:` field (e.g. "Backend Architect" → `backend-architect`). Claude Code uses the upstream filename (`engineering-backend-architect`). They are different — do not mix them.

## SUPERPOWERS OVERRIDE (non-negotiable)

`.claude/skills/subagent-driven-development/implementer-prompt.md` instructs dispatching with `Task tool (general-purpose)`. In Cursor this means sending a task prompt with no specialist attached. **You MUST prepend the matching `@agency-<slug>.mdc` reference** at the top of every implementation task prompt — even when following Superpowers' template verbatim for the rest of the content.

Correct task prompt structure:

```
@agency-backend-architect.mdc

You are implementing Task N: [task name]
[full task text + context from Superpowers implementer-prompt.md template]
```

- The Superpowers body (task text, context, self-review, status reporting) stays unchanged.
- Only the persona binding (`@agency-*.mdc`) is added on top.
- Superpowers' reviewer subagents (spec-compliance, code-quality) keep their Superpowers role — do **not** prepend Agency rules to those.
- Plan is silent on roles? Map using `.ai/agents.md` before dispatch.
- Missing rule? Run `npx ai-dev-setup init --vendor-only --force` to refresh.

"Sub-agent driven" = Superpowers transport (isolation, reviews, todos). Persona = Agency specialist. **Both always apply.**

---
> Source: [fahadsubzwari924/privacy-diff](https://github.com/fahadsubzwari924/privacy-diff) — distributed by [TomeVault](https://tomevault.io/claim/fahadsubzwari924).
<!-- tomevault:4.0:copilot_instructions:2026-04-18 -->
