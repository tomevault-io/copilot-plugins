## cubestack

> Engineering guidance for AI coding agents working in this repository.

# AGENTS.md

Engineering guidance for AI coding agents working in this repository.
This is a multi-project monorepo for the CubeStack.

## Repository Layout

| Path | Tech Stack | Description | Agent Guidance |
|------|-----------|-------------|----------------|
| `operator/` | Go / Kubebuilder | K8s operator for platform resources | See `operator/AGENTS.md` |
| `web/` | TypeScript / React (example) | Platform Web Portal | See `web/AGENTS.md` |

> **Rule:** Each sub-project MUST have its own `AGENTS.md` with tech-stack-specific rules.
> The root `AGENTS.md` only contains cross-cutting concerns.

## Cross-Cutting Principles

### Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

### Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

### Surgical Changes

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

### Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:
```text
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

## Cross-Cutting Conventions

- Code comments must be English
- All sub-projects must include lint + test in CI
- Never commit secrets, credentials, or tokens

---
> Source: [suanova/cubestack](https://github.com/suanova/cubestack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-01 -->
