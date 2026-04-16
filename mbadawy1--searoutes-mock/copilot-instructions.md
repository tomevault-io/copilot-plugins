## searoutes-mock

> > **Purpose:** Give Gemini CLI strict project context so Claude can delegate **large codegen / multi-file edits / refactors** safely.

# GEMINI.md — Heavy-Lifting Assistant (used by Claude as the conductor)

> **Purpose:** Give Gemini CLI strict project context so Claude can delegate **large codegen / multi-file edits / refactors** safely.  
> **Source of truth:** If anything here conflicts with **AGENTS.md**, **AGENTS.md wins**.

---

## Project Snapshot (mirror of AGENTS.md)
- **App:** Shipping Line Schedule App (backend FastAPI; frontend separate)
- **Current Focus:** Task 3 — Fixtures Loading → then Task 4 — Basic GET endpoint
- **Branch:** `feat/fixtures-provider`
- **One-task rule:** One task → one PR → merge → next task

---

## Contracts You Must Honor
These are **hard constraints** when proposing patches.

### API envelope
All list endpoints return:
```json
{ "items": [...], "total": <int>, "page": <int>, "pageSize": <int> }
```

### Sorting & filtering
- Sorting is **server-side only** via `?sort=etd|transit` (default: `etd`).
- UI toggles only change the query param; server re-fetches with new order.

### Exports (CSV/XLSX)
- Exports reflect **exactly the filtered & sorted dataset**, not just the current page.
- **Column order (default unless AGENTS.md says otherwise):**  
  `originLocode,destinationLocode,etd,eta,vessel,voyage,carrier,routingType,transitDays,service`

### Query param aliases
- Accept `from` and `to` in the public API but map internally to `date_from` and `date_to` (Pydantic aliases).

### Secrets
- **No provider keys in client code.** Live provider keys live on the server only.

---

## Scope Limits (strict)
- ≤ **~300 LOC** and **≤ 5 files** changed per task/PR (unless the task explicitly allows more).
- Prefer **editing existing files**; create **only** files the task requires.
- When in doubt, propose the **smallest viable patch** and add a `TODO` for follow‑ups.

---

## What Gemini Should Do (and Not Do)
**Do:**
- Generate **minimal unified diffs** or complete file bodies for exactly the files in scope.
- Keep code idiomatic and consistent with the repo style (ruff/black for Python).
- Include a **quick verification step** (curl command or pytest snippet) per task.

**Don’t:**
- Change API envelopes, CSV columns, routing rules, or task order.
- Introduce secrets, new external services, or sweeping rewrites outside the task.
- Edit more than 5 files or exceed ~300 LOC for a single task.

---

## Tasks (high-level cues for Gemini)
- **Task 3 — Fixtures Loading (current):**  
  Provider returns sample schedules from disk; ignore filters; aim for deterministic fixtures to unblock the UI.
- **Task 4 — Basic GET endpoint (next):**  
  Implement `/api/schedules` that returns fixtures using the envelope above.

> If a task requires details not stated here, **read AGENTS.md**. If still ambiguous, prefer the **smallest** change and document assumptions.

---

## Reusable Slash Commands (used by Claude in Gemini CLI)

### `/plan` — summarize and bound the work
Goal: produce a short, auditable plan **within scope** for the **top task in AGENTS.md**.

**Prompt:**
```
Read AGENTS.md (top task only). Output:
1) Task summary
2) Files to touch (≤5) and why
3) LOC estimate (≤300)
4) Risks/edge cases and how to stay within contracts
Do NOT propose changes outside the current task.
```

### `/implement` — emit the patch
Goal: generate **unified diffs** (preferred) or full file replacements for only the files listed in `/plan`.

**Prompt:**
```
Based on /plan and the contracts:
- Produce UNIFIED DIFFs (or full file contents if clearer)
- ≤ ~300 LOC, ≤ 5 files
- Preserve API envelope, sorting rules, export column order
- Include a quick verification snippet (curl/pytest)
```

> These slash commands are configured as TOML files in `.gemini/commands/` (see CLAUDE.md for exact paths).

---

## Example Verification Snippets
- **Server runs:** `uvicorn backend.app.main:app --reload`
- **Health:** `curl -s http://localhost:8000/health`
- **Schedules (fixtures):** `curl -s 'http://localhost:8000/api/schedules?sort=etd' | jq .page`

---

## Stop Conditions (halt & return control to Claude)
- Task success criteria not met with a small patch
- Any contract would be broken
- Scope would exceed limits
- Unclear requirements not covered in AGENTS.md

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mbadawy1) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
