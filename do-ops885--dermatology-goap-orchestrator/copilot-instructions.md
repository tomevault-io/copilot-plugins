## dermatology-goap-orchestrator

> **For AI Coding Agents** — Entry point for working with this codebase.

# AGENTS.md

**For AI Coding Agents** — Entry point for working with this codebase.

**Last Updated:** 2026-02-10

---

## 1. Quick Start (TL;DR)

| Command               | Purpose                                       |
| :-------------------- | :-------------------------------------------- |
| `npm run dev`         | Start Vite dev server (http://localhost:5173) |
| `npm run build`       | Production bundle                             |
| `npm run lint`        | ESLint v9 + security rules                    |
| `npm run typecheck`   | TypeScript strict mode                        |
| `npm run test`        | Vitest suite (jsdom)                          |
| `npx playwright test` | E2E tests (Playwright)                        |

Test locations:

- Unit tests: `tests/unit/*.test.ts`
- E2E tests: `tests/e2e/*.spec.ts`
- Setup: `tests/setup.ts`

---

## 2. Git Workflow & CI/CD Rules

### 2.1 Atomic Git Commits

- Each commit must represent a single logical change
- Commit messages must follow conventional commit format (`feat:`, `fix:`, `docs:`, `test:`, etc.)
- Do not bundle unrelated changes in a single commit
- Use `git add -p` for granular staging when needed

### 2.2 Cleanup Requirements

Before committing, remove temporary files that are not meant for git:

- Delete test output files (coverage reports, test artifacts)
- Remove analysis temp files (_.tmp, _.log)
- Clean report files not intended for the repository
- Run `git status` to verify only intended files are staged

### 2.3 Git Worktree for Existing PRs

When working on existing pull requests:

- Use `git worktree` to checkout PR branches without disrupting current work
- Example: `git worktree add ../pr-123 origin/pr-branch`
- Clean up worktrees when done: `git worktree remove ../pr-123`

### 2.4 GitHub Actions Source of Truth

- **All GitHub Actions must pass** — this is the ultimate validation
- Use `gh run list` to view recent workflow runs
- Use `gh run watch` to monitor running workflows
- Use `gh run view <run-id>` to inspect failed runs
- Never merge or complete work with failing CI checks

### 2.5 Pre-existing Issue Policy

- **Always fix pre-existing issues** discovered during work
- If lint, typecheck, or tests fail due to existing code, fix them
- Do not skip or ignore failures
- Create separate commits for fixes unrelated to the main task

---

## 3. Document Index (Agents Doc Set)

Details are split into `agents_doc/` for maintainability. Use these files as the source of truth for agent behavior and engineering rules:

- `agents_doc/01_principles.md`
- `agents_doc/02_runtime_agents.md`
- `agents_doc/03_dev_agents.md`
- `agents_doc/04_safety_memory_error_style.md`
- `agents_doc/05_verification_checklist.md`
- `agents_doc/06_documentation_policy.md`

---

## 4. Source Of Truth

Operational mappings live in code, not prose. When in doubt, consult:

- GOAP actions: `services/goap.ts`
- Agent registry: `services/goap/registry.ts`
- Agent executors: `services/executors/*.ts`
- GitHub Actions: `.github/workflows/*.yml`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/do-ops885)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/do-ops885)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
