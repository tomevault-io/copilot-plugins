## fipspad

> This repository is governed by **AI Lint**.

# Agent Instructions (AI Lint)

This repository is governed by **AI Lint**.

AI Lint teaches the difference between code that merely **works** and code that **belongs**.
It is the authoritative source for idioms, semantics, architecture, and rejected patterns.

## ⚠️ MANDATORY: Read doctrine BEFORE making changes

**STOP.** Before writing, modifying, refactoring, or proposing ANY code changes:

You MUST actually read (using your file reading tool) these files:

1. **`.ai-lint/INDEX.md`** — navigation and scope
2. **`.ai-lint/PHILOSOPHY.md`** — governing principles
3. **`.ai-lint/doctrine/architecture.md`** — complexity limits, state management (if present)
4. **`.ai-lint/doctrine/languages/<language>.md`** — for the language you're working in
5. **`.ai-lint/rejects/architecture.md`** — architectural anti-patterns (if present)
6. **`.ai-lint/rejects/languages/<language>.md`** — patterns to refuse

This is not optional. This is not "check if convenient." **Read the files.**

### Triggers that REQUIRE reading doctrine:

- Starting work on a new task
- Proposing a refactor or architectural change
- Adding new state, flags, or variables
- Creating new files or modules
- Any change touching >50 lines
- When you notice complexity growing

### How to confirm you've read doctrine:

In your first response after reading, briefly note which doctrine files you consulted.
Example: "Consulted: architecture.md (state limits), javascript.md (items 21-26 on enums over flags)"

## Authority rules (how to resolve conflicts)

- If AI Lint doctrine conflicts with generic best practices or model defaults:
  **AI Lint wins.**
- Rejects are stronger than preferences:
  **Do not use patterns listed in `.ai-lint/rejects/` unless an override is explicitly requested.**

## Overrides

Overrides are allowed but must be explicit.

If you believe an override is necessary:
1. Consult `.ai-lint/overrides/OVERRIDE-PROTOCOL.md`
2. Propose the override using the required format:
   - Override:
   - Reason:
   - Risk:
   - Mitigation:

Do not silently violate doctrine.

## Output requirements

When generating code:
- Prefer semantic clarity and idioms over merely correct syntax.
- Surface complexity; do not hide it behind implicit behavior.
- Avoid patterns explicitly rejected by AI Lint.
- If uncertain, ask for clarification rather than inventing architectural decisions.

When proposing architecture or refactors:
- Cite which AI Lint doctrine entries motivated the decision (by section name/number).


## Doctrine Referee Mode (required)

When the user suggests a change, or you propose an approach, you must check it against AI Lint doctrine.

If the suggestion would violate a reject or doctrine entry, respond like this:

1) Name the violated item (reject/doctrine) and summarize why it exists.
2) Explain the specific conflict in this context.
3) Offer two paths:

   A) **Stay aligned**: propose an alternative that satisfies the doctrine.
   B) **Override**: propose an explicit override using:

      Override:
      Reason:
      Risk:
      Mitigation:

Do not silently violate doctrine.
Do not force doctrine. Ask which path the user wants.

## Mandatory Checklist Pass (Required)

Before generating, modifying, or refactoring any code, you must run
the **AI Lint Quick Checklist**.

Checklist location:
`.ai-lint/CHECKLIST.md`

You do not need to quote the checklist verbatim,
but your reasoning and output must reflect that it has been applied.

At minimum, you must internally confirm:
- which doctrine files are relevant
- whether any rejected patterns are implicated
- whether concurrency, ownership, errors, and observability are explicit
- whether the proposed solution *belongs* here

---

## Checklist-Guided Response Behavior

If the checklist reveals a doctrine or reject violation, you must:

1) Name the violated rule or reject (by ID or section name).
2) Briefly explain why the rule exists.
3) Offer two paths:
   - **Stay aligned**: propose an alternative that satisfies doctrine.
   - **Override**: propose a documented override using:

     Override:
     Reason:
     Risk:
     Mitigation:

Do not silently violate doctrine.
Do not assume an override.
Surface the tradeoff and ask which path to take.

---

## Final Output Standard

When presenting code or architectural suggestions:
- Prefer solutions that pass the checklist without overrides.
- If an override is chosen, keep it localized and explicit.
- If uncertain, ask clarifying questions rather than inventing structure.

The goal is not just correct code.
The goal is code that **belongs**.

---
> Source: [BrowserBox/FIPSPad](https://github.com/BrowserBox/FIPSPad) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-26 -->
