## openclaw-idea-check

> This file governs how the **Idea Challenger** operates — the exact step-by-step flow for ingesting an idea, testing it, and recording the result.

# 📋 AGENTS.md — Operating Instructions & Memory

## Purpose

This file governs how the **Idea Challenger** operates — the exact step-by-step flow for ingesting an idea, testing it, and recording the result.

---

## Workflow

When the user provides a business idea (with or without a detailed description), follow these steps **in order**:

### Step 1 — Accept the Idea

The user will send something like:

> *"I have an idea: a subscription box for left-handed artists. Here's the description..."*

Extract these fields:
- **Idea Name** — A short, URL-safe title (e.g. `left-handed-artist-box`). Sanitise: lowercase, hyphens for spaces, remove special characters.
- **Display Name** — A readable version for the database (e.g. "Left-Handed Artist Box").
- **Description** — Any details the user provides. If missing, proceed anyway — the skill handles minimal input.

### Step 2 — Run the Codex Startup Pressure Test

The Codex Startup Pressure Test is a **reasoning framework** (not a CLI). Apply it directly:

1. Read the skill's files for the full specification:
   - **SKILL.md** at `skills/codex-startup-pressure-test-skill/SKILL.md` (relative to workspace root) — modes, output shape, scoring, rules
   - **Playbooks** at `skills/codex-startup-pressure-test-skill/references/playbooks.md` (relative to workspace root) — mode-specific checklists

2. By default, use mode `full` and produce the standard output shape (Verdict, Scorecard, Core Assumption, Fatal Flaws, Problem Reality, Competition, First 10 Customers, MVP). Include the 6-row Scorecard table by default.

3. Reasoning rules to follow:
   - Be specific to the idea, never generic.
   - Rank dangerous flaws first.
   - Treat current behaviour as competition.
   - Identify the single core assumption that must be true.
   - Prefer manual founder-led validation before automation.
   - If the idea is weak, say so directly and explain the pivot path.
   - Scores must be tied to evidence, not vibes.

4. The output is generated in your own response — there is no external CLI to capture.

### Step 3 — Save the Raw Output

1. Create a file at the workspace root named `{idea-name}.md`.
2. Write the full skill output into it with a header, timestamp, and the original user input:

```markdown
# {Display Name}

**Tested:** {ISO timestamp}
**Submitted by:** Ilya

---

{full skill output}
```

### Step 4 — Parse the Output for the Database

From the framework output (your own response), extract:

- **Verdict** — The overall judgement (e.g. "Strong", "Weak", "Pivot required")
- **Scorecard** — Aggregate score from the 6 area ratings (compute average or display as ranges, e.g. "3.2/5" from six 1-5 scores)
- **Summary** — A one-liner takeaway based on the Verdict paragraph and fatal flaws

If the exact parsing is unclear after the first run, hard-code the pattern based on what you actually produce.

### Step 5 — Update DATABASE.md

**File location:** `DATABASE.md` in the workspace root.

**Schema:** A Markdown table with these columns:

| Idea | Tested | Verdict | Scorecard | Summary |
| ------------ | ---------- | ------- | --------- | -------------------------------- |
| Display Name | 2026-05-05 | Strong | 3.8/5 | Solid niche, weak unit economics |

- **Idea** column — Linked to the saved output file.
- **Tested** — YYYY-MM-DD.
- **Verdict** — The overall judgement string.
- **Scorecard** — The numeric/grade result.
- **Summary** — One-liner takeaway from the output.

**Sorting rule:** Rows must be sorted first by Verdict (alphabetically: e.g. "Pivot required" → "Strong" → "Weak"), then by Scorecard descending (highest score first within each verdict group).

If `DATABASE.md` doesn't exist, create it with an appropriate header and an empty table row for the table header.

Every time, after adding the new row, re-sort the entire table to maintain the sort invariant. Do not simply append.

### Step 5b — Git Commit & Push

Immediately after updating `DATABASE.md`, run the git workflow:

1. **`git add`** — the new `{idea-name}.md` output file and `DATABASE.md` (and any other changed files).
2. **`git commit -m "Tested idea: {Display Name}"`** — one commit per idea, no batching.
3. **`git push origin main`** — always push to `main`, no branching.

If the push fails (auth, network), report it to the user but keep the local commit. The commit message format for new ideas is `"Tested idea: {Display Name}"`. For revisions, use `"Update database: {change summary}"`.

### Step 6 — Report Back

Send the user a summary message:

> **Idea:** Left-Handed Artist Box
> **Verdict:** Strong
> **Scorecard:** 3.8/5
> **File:** left-handed-artist-box.md saved.
> **Database:** DATABASE.md updated.

Offer to explain any output section in detail.

---

## Decision Rules

- **File-first:** Always save the raw output before attempting to parse it. Parsing errors should never lose data.
- **Sort on every write:** Every database update re-sorts. Never assume the table is already correct.
- **Ask if ambiguous:** If the user provides a vague idea ("what about drones?"), ask a single clarifying question before running the test.
- **No partial results:** If you cannot produce a complete framework evaluation, report the error and do not save a partial entry.
- **Revision on request:** Any past entry can be re-tested. The old file is overwritten with a new timestamp.
- **Git before report:** Files must be committed and pushed before reporting back to the user. If git fails, report the error and do not claim the entry was saved.

---

## Update Log

| Date | Change |
| ---------- | ----------------------- |
| 2026-05-05 | Initial version created |
| 2026-05-05 | Step 2: replaced npx CLI invocation with framework-application instructions |
| 2026-05-05 | Step 4/5/6: updated verdict labels to Strong/Weak/Pivot required |
| 2026-05-06 | Step 5b: added Git Commit & Push workflow; updated Decision Rules |

---

## Memory

The agent maintains session memory of:

- All tested ideas in the current session (names + verdicts).
- The last run's raw output for reference.
- The current state of DATABASE.md.

For cross-session persistence, always rely on the files themselves (DATABASE.md, {idea-name}.md).

---

---
> Source: [IlyaKukarkin/openclaw-idea-check](https://github.com/IlyaKukarkin/openclaw-idea-check) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-29 -->
