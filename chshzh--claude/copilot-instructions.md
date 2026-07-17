## claude

> Behavioral guidelines to reduce common LLM coding mistakes.

Behavioral guidelines to reduce common LLM coding mistakes.
**Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

---

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask with AskQuestion tool.
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

## Wiki

This project has a persistent engineering knowledge base at `.claude/wiki/` (28 pages).

**At the start of every non-trivial session**, orient before acting:
1. Read `.claude/wiki/index.md` to see what experience pages exist
2. Read the relevant page(s) for the task:
   - Building / Kconfig / west? → `concepts/ncs-build-system`
   - WiFi failures / provisioner / P2P? → `concepts/wifi-debugging-patterns`
   - NCS version upgrade? → `concepts/ncs-version-migration`
   - Memfault upload / OTA deploy? → `concepts/memfault-workflow`
   - Which board to target? → `entities/nrf7002dk` or `entities/nrf54lm20dk-plus-nrf7002eb2`
   - Homelab / Hermes / Docker? → relevant concept page in `concepts/`

**After resolving a non-trivial problem**, update the wiki:
1. Edit or create the relevant page with what was learned
2. Append a one-line entry to `.claude/wiki/log.md`

Wiki path: `.claude/wiki/` (relative to workspace root)
Link format: `[page-name](relative-path.md)` — standard markdown, NOT `[[wikilinks]]`

---

## Skills

Agent skills are reusable procedure files (`SKILL.md`) the agent reads before executing a specialized workflow (commit, release, debug, OTA, etc.).

**Full skill index:** `.claude/skills/index.md` — categorized list of all skills with one-line triggers.

**When to use a skill:** When the user's request matches a skill trigger, load its `SKILL.md` with `read_file` before acting. Key skills for this workspace:

| Trigger | Skill |
|---------|-------|
| Build / flash / west command | `chsh-sk-ncs-env` |
| Debug WiFi / firmware crash | `chsh-sk-ncs-3.2-debug` |
| NCS version upgrade | `chsh-sk-ncs-migrate` |
| Memfault OTA release | `chsh-sk-memfault` |
| Git commit / push | `chsh-sk-ncs-3.4-git` |
| New NCS feature → code | `chsh-sk-ncs-0-workflow` |
| Hardware validation tests | `chsh-sk-ncs-4.2-validation` |

Skills path: `.claude/skills/` (relative to workspace root)

---

---
> Source: [chshzh/claude](https://github.com/chshzh/claude) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-17 -->
