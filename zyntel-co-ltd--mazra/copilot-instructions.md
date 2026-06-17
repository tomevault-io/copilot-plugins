## project-status-enforcement

> Enforce PROJECT_STATUS/ folder update before any push or PR


# PROJECT_STATUS/ — Mandatory Update Gate

This rule applies to every session in every Zyntel repo. It is not optional.

The canonical standard lives in **`zyntel-playbook/05-infrastructure/project-status-file.md`** (knowledge repo) or `knowledge/zyntel-playbook/...` when working from another clone.

---

## On Session Start

1. Read **`PROJECT_STATUS/START_HERE.md`** before writing any code.
2. Skim **`PROJECT_STATUS/phase-log.md`** (first entries). If the latest dated block does not reflect work you are about to ship or just shipped, **prepend the missing entry before writing new feature code** (do not leave the log stale across sessions).
3. Before touching a feature, read **`PROJECT_STATUS/features/[relevant-module].md`** (or `features/app.md` for simple repos).
4. If the `PROJECT_STATUS/` folder does not exist yet, create it using the templates in `project-status-file.md` and migrate any legacy root `PROJECT_STATUS.md` content into **`phase-log.md`**, then replace the root file with the two-line redirect.

---

## The Gate

**Before this session ends — before any `git push`, before any PR — you must update the relevant file(s) under `PROJECT_STATUS/`.**

If you complete a task without updating the folder, the task is not done.

---

## What to Update, Based on What Changed

**Feature added or modified:**
- `PROJECT_STATUS/features/[module].md` — add or update the feature block (all six questions below)
- `PROJECT_STATUS/START_HERE.md` — update build state table (🔨 → ✅ or new row) and `Last updated`
- `PROJECT_STATUS/issues.md` — if a Linear issue changed state

**Schema changed:**
- `PROJECT_STATUS/data-model.md` — affected table(s): columns, types, constraints, business rules
- `PROJECT_STATUS/START_HERE.md` — if module state changed

**New API endpoint (external or contract-critical):**
- `PROJECT_STATUS/api.md` — method, path, auth, description, rate limit

**New external integration:**
- `PROJECT_STATUS/integrations.md` — system, connection, direction, purpose, fallback

**Role or permission changed:**
- `PROJECT_STATUS/roles.md` — affected role(s)

**Architectural decision:**
- `PROJECT_STATUS/decisions.md` — append a new row (**never** edit existing rows)

**Phase or significant milestone completed:**
- `PROJECT_STATUS/phase-log.md` — **prepend** a new dated `###` block **immediately under the first `---` after the title** (most recent first). Do not edit or reorder older entries (**never** rewrite history).

**Minimum for any change (including typos, copy, config):**
- `PROJECT_STATUS/START_HERE.md` — bump **`Last updated`**

---

## The Six Questions (features/*.md Quality Bar)

Every feature block must answer:

1. What does the user see and do?
2. What does the system do in response?
3. Which files are involved? (key paths)
4. What business rules does it enforce?
5. What edge cases are handled?
6. What is explicitly out of scope?

If any question is unanswered, the block is incomplete. Complete it before pushing.

---

## MANDATORY FINAL STEP — do not skip

Before ending the session or running **`git push`** / opening a PR:

1. If you shipped a phase, milestone, or cross-cutting behaviour: **prepend** a new block to **`PROJECT_STATUS/phase-log.md`** (format matches existing `### YYYY-MM-DD — …` entries; `- [x]` bullets; deploy notes if any).
2. Bump **`PROJECT_STATUS/START_HERE.md`** — at minimum **`Last updated`**; build table / status line if module state changed.
3. Confirm files are saved. Cursor rules do **not** run automatically — this checklist is the only enforcement.

---

## Pre-Push Checklist

```
[ ] PROJECT_STATUS/START_HERE.md — build state current, Last updated bumped
[ ] PROJECT_STATUS/features/[module].md — complete feature block(s) for every changed feature
[ ] PROJECT_STATUS/data-model.md — updated if schema changed
[ ] PROJECT_STATUS/api.md — updated if new external-facing endpoint added
[ ] PROJECT_STATUS/integrations.md — updated if new external system added
[ ] PROJECT_STATUS/roles.md — updated if permissions changed
[ ] PROJECT_STATUS/decisions.md — new row appended if architectural decision made
[ ] PROJECT_STATUS/issues.md — Linear / blocker state current
[ ] PROJECT_STATUS/phase-log.md — new block prepended if a named phase or milestone shipped
```

---

## Git Hook (Pre-Push)

Add `.husky/pre-push` in Node repos to block pushes when nothing under `PROJECT_STATUS/` changed:

```bash
#!/bin/sh
CHANGED=$(git diff --name-only HEAD~1 HEAD 2>/dev/null | grep "PROJECT_STATUS")
STAGED=$(git diff --cached --name-only | grep "PROJECT_STATUS")
if [ -z "$CHANGED" ] && [ -z "$STAGED" ]; then
  echo "PUSH BLOCKED: No PROJECT_STATUS file was updated."
  echo "Update PROJECT_STATUS/START_HERE.md (at minimum) before pushing."
  echo "See .cursor/rules/project-status-enforcement.mdc"
  exit 1
fi
echo "PROJECT_STATUS updated — push allowed."
exit 0
```

Install: `npm install -D husky && npx husky init`, then add the script to `.husky/pre-push`.  
Bypass when genuinely needed: `git push --no-verify`.

---

## Claude Project Maintenance

The files in **`PROJECT_STATUS/`** (except **`phase-log.md`**) are attached to the Claude Project for this repo. When files are updated in a push, re-attach the changed files before starting a new Claude session. **`phase-log.md`** is history — not attached to Claude.

---

## What "Complete Feature Documentation" Looks Like

See the **Good** example in the previous revision of this rule (QR Equipment Scan narrative) in `project-status-file.md` feature template — same quality bar inside `features/[module].md`.

---
> Source: [zyntel-co-ltd/mazra](https://github.com/zyntel-co-ltd/mazra) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
