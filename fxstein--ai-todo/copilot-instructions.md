## commit-standards

> Commit message standards for ai-todo

# Commit Standards

## 1. Commit Prefixes

Use these prefixes to help the release script determine version bumps:

**Backend/Infrastructure (PATCH releases):**
-   `backend:` - Infrastructure changes
-   `infra:` - CI/CD, logging, tooling
-   `release:` - Release process changes
-   `internal:` - Refactoring, tests

**Frontend/User-Facing (MINOR/MAJOR releases):**
-   `feat:` - New features (e.g., `feat: Add tamper detection`)
-   `fix:` - Bug fixes (e.g., `fix: Resolve checksum mismatch`)
-   `docs:` - Documentation updates

## 2. Task References

**CRITICAL REQUIREMENT:**

-   **MUST use `task#nn` format** (e.g., `task#210`, `task#212`) when referencing ai-todo tasks.
-   **NEVER use `#nn` format** (e.g., `#210`) - this links to GitHub Issues/PRs, not tasks.

**Examples:**
-   ✅ `feat: Implement tamper detection (task#210)`
-   ✅ `docs: Update README for v3.0 (task#203)`
-   ❌ `fix: Bug in task #210` (Wrong format)

---
> Source: [fxstein/ai-todo](https://github.com/fxstein/ai-todo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-07 -->
