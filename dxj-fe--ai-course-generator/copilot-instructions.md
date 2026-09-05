## ai-course-generator

> Behavioral guidelines to reduce common LLM coding mistakes. Merge with project-specific instructions as needed.

# AGENTS.md

Behavioral guidelines to reduce common LLM coding mistakes. Merge with project-specific instructions as needed.

**Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

## 0. Project Background

The current project is a node full-stack project. Please spare no effort to build it according to the best practices, including code conventions, directory structure, architecture design, performance optimization, and so on

### Product UI source of truth

- The current product UI is the Keya experience in `src/features/keya` and the `/`, `/chat`, and `/course` routes.
- Future handbook tasks must add behavior to this UI instead of replacing product routes with `AiPlayground` or introducing a second visual system.
- Treat older training components as behavior and API references. Extract or reuse their client logic, types, and focused result renderers, then present them with the current Keya components and design tokens.
- Put prompt entry and task creation in `/chat`, public Agent progress in the chat thread, generated course/page/preview details in the right-side learning workspace, and persisted history in `/course`. Keep `/templates` as the template catalog.
- Agent progress may show structured public summaries and errors, but never private chain-of-thought. Keep the UI event-driven so the current JSON transport can later be replaced by the handbook's SSE task stream without redesigning components.
- Before implementing a new daily task, read `docs/ui-integration.md` and update its capability map when the task adds a new product surface.

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

## 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## 3. Surgical Changes

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

## 4. Goal-Driven Execution

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

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

---

**These guidelines are working if:** fewer unnecessary changes in diffs, fewer rewrites due to overcomplication, and clarifying questions come before implementation rather than after mistakes.

---
> Source: [dxj-fe/ai-course-generator](https://github.com/dxj-fe/ai-course-generator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-04 -->
