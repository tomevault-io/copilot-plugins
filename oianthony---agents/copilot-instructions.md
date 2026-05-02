## agents

> Behavioral guidelines to reduce common LLM coding mistakes. Merge with project-specific instructions as needed.

# AGENTS.md

Behavioral guidelines to reduce common LLM coding mistakes. Merge with project-specific instructions as needed.

**Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

## Core Principles

- Employ first principles reasoning to distill core needs, eliminate pseudo-requirements, identify genuine value, simplify complexity, and prevent over-engineering.
- Based on the YAGNI principle (You Aren't Gonna Need It) and the KISS principle (Keep It Simple, Stupid).

## Learn from Corrections

**When the user corrects you, record it in the project's AGENTS.md or CLAUDE.md.**

- If the user says "no", "don't", "stop doing X", or otherwise corrects your approach, and the mistake is likely to recur in the project, add a rule to the project's `AGENTS.md` or `CLAUDE.md`.
- Only record when the lesson is project-relevant and reusable — not one-off misunderstandings.
- Keep rules concise: what to avoid and what to do instead.

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
- Avoid inventing extra entities/components/abstractions without necessity.
- Use modern best practices by default.
- Add backward compatibility / legacy workarounds only when requested.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:

- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.
- Don't ignore lint errors, unless you are explicitly asked to do so.

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

## Communication

- Use Chinese for conversations, include the code review result and plan file content of plan mode.
- Use English for all code-related content (code comments, UI strings, commit messages, PR descriptions) to ensure better internationalization and collaboration in open-source projects.
- OpenSpec Propose artifacts (design docs, specs, task lists) must be written in Simplified Chinese.

## Documentation Standards

- Include: assumptions, setup, usage, verification steps when relevant.
- When writing or editing a document file: Include Mermaid diagrams if they help clarify complex workflows or system architecture.
- Avoid using double quotes and parentheses inside square brackets in Mermaid diagrams, as this can cause parsing errors.
- **CRITICAL: Never provide level of effort time estimates (e.g., hours, days, weeks) for tasks. Focus solely on breaking down the work into clear, actionable steps without estimating how long they will take.**

## Development Environment

### Package Management

- When no package manager is specified in the front-end repo, `bun` is preferred.

## Browser Automation

- Use `agent-browser` for browser automation tasks.
- Run `agent-browser --help` for command reference.
- Core workflow:
  1. `agent-browser open <url>`
  2. `agent-browser snapshot -i`
  3. `agent-browser click @e1` or `agent-browser fill @e2 "text"`
  4. Re-run snapshot after page state changes.

## GitHub CLI (gh)

- Use `gh` for GitHub-related operations (issues, PRs, repos, workflows, API requests).

## RTK - Rust Token Killer

@~/.agents/RTK.md

---

**These guidelines are working if:** fewer unnecessary changes in diffs, fewer rewrites due to overcomplication, and clarifying questions come before implementation rather than after mistakes.

---
> Source: [OiAnthony/.agents](https://github.com/OiAnthony/.agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->
