## sanimex

> Before starting any work, read and follow these skills:

# SANIMEX Project Guidelines

## Required Skills

Before starting any work, read and follow these skills:

1. **Git Workflow** (`skills/git-workflow/SKILL.md`) - MANDATORY for all code changes
   - Create GitHub Issue first
   - Create feature/fix branch
   - Create PR instead of pushing to main
   - Wait for CI before merging

2. **Web Artifacts Builder** (`skills/web-artifacts-builder/SKILL.md`) - For building React components
   - Component patterns, animation classes, i18n
   - Avoid "AI slop" (no purple gradients, excessive centering)

3. **Brand Guidelines** (`skills/brand-guidelines/SKILL.md`) - For UI/design work

4. **Frontend Design** (`skills/frontend-design/SKILL.md`) - For component development

5. **Web Testing** (`skills/webapp-testing/SKILL.md`) - For test-related tasks

6. **Canvas Design** (`skills/canvas-design/SKILL.md`) - For design mockups

7. **Theme Factory** (`skills/theme-factory/SKILL.md`) - For theming/styling

## Git Workflow (MANDATORY)

**For ALL git/GitHub commands, follow `skills/git-workflow/SKILL.md` exactly.**

Quick reference:
1. Create issue first
2. Create feature/fix branch
3. Make commits (Conventional Commits)
4. **Run pre-push verification (lint, tsc, test, build)**
5. Push branch and create PR
6. Wait for CI, then merge

## Tech Stack

- React 18 + TypeScript + Vite
- TailwindCSS with custom CSS variables
- i18next for translations (EN, FR, AR with RTL)
- Playwright for E2E tests
- react-helmet-async for SEO

## Key Directories

- `src/pages/` - Page components
- `src/components/` - Shared components
- `src/locales/` - Translation files (en.json, fr.json, ar.json)
- `src/data/` - SEO metadata, schemas
- `tests/` - Playwright E2E tests
- `skills/` - Project skills documentation

## Sub-Agent and Retry Limits (CRITICAL)

To prevent infinite loops and excessive token consumption:

1. **Maximum 3 retries for any failing command**
   - If a command (npm install, git push, etc.) fails 3 times, STOP and report the failure
   - Do NOT keep retrying the same command hoping for a different result

2. **Detect repeated failures**
   - If you find yourself running the same command more than 3 times, something is wrong
   - Stop immediately and either:
     - Try an alternative approach
     - Report the blocker to the user
     - Skip that task and continue with others

3. **Network/dependency failures**
   - `npm install` timing out = network issue, not fixable by retrying
   - After 2 failed attempts, skip and document what needs manual intervention

4. **Time-boxed operations**
   - No single task should take more than 10 minutes of repeated attempts
   - If stuck, commit partial progress and report status

5. **Sub-agent tasks**
   - Sub-agents MUST follow these same rules
   - If a sub-agent task involves installation or network ops, set explicit retry limits
   - Prefer failing fast over infinite retries

---
> Source: [sanimexsa/SANIMEX](https://github.com/sanimexsa/SANIMEX) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
