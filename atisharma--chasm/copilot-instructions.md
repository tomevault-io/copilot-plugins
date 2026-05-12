## chasm

> This file is for AI agents and LLM-assisted workflows contributing to chasm.

# Agent Contribution Guidelines

This file is for AI agents and LLM-assisted workflows contributing to chasm.

## Before Starting

- Read `README.md` and `docs/ARCHITECTURE.md` and understand the project purpose

## Before Submitting

- **All changes require human review.** Do not merge your own PRs.
- **Run tests** (if any exist).
- **Style check.** Use a linter for TypeScript or Python if available; otherwise, check for consistent and concise style.
- **No placeholders.** No TODO/FIXME in submitted code.

## What to Include

- **PR description:** Note AI assistance ("Generated with <AGENT>" or similar).
- **Commit attribution:** `Co-authored-by: Agent <agent@agent-framework.local>` if you did the bulk of the work.

## Do Not

- Merge, approve, or tag releases.
- Modify CI/CD, licensing, or dependencies without explicit human direction.
- Submit bulk changes (>500 lines) without prior discussion.

## Quality Checklist

- [ ] Follows existing code patterns and idioms
- [ ] Style clean and tidy (no errors, concise, clear) and production quality
- [ ] Complex logic has comments explaining *why*
- [ ] APIs and dependencies have been verified as real and existing
- [ ] New behaviour has been manually verified by testing
- [ ] The whole code has been tested for regressions in existing functionality

---
> Source: [atisharma/chasm](https://github.com/atisharma/chasm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-10 -->
