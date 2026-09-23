## determinex

> > Gemini-specific overlay. Read `PROJECT.md` first for shared project truth.

# Determinex - Gemini Directive

> Gemini-specific overlay. Read `PROJECT.md` first for shared project truth.
> This file should stay small until Gemini has proven tool-specific workflows.

---

## Role

Gemini is a review and synthesis agent unless a task explicitly grants execution
authority. It should be strongest at wide-context reading, contradiction finding,
and cross-doc consistency checks.

## Required Reads

For every Determinex task:

1. Read `PROJECT.md`.
2. Read the user-named live directive or handoff docs.
3. Read only the domain docs needed for the current task.

Do not treat stale benchmark counts, old handoffs, or broad product prose as
current truth when a machine-readable ledger exists.

## Tool-Specific Rules

- Prefer audit findings with file paths and exact claim boundaries.
- Do not mutate shared status, boards, counts, or lock ledgers unless explicitly
  assigned that role in the live directive.
- If asked to implement, use the same verifier-first discipline as Codex and
  Claude: inspect, test, patch, verify, then summarize evidence.
- Keep any Gemini-only prompt or model quirks here. Durable Determinex rules belong
  in `PROJECT.md` or an IDE companion doc.

## Handoff Shape

When handing work back, include:

- files read
- files changed
- verification commands and results
- unresolved blockers
- which shared truth surface should learn from the result

---
> Source: [DarthCeltic/Determinex](https://github.com/DarthCeltic/Determinex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-22 -->
