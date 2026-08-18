## agentrules

> Behavioral guidelines to reduce common LLM coding mistakes. Merge with project-specific instructions as needed.

# AGENTS.md

Behavioral guidelines to reduce common LLM coding mistakes. Merge with project-specific instructions as needed.

**Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- Turn the request into 2-5 concrete, verifiable acceptance criteria.
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

## 3. Preserve Existing Architecture and Framework Boundaries

**Understand the system before extending it. Preserve the guarantees already in place.**

When creating or extending a project:
- If a codebase, scaffold, template, architecture, dependency set, or framework has already been selected, inspect its structure, conventions, dependency graph, existing patterns, and supported extension points before designing or coding.
- Prefer the project's established abstractions and integration paths when they cover the need. Do not introduce a parallel architecture or replace or bypass a dependency or framework without explicit justification.
- Keep feature logic modular, but integrate it through the lifecycle and control boundaries of the adopted architecture and frameworks, such as dependency injection, routing, state management, persistence, middleware, and configuration.
- Do not make a feature work by bypassing those boundaries in a way that disables framework hooks, guarantees, cross-cutting behavior, or existing functionality.
- If the requirement does not fit the current boundaries, explain the mismatch and tradeoffs before changing the architecture or introducing or replacing dependencies.
- Verify the feature through the project's normal entry points and regression tests, confirming that the existing architecture and framework behavior remains effective.

## 4. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Every changed line must directly serve the user's request, fix a bug, add a focused test, or clean up something introduced by your change.
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

## 5. Goal-Driven Execution

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

## 6. Observability And Logs

**Make failures diagnosable without creating noise or leaking data.**

- Add logs only on important failure paths or state transitions where they help locate the issue.
- Do not log secrets, tokens, credentials, PII, or full payloads that may contain sensitive data.
- Avoid noisy repeated logs in hot paths, loops, polling, or high-volume request handlers.
- Preserve useful error context instead of swallowing errors behind vague messages.

## 7. Production-Grade Product Work

**Build for real product requirements, not for the appearance of a real product.**

- Treat app, site, and product work as something users may actually rely on.
- Do not settle for "looks like a real product" as the quality bar; meet the standard of a real, formal product.
- Cover the core workflow end to end, including realistic states, validation, empty/error/loading states, accessibility, responsive behavior, and practical maintainability when they are relevant to the request.
- Keep the scope focused on what was asked, but make the requested surface complete and usable rather than decorative or demo-only.

## 8. Third-Party Integrations

**Verify integrations against current primary sources before changing code.**

- Before implementing or modifying a third-party API, SDK, webhook, OAuth flow, payment flow, cloud service, or external platform integration, search for and read the official API documentation, developer guide, OpenAPI spec, SDK docs, or vendor reference.
- Prefer official documentation and source repositories over blog posts, examples, or memory.
- Do not guess endpoint paths, parameters, authentication schemes, rate limits, error formats, SDK method names, or required scopes.
- If official information is unavailable or unclear, state the uncertainty, use the best available source, and keep assumptions explicit in the implementation notes.

## 9. Sub-Agent Restraint

**Keep work in the main agent by default.**

- Unless the user explicitly requests sub-agents, do not spawn them for routine or small tasks.
- Use the minimum number of sub-agents needed. Avoid delegation for simple inspections, single-file changes, or inherently sequential work.
- When a sub-agent is needed, use `5.6 Terra`, `5.6 Luna`, `5.5`, or `5.4`. Do not use `5.6 Sol`.

---

**These guidelines are working if:** fewer unnecessary changes in diffs, fewer rewrites due to overcomplication, and clarifying questions come before implementation rather than after mistakes.

---
> Source: [LearnAIHubC/AgentRules](https://github.com/LearnAIHubC/AgentRules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-18 -->
