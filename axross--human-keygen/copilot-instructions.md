## human-keygen

> Apply these keywords consistently in this document and the documents linked from this document.

# AGENTS.md

## Requirement Level Keywords

Apply these keywords consistently in this document and the documents linked from this document.

| Keyword | Synonym | Meaning |
| ------- | ------- | ------- |
| "MUST" | "REQUIRED" | Non-negotiable requirement; no exceptions. |
| "MUST NOT" |  | Non-negotiable prohibition; no exceptions. |
| "SHOULD" | "RECOMMENDED" | Strongly preferred; deviation is allowed only after weighing the implications. |
| "SHOULD NOT" | "NOT RECOMMENDED" | Strongly discouraged; allowed only after weighing the implications. |
| "MAY" | "OPTIONAL" | Genuinely optional; no preference implied. |

## Project Overview

- This is a web application project for generating human-friendly passwords.
- The target users are people who use non-QWERTY keyboard layouts.
- Generated passwords use only characters that are common to both the QWERTY keyboard layout and the user's selected keyboard layout.
- Passwords should remain practical to type on the user's keyboard while preserving clear generation rules.
- For tech stack, deployment target, npm run-scripts, and durable directory placement, consult [Project Structure](.agents/skills/project-structure/SKILL.md) and [Development Guidelines](.agents/skills/development-guidelines/SKILL.md).

## Skill Index

`AGENTS.md` is the master routing index for project skills. Consult the relevant skill before acting on matching work.

| Skill | When to apply |
| ----- | ------------- |
| [Agent Skills Best Practices](.agents/skills/agent-skills-best-practices/SKILL.md) | Creating, refining, splitting, renaming, deleting, or auditing project skills or this skill index |
| [Application Security Requirements](.agents/skills/application-security-requirements/SKILL.md) | Reviewing generated password exposure, Web Crypto, clipboard use, storage, URLs, telemetry, dependencies, environment variables, or privacy-sensitive behavior |
| [Code Review Guideline](.agents/skills/code-review-guideline/SKILL.md) | Reviewing a diff, pull request, local change, or post-implementation self-review |
| [Development Guidelines](.agents/skills/development-guidelines/SKILL.md) | Starting any task; implementing, refactoring, running commands, checking current docs, adding dependencies, changing verification behavior, writing commit messages, or opening pull requests |
| [E2E Testing Guidelines](.agents/skills/e2e-testing-guidelines/SKILL.md) | Writing, running, reviewing, or maintaining Playwright tests for generator workflows, clipboard feedback, layout selection, or responsive behavior |
| [Keyboard Layout Requirements](.agents/skills/keyboard-layout-requirements/SKILL.md) | Adding, editing, testing, or reviewing keyboard layout data, QWERTY intersections, character categories, or layout metadata |
| [Maintainable Code Guidelines](.agents/skills/maintainable-code-guidelines/SKILL.md) | Reviewing readability, naming, abstraction boundaries, complexity, dead code, or scope discipline |
| [Observability Guidelines](.agents/skills/observability-guidelines/SKILL.md) | Throwing, catching, reporting, or logging errors; console output; browser failure messages; telemetry boundaries; or debugging generated-password failures |
| [Password Generation Requirements](.agents/skills/password-generation-requirements/SKILL.md) | Implementing, testing, reviewing, or changing password generation, entropy, randomness, grouping, character pools, or output rules |
| [Performance and Reliability Requirements](.agents/skills/performance-and-reliability-requirements/SKILL.md) | Reviewing bundle weight, dependency restraint, browser API failure modes, local responsiveness, or runtime reliability |
| [Project Structure](.agents/skills/project-structure/SKILL.md) | Navigating the repository, locating files, placing modules, checking stack/deployment context, or updating durable directory conventions |
| [Quality Assurance Guidelines](.agents/skills/quality-assurance-guidelines/SKILL.md) | Reviewing verification evidence, test coverage, skipped checks, manual checks, lint/format proof, or residual risk |
| [React Component Guidelines](.agents/skills/react-component-guidelines/SKILL.md) | Writing, reviewing, or refactoring React components, Tailwind class usage, local state boundaries, accessible controls, or test locators |
| [Routing Guidelines](.agents/skills/routing-guidelines/SKILL.md) | Creating, moving, renaming, or reviewing TanStack Start routes, root metadata, router config, stylesheet links, or URL contracts |
| [UI Design Principles](.agents/skills/ui-design-principles/SKILL.md) | Designing, implementing, or reviewing user-facing surfaces, responsive behavior, accessibility, visual tone, copy, focus states, or feedback states |

## Response Approach

Use this workflow for single-agent work in this repository. The agent owns planning, implementation, investigation, verification, review, and reporting directly.

### Overall Strategy

Non-trivial work should move through the same decision sequence even when some steps are brief.

1. Classify the request and load the relevant project guidance.
2. Define success criteria, constraints, affected surface, dependencies, and verification expectations.
3. Inspect the smallest useful code and documentation context.
4. Draft an ordered local workflow with acceptance criteria.
5. Implement, investigate, or review within the narrowest scope that satisfies the request.
6. Self-review the result as a separate phase.
7. Run or report the relevant verification.
8. Update or propose skill guidance when the work exposes reusable project learning.
9. Summarize outcome, verification status, trade-offs, and open follow-ups.

**Guidelines:**

- MUST consult [Development Guidelines](.agents/skills/development-guidelines/SKILL.md) at the start of every task.
- MUST classify non-trivial work as UI-bearing, implementation-only, review-only, skill-maintenance, exploratory, or mixed workflow before editing files.
- MUST consult every skill whose routing condition matches the changed surface or requested review lens.
- MUST ask a concrete question when progress depends on a product, platform, privacy, compatibility, or scope decision that cannot be inferred from local context.
- SHOULD compress the sequence for small answer-only requests without skipping relevant safety checks.

### Planning and Execution

Planning exists to make the work checkable. It should name what changes, what must stay unchanged, and how the result will be verified.

**Guidelines:**

- MUST restate success criteria, constraints, affected surface, and verification expectations before non-trivial edits.
- MUST preserve public behavior during refactors unless the requested change intentionally modifies it.
- MUST keep edits scoped to the smallest surface that satisfies the acceptance criteria.
- SHOULD inspect independent discovery targets in parallel when their outputs do not depend on each other.
- SHOULD revise the plan when new evidence changes affected files, risks, or acceptance criteria.

### UI-Bearing Work

User-facing changes need design intent before implementation mechanics. The single agent owns both, but the phases must stay distinct.

**Guidelines:**

- MUST establish design intent before implementing UI-bearing changes: hierarchy, interaction states, accessibility intent, responsive behavior, and copy constraints.
- MUST consult [UI Design Principles](.agents/skills/ui-design-principles/SKILL.md) for design decisions and [React Component Guidelines](.agents/skills/react-component-guidelines/SKILL.md) for implementation mechanics.
- MUST express design intent in user-facing terms before translating it into components, CSS, or tests.
- MUST verify that text, layout, focus behavior, loading states, and responsive behavior remain coherent across relevant viewports.
- SHOULD keep design-system rules in design vocabulary and link to implementation-mechanics skills instead of duplicating CSS wiring rules.

### Review Independence Gates

A single agent cannot provide true independent review. This repository compensates with a mandatory separate review phase for ordinary work and external review gates for high-risk work.

**Guidelines:**

- MUST perform a reviewer-mode reset after non-trivial implementation: stop editing, reread the request, inspect `git status` and `git diff`, and review only the produced diff.
- MUST apply [Code Review Guideline](.agents/skills/code-review-guideline/SKILL.md) during self-review, including severity labels, file-line evidence, concrete fixes, and an explicit verdict when findings exist.
- MUST load topic-specific review lenses when relevant: maintainability, quality assurance, security, performance/reliability, UI design, routing, observability, password generation, keyboard layouts, or e2e testing.
- MUST judge the actual diff and observed behavior, not the implementation intent.
- MUST fix Critical or Major self-review findings before claiming completion.
- MUST perform a second-pass re-review after fixing any blocking self-review finding.
- MUST report verification evidence before completion: commands run, manual checks, failures, skipped checks, and residual risk.
- MUST escalate high-risk changes to user review, CI/PR review, or an explicitly requested secondary review before calling them merge-ready.
- SHOULD treat generated password exposure, randomness changes, clipboard behavior, dependency additions, telemetry/logging, public route contracts, production config, deployment config, and large refactors as high-risk.

### Verification

Verification should match the changed surface. Documentation-only changes need format and link checks; route, UI, password logic, layout data, tooling, deployment, and runtime changes need stronger evidence.

**Guidelines:**

- MUST run the relevant verification commands after non-trivial changes, or report why they could not run.
- MUST run `npm run format` and `npm run lint` after code or documentation edits.
- MUST run `npm run test:e2e` when a change affects a UI output surface or e2e coverage.
- MUST run `npm run test` when a change affects password generation, keyboard-layout data, entropy, random-source behavior, or shared utilities.
- MUST run `npm run build` when a change affects TanStack Start routes, route metadata, Vite/Tailwind configuration, Wrangler deployment config, dependencies, or TypeScript signatures.
- SHOULD perform focused manual checks when browser behavior, clipboard behavior, route metadata, responsive layout, focus states, or Cloudflare preview behavior changes.
- MUST report unverified acceptance criteria and residual risk in the final summary.

### Skill Maintenance

Skill maintenance keeps reusable workflow learning close to the project rules. It should happen when a change reveals durable guidance, not after every narrow fix.

**Guidelines:**

- MUST consult [Agent Skills Best Practices](.agents/skills/agent-skills-best-practices/SKILL.md) when adding, renaming, moving, deleting, splitting, or cross-linking skills, changing reference files, or updating this index.
- MUST keep this skill index synchronized when skills are added, renamed, moved, or removed.
- MUST make one skill the source of truth for a rule instead of copying detailed guidance across multiple skills.
- SHOULD propose or implement skill updates when the workflow exposes a reusable convention, outdated guidance, recurring review issue, or missing project rule.
- SHOULD skip skill maintenance when the workflow produced no generalizable learning, and state that it was skipped.

### Communication

User-facing communication should expose decisions, blockers, verification, and outcomes without narrating every local inspection step.

**Guidelines:**

- MUST keep progress updates concise and focused on decisions, blockers, and outcomes.
- MUST summarize changed files, verification status, trade-offs, unresolved risks, and deferred follow-ups at completion.
- MUST state whether skill maintenance was performed, skipped, or blocked when skill guidance governed the work.
- SHOULD include detailed plans, command logs, or iteration logs only when the user asks for auditability or when the outcome depends on them.
- MUST ask a concrete question when progress depends on a product, platform, privacy, or scope decision.

---
> Source: [axross/human-keygen](https://github.com/axross/human-keygen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-12 -->
