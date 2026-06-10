## god-mode-claude

> <!-- ✏️ CUSTOMIZE: Replace with your project description -->

# CLAUDE.md — God Mode

## Project Overview
<!-- ✏️ CUSTOMIZE: Replace with your project description -->
[Your project description here]

## Tech Stack
<!-- ✏️ CUSTOMIZE: Replace with your actual stack -->
- Language: TypeScript
- Framework: Next.js 15 (App Router)
- Database: PostgreSQL + Prisma
- Package Manager: pnpm
- Testing: Vitest + Playwright

## Commands
<!-- ✏️ CUSTOMIZE: Replace with your actual commands -->
- `pnpm dev` — Start dev server
- `pnpm build` — Production build
- `pnpm test` — Run unit tests
- `pnpm test:e2e` — Run E2E tests
- `pnpm lint` — Lint check
- `pnpm format` — Format code

---

## 🧠 Thinking Rules

1. **Read before write.** Before modifying any file, read it first. Match existing patterns, naming conventions, and code style. Do not introduce new paradigms.
2. **Verify before assuming.** Check actual source files for import paths, function signatures, and type definitions. Never rely on memory or assumptions.
3. **Ask when unsure.** If confidence in understanding the requirement is below 95%, ask clarifying questions before modifying any files. List what you understand and what needs clarification.
4. **Plan before executing.** For tasks touching 3+ files, outline the plan and get approval before making changes.

## 🛡️ Safety Rules

1. **Off-limits files — NEVER modify:**
   - `*.lock` / `*-lock.*` (package lock files)
   - `.env*` (environment/secrets)
   - `src/generated/*`, `prisma/migrations/*`
   - `node_modules/`, `vendor/`, `.next/`, `dist/`
2. **Destructive operations require confirmation:**
   - Deleting files or directories
   - Dropping/truncating database tables
   - Force-pushing or modifying CI/CD configs
   - Any operation with `--force` or `sudo`
3. **Never commit secrets.** No API keys, tokens, passwords, or credentials in code. Use environment variables.
4. **Handle errors explicitly.** No bare `catch {}` or `except:`. Always catch specific errors with meaningful handling.

## ⚡ Quality Rules

1. **Test everything.** Write tests for new functions. Run the full test suite before AND after changes. If tests fail, fix them before reporting completion.
2. **Keep it small.** Functions: <50 lines. Files: <300 lines. If larger, refactor.
3. **Explicit > clever.** Readable code beats concise code. Name variables descriptively. Avoid nested ternaries and clever one-liners.
4. **One thing per commit.** Each commit addresses one logical change. Use Conventional Commits: `feat|fix|docs|refactor|test|chore(scope): description`.
5. **No dead code.** Don't comment out code "for later." Delete it. Git has history.

## 📦 Output Rules

1. **Be concise.** Show only changed lines with surrounding context. Don't echo back entire files or code the user just showed you.
2. **Diff over dump.** When explaining changes, describe what changed and why — don't reproduce the full file.
3. **Batch related edits.** Group related file changes into a single logical operation. Don't read/write the same file multiple times.
4. **Verify before done.** After implementing, run tests and linting. Only report completion after verification passes.

## 📐 Architecture Notes
<!-- ✏️ CUSTOMIZE: Add your project-specific architecture decisions -->
<!--
Example:
- We use Server Components by default; add 'use client' only for interactivity
- State management: Zustand for client state, React Query for server state
- API validation: Zod schemas in src/lib/validators/
- Auth: NextAuth.js v5 with JWT strategy
-->

---

<!-- 
📏 LINE COUNT CHECK: This file should stay under 200 lines.
    If you need more rules, use .claude/rules/ directory for topic-specific files.
    Current: ~75 lines (excluding comments and blanks)
-->

---
> Source: [0rnot/god-mode-claude](https://github.com/0rnot/god-mode-claude) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-10 -->
