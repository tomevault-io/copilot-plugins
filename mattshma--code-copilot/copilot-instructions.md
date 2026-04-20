## code-copilot

> > This file is the primary context source for AI review (claude-code-action / codex-connector) and local AI development.

# CLAUDE.md

> This file is the primary context source for AI review (claude-code-action / codex-connector) and local AI development.
> All AI participants must follow these standards. Human developers should as well.
>
> **Setup:** Copy this file to your project root as `CLAUDE.md` and customize each section for your project.

---

## Project Overview

<!-- Describe your project in 2-3 lines -->
- **Type:** <!-- e.g., Web app, CLI tool, mobile app, monorepo -->
- **Language:** <!-- e.g., TypeScript, Python, Go, Rust -->
- **Framework:** <!-- e.g., React, Next.js, FastAPI, none -->
- **Package Manager:** <!-- e.g., pnpm, npm, yarn, pip, cargo -->

---

## Project Structure

<!-- Describe your directory layout -->
```
src/
  ...

docs/specs/{feature}/       ← not committed to git
  design.md                 ← technical design (living doc, task checkboxes)
  review-design.md          ← design review log, append-only
  review-code.md            ← local code review log, append-only
  review-pr.md              ← PR review log, append-only
  decisions.md              ← implementation decision log, append-only
  flow.md                   ← flow pipeline state (for resumability)
```

---

## AI Role Assignment

<!-- Define which AI handles which copilot commands. Adjust based on your preference. -->

When running copilot commands, tasks are assigned as follows:

| Command | Assigned to | Notes |
|---------|-------------|-------|
| `copilot-branch` | Claude | Creates feature/fix branch from main |
| `copilot-design` | Claude | Creates technical design |
| `copilot-review-design` | Codex | Reviews design via `codex exec` |
| `copilot-dev` | Claude | Implements code |
| `copilot-ut` | Codex | Writes tests via `codex exec` |
| `copilot-check` | Claude | Runs local quality checks |
| `copilot-pr` | Claude | Creates PR via `gh` CLI |
| `copilot-review-fetch` | Claude | Fetches PR review comments |
| `copilot-review-fix` | Codex | Fixes review issues via `codex exec` |
| `copilot-flow` | **Claude** | Orchestrates full pipeline, delegates to agents per above |

When a task is assigned to an external agent, invoke it via the `copilot-agent` skill (`.claude/skills/copilot-agent/SKILL.md`).

> **Note:** All commands except `copilot-flow` can be assigned to any agent — Codex, Kimi, or any other supported agent. `copilot-flow` always runs in Claude as the orchestrator. Adjust the table above to match your preference.

---

## Tech Stack & Constraints

<!-- List your technologies and any constraints AI should know about -->

| Technology | Purpose | Notes |
|------------|---------|-------|
| <!-- e.g., React --> | <!-- e.g., UI framework --> | <!-- e.g., Function components only --> |
| <!-- ... --> | | |

---

## Coding Standards

<!-- Define your coding standards. Examples: -->

### General
- <!-- e.g., Use strict TypeScript, no `any` -->
- <!-- e.g., Prefer functional style over OOP -->
- <!-- e.g., All exports must have explicit types -->

### Naming Conventions

| Target | Style | Example |
|--------|-------|---------|
| <!-- e.g., Files (components) --> | <!-- e.g., PascalCase --> | <!-- e.g., UserProfile.tsx --> |
| <!-- ... --> | | |

---

## Banned Patterns

<!-- List patterns that are treated as blocking issues during review -->

1. <!-- e.g., `any` type — use `unknown` with type narrowing -->
2. <!-- e.g., `console.log` in production code — use project logger -->
3. Modifying or removing code marked with `IMPORTANT: Do not change` without explicit design review approval
4. <!-- ... -->

---

## Test Requirements

<!-- Define your testing standards -->
- <!-- e.g., New features must include tests -->
- <!-- e.g., Minimal mock principle -->

### Test Directory Structure

<!-- Define where test files live. Example for monorepo with packages/web, packages/api: -->
<!-- Each package has its own test/ directory mirroring src/:
     src/{module}/{file}.ts  →  test/{module}/{file}.test.ts
     e.g., src/auth/login.ts → test/auth/login.test.ts -->
<!-- Shared fixtures/helpers used by multiple packages go in packages/shared/test/ -->

---

## Git & PR Standards

**Commit messages:**
```
feat(scope): description
fix(scope): description
refactor(scope): description
```

**PR standards:**
- Title follows commit message format
- Description includes: what changed, why, attention points, how to test
- One PR does one thing

**Review standards (for AI reviewers):**
- Distinguish blocking (must fix) from non-blocking (suggestion)
- Cite specific rules from this file as rationale
- No pure style preference comments
- Suggest additions to this file when new patterns are discovered

---

## Architectural Decision Records

<!-- Record important decisions with rationale. AI reviewers use these as judgment basis. -->

| Date | Decision | Rationale |
|------|----------|-----------|
| | | |

<!-- After each feature merge, check if new decisions should be appended here -->

---

## Quality Check Commands

<!-- Define the actual commands used by /copilot-check and /copilot-pr -->

```bash
# Type check
# e.g., pnpm -r run typecheck
# e.g., tsc --noEmit
# e.g., mypy .

# Lint
# e.g., pnpm -r run lint
# e.g., eslint .
# e.g., ruff check .

# Format check
# e.g., pnpm biome check .
# e.g., prettier --check .
# e.g., black --check .
```

---

## AI Collaboration Notes

- This file is the primary reference for reviews and code implementation
- If a PR contains patterns not covered by this file, follow the "minimal change, safest option" principle
- Implement strictly according to design.md — do not expand feature scope
- When uncertain, ask rather than assume
- For the full collaboration workflow, see `README.md`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mattshma) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
