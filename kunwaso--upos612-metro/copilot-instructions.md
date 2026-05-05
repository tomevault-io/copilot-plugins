## clarification-gate

> Require agent to restate goal, list assumptions, and ask ≤3 blocking questions before writing code on any non-trivial implement task. Applied in Agent mode automatically.


# Clarification Gate — Understand Before You Code

**Applies to:** `implement`, multi-file, and any higher-risk task in Agent mode.
**Skip for:** `tiny` (single-file, tightly scoped), `explain`, `investigate`, and when the user has already said "just implement," "go ahead," or equivalent.

## Required output BEFORE the first file edit

Output all three sections together in a single response, then **stop and wait** for user confirmation:

### 1. Restate
One or two sentences:
- What does the user want (fix / feature / refactor)?
- In which area of the codebase (module, page, flow)?
- What does success look like?

If ambiguous after reading the message and doing at most 2 targeted searches, state what you inferred and flag it as an assumption.

### 2. Assumptions
Bullet list of every default choice you are about to rely on. Examples:
- "Web channel only; Telegram not in scope."
- "No new DB migration required."
- "Existing permission `aichat.manage` is used; no new role."
- "Only the listed controller methods; shared Util is read-only."

State these so the user can catch a wrong one before you write code.

### 3. Blocking questions (max 3)
Only questions where a wrong answer would change **which files you edit** or the **design direction**. Prefer:
- Multiple-choice: `A: …  B: …  C: …`
- Yes/No with default stated: `"Should X apply? (default: Yes)"`

**Do NOT ask about things grep / semantic search can resolve** (e.g. "which controller handles this route?"). Look those up yourself.
**Do NOT ask more than 3 questions.** Pick the blocking ones only.

---

## After confirmation

Once the user says "looks right," "go ahead," answers the questions, or gives any green-light signal — proceed immediately without repeating the gate.

---

## Anti-patterns (violations of this rule)

- Starting to write code without outputting the gate on a multi-file task.
- Asking more than 3 questions at once.
- Asking questions answerable from the codebase.
- Skipping the gate and silently burying assumptions in code comments.
- Running the gate on `tiny` single-file fixes or pure explanation turns.

---

## Reference

Full policy: `AGENTS.md §0.2b`
Fast summary: `AGENTS-FAST.md §3.1`

---
> Source: [kunwaso/upos612-metro](https://github.com/kunwaso/upos612-metro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->
