## deci-fe

> This project includes **GEMINI.md** so the Karpathy-inspired behavioral guidelines apply automatically when you work here.

# Using this repo with Gemini CLI

@CONVENTION.md

@DESIGN.md
@DESIGN-DARK.md

This project includes **GEMINI.md** so the Karpathy-inspired behavioral guidelines apply automatically when you work here.

## In this repository

1. Open the folder in Gemini CLI.
2. The file [`GEMINI.md`](GEMINI.md) is committed in the root, so Gemini CLI reads it automatically upon startup.
3. The behavioral guidelines below govern all implementation, refactoring, and research tasks.

## Karpathy Behavioral Guidelines

### 1. Think Before Coding
**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

### 2. Simplicity First
**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

### 3. Surgical Changes
**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

### 4. Goal-Driven Execution
**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

## Use the same guidelines in another project

**Gemini CLI (recommended):** Copy [`GEMINI.md`](GEMINI.md) into that project’s root directory. Adjust or merge with existing instructions as you like.

**Other tools:** If using Cursor, use [`.cursor/rules/karpathy-guidelines.mdc`](.cursor/rules/karpathy-guidelines.mdc). If using Claude Code, use [`CLAUDE.md`](CLAUDE.md).

## Optional: personal Agent Skills

If you want the same content as a reusable skill for Gemini CLI, you can use the `skill-creator` to wrap these guidelines into a persistent skill.

## Claude vs Cursor vs Gemini CLI

- **Claude Code:** Uses [`CLAUDE.md`](CLAUDE.md).
- **Cursor:** Uses [`.cursor/rules/karpathy-guidelines.mdc`](.cursor/rules/karpathy-guidelines.mdc).
- **Gemini CLI:** Uses [`GEMINI.md`](GEMINI.md).

## For contributors

When you change the four principles, keep **[`GEMINI.md`](GEMINI.md)**, **[`CLAUDE.md`](CLAUDE.md)**, and **[`.cursor/rules/karpathy-guidelines.mdc`](.cursor/rules/karpathy-guidelines.mdc)** in sync. If the published skill/plugin text should match, update **[`skills/karpathy-guidelines/SKILL.md`](skills/karpathy-guidelines/SKILL.md)** as well.

---

**These guidelines are working if:** fewer unnecessary changes in diffs, fewer rewrites due to overcomplication, and clarifying questions come before implementation rather than after mistakes.

---
> Source: [j0hs04/Deci_FE](https://github.com/j0hs04/Deci_FE) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-07 -->
