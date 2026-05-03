## grundsicherungsrechner

> - **Primary languages:** TypeScript, Python

# AGENTS.md — Claude Code Project Guidelines

## Language & Stack Preferences

- **Primary languages:** TypeScript, Python
- **TypeScript:** Use strict mode (`"strict": true` in tsconfig). Prefer `type` over `interface` unless extending. Use Zod for runtime validation.
- **Python:** Target 3.11+. Use type hints everywhere. Prefer Pydantic for data models. Use `ruff` for linting/formatting.
- **Package managers:** `pnpm` for Node projects, `uv` for Python projects.
- **Testing:** Every module must have tests. Use `vitest` (TS) or `pytest` (Python). Aim for tests that document behavior, not just coverage.

---

## Documentation Requirements

### Living Documentation

Maintain the following Markdown files at the project root. **Update them as you work, not after.**

| File | Purpose |
|---|---|
| `ARCHITECTURE.md` | High-level system design, component relationships, data flow diagrams |
| `PROGRESS.md` | Running log of what's been done, what's in progress, and what's blocked |
| `TODO.md` | Prioritized task list with `[ ]` / `[x]` checkboxes |
| `DECISIONS.md` | Architecture Decision Records (ADRs) — why non-obvious choices were made |
| `CHANGELOG.md` | User-facing summary of changes per logical milestone |

### Documentation Rules

- When starting a new task, **add it to `TODO.md` first**.
- When completing a task, **update `PROGRESS.md`** with a short summary of what changed and why.
- When making a non-trivial architectural choice, **add an ADR to `DECISIONS.md`** with: context, options considered, decision, and consequences.
- Keep `ARCHITECTURE.md` in sync whenever modules, services, or data flows change.
- Use Mermaid diagrams in `ARCHITECTURE.md` where they add clarity.

---

## Git Workflow (Local)

### Branch Naming

```
<type>/<short-description>

Examples:
feat/user-auth
fix/db-connection-timeout
refactor/api-client-types
docs/architecture-update
```

### Commit Messages

Follow [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>(<scope>): <short summary>

<optional body — explain *why*, not *what*>
```

Types: `feat`, `fix`, `refactor`, `docs`, `test`, `chore`, `perf`, `ci`

Keep commits atomic — one logical change per commit. If you need "and" in the summary, split it into two commits.

### Pull Request Process

Even for local development, treat branches as PRs to maintain quality:

1. **Create a feature branch** from `main`:
   ```bash
   git checkout -b feat/my-feature main
   ```

2. **Work in small, focused commits.** Commit early and often.

3. **Before merging, self-review:**
   - Re-read every changed file. Look for debug leftovers, TODO comments, and dead code.
   - Run the full test suite: `pnpm test` / `pytest`
   - Run the linter: `pnpm lint` / `ruff check .`
   - Confirm type-checking passes: `pnpm typecheck` / `pyright`

4. **Write a PR description** as a commit message body or in `PROGRESS.md`:
   ```
   ## What
   Brief summary of the change.

   ## Why
   What problem does this solve or what goal does it advance?

   ## How
   Key implementation details, trade-offs, or things a reviewer should look at.

   ## Testing
   How was this tested? What edge cases were considered?
   ```

5. **Merge with a squash or rebase** to keep `main` history clean:
   ```bash
   git checkout main
   git merge --squash feat/my-feature
   git commit  # Use the PR description as the commit body
   git branch -d feat/my-feature
   ```

---

## Coding Standards

### General

- **No `any` types** in TypeScript. If truly unavoidable, use `unknown` with a type guard.
- **No bare `except`/`catch`** — always catch specific errors.
- **Functions should do one thing.** If a function is longer than ~40 lines, consider splitting it.
- **Name things for what they mean**, not how they're implemented. `getUserPermissions()` not `queryPermissionsTableAndFilter()`.
- **Fail early, fail loud.** Validate inputs at boundaries (API handlers, CLI entry points). Trust internal code more.
- **No secrets or credentials in code.** Use environment variables and `.env` files (which are `.gitignore`'d).

### Error Handling

- Use custom error classes/types. Don't throw raw strings.
- Include context in errors: what was being attempted, with what inputs.
- Log errors at the boundary where they're handled, not where they're thrown.

### File & Folder Structure

- Group by feature/domain, not by file type. Prefer `src/auth/` with its own models, routes, and tests over `src/models/`, `src/routes/`, etc.
- Keep a flat structure until complexity demands nesting.
- Colocate tests next to the code they test: `user-service.ts` → `user-service.test.ts`.

---

## Task Execution Approach

When working on a task:

1. **Understand first.** Read relevant existing code and docs before writing anything.
2. **Plan in `TODO.md`.** Break the task into small, verifiable steps.
3. **Implement incrementally.** Get something working end-to-end, then iterate.
4. **Test as you go.** Write the test alongside or immediately after the implementation.
5. **Document what you did.** Update `PROGRESS.md` and any other affected docs.
6. **Commit logically.** Each commit should leave the codebase in a working state.

If you are unsure about something, ask for clarification.

---

## Things to Avoid

- Don't over-engineer. Build for current requirements with reasonable extension points, not for imaginary future ones.
- Don't introduce new dependencies without justification. Prefer the standard library when it's close enough.
- Don't leave commented-out code. That's what git history is for.
- Don't ignore warnings. Fix them or explicitly suppress with a comment explaining why.
- Don't write documentation that restates the code. Document *why*, *constraints*, and *gotchas*.

---
> Source: [bnjmnsbl/grundsicherungsrechner](https://github.com/bnjmnsbl/grundsicherungsrechner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
