## pax-automata

> This file defines the **permanent operating rules** for Claude Code in this repo. It is the "Rules of the Land": immutable conventions. It is **not** a scratchpad or status log. Do not store task lists or progress here—use `docs/project-tracker.md` for that.

# Pax-Automata — Claude Code Constitution

This file defines the **permanent operating rules** for Claude Code in this repo. It is the "Rules of the Land": immutable conventions. It is **not** a scratchpad or status log. Do not store task lists or progress here—use `docs/project-tracker.md` for that.

---

## 1. Project Identity

- **Repo:** Pax-Automata — autonomous agent for **Pax Historia** (browser grand-strategy game).
- **Architecture:** Cognitive loop: **Perception (Spy)** → **War Room (local files)** → **Brain (LLM)** → **Execution (Hand)**.
- **Source of truth for product/design:** `docs/PRD.md`, `docs/diagrams/war-room-flow.mmd`, `docs/diagrams/war-room-flow.png`.

---

## 2. Tech Stack (Canonical)

| Layer        | Choice              | Notes |
|-------------|---------------------|--------|
| Browser automation | **Playwright** | Intercept `/api/simple-chat`, drive UI (action box, advisor box). |
| Runtime     | **Node.js + TypeScript** | Preferred for Playwright and tooling; use Python only if explicitly required. |
| LLM         | **Anthropic Claude API** (via SDK) | Reasoning and batch-of-actions generation. |
| Validation  | **Zod**              | Schema validation at all JSON boundaries. |
| TS execution | **tsx**             | Fast TypeScript execution (replaces ts-node). |
| War Room storage | JSON + Markdown + TXT in `war-room/` | See folder layout below. |

---

## 3. Folder Structure (Canonical Paths)

```
/
├── CLAUDE.md                 # This file — constitution only.
├── docs/
│   ├── PRD.md                # Product/technical spec (read first for features).
│   ├── project-tracker.md    # Status scratchpad: done vs pending (update here, not in CLAUDE.md).
│   ├── diagrams/
│   │   ├── war-room-flow.mmd # Mermaid sequence diagram.
│   │   └── war-room-flow.png # Rendered diagram.
│   └── logs/                 # Timestamped logs for refactors/debug sessions (e.g. 2026-02-07-perception-layer.md).
├── war-room/                 # Agent identity & memory (in-repo templates; runtime may copy or symlink).
│   ├── constitution.md       # Long-term goals (fixed).
│   ├── crisis_handbook.txt   # Tactical playbook.
│   ├── strategic_ledger.json # Active plans / long-term memory.
│   └── current_state.json    # Latest game state (written by Spy).
├── src/                      # Application code.
│   ├── spy/                  # Perception layer (Playwright intercept, state parsing).
│   ├── brain/                # Reasoning layer (context assembly, LLM calls, action generation).
│   ├── hand/                 # Execution layer (Playwright UI automation).
│   └── shared/               # Types, Zod schemas, utilities shared across modules.
└── scripts/                  # One-off or scaffold scripts (e.g. scaffold.sh).
```

**Search / references:** Prefer `docs/PRD.md`, `docs/diagrams/war-room-flow.mmd`; War Room files live under `war-room/`.

---

## 4. Naming Conventions

- **Files:** `kebab-case` for scripts and config; `snake_case` for War Room data files to match PRD (`current_state.json`, `strategic_ledger.json`).
- **Branches:** `feature/<short-name>`, `fix/<short-name>`, or `chore/<short-name>` (e.g. `feature/perception-layer`, `fix/ledger-schema`, `chore/deps`).
- **Commits (on feature branches):** Clear, present-tense messages; meaningful and scoped (checkpoints are fine, but avoid noisy micro-commits).

---

## 5. Pre-flight Rule

**Before creating any new file or module**, check this file (`CLAUDE.md`) to confirm the correct path, naming convention, and module boundary. If the file doesn't fit the canonical structure, stop and discuss.

---

## 6. Coding Standards

- **Functional-first:** No classes unless there's a clear reason (e.g. Playwright Page wrapper). Prefer plain functions and modules.
- **Zod at boundaries:** Every JSON file read/write and every external API response gets a Zod schema. Validate, don't assume.
- **Explicit return types** on all exported functions.
- **Barrel exports:** Each module directory (`spy/`, `brain/`, `hand/`, `shared/`) has an `index.ts` re-exporting its public API.
- **No `any`:** Use `unknown` + Zod parsing instead. TypeScript strict mode.
- **Imports:** Prefer relative imports within a module; use `../shared/` for cross-module shared code.

---

## 7. Architecture — Data-Flow Contracts

The cognitive loop passes data through well-defined boundaries:

```
Spy (Playwright intercept)
  → writes current_state.json (Zod-validated GameState)
  → emits "new-turn" event

Brain (context assembly)
  ← reads current_state.json, constitution.md, crisis_handbook.txt, strategic_ledger.json
  → calls Anthropic Claude API with assembled prompt
  → receives ActionBatch (Zod-validated)
  → updates strategic_ledger.json

Hand (Playwright UI driver)
  ← receives ActionBatch from Brain
  → types each action into the action box, submits
  → optionally queries advisor, returns advice to Brain
```

Each boundary has a Zod schema. Modules communicate through files (War Room) and function calls — no shared mutable state.

---

## 8. Workflow + Git Rules

- **Branch first:** Always do work on a feature branch, never directly on `main`. Use branch names like `feature/...`, `fix/...`, `chore/...`.
- **Commits:** You may create local commits as checkpoints. Keep them meaningful and scoped. **Never push** to the remote—pushing is always a human action.
- **No AI attribution:** Do not add any AI attribution trailers (e.g. "Co-authored-by") to commit messages. Messages should look human.
- **Before any commit,** review changes with `git diff` and `git diff --staged`.
- **Merge to main:** The human performs `git merge --squash` (or equivalent) into `main` after review. One clean, human-readable commit per feature on `main`.
- **Secrets:** Do not commit API keys, `.env`, or secrets; keep them in `.gitignore` and/or env only.

---

## 9. Memory Rules (Dual-Document System)

| File / Location      | Type        | Purpose |
|----------------------|------------|---------|
| `CLAUDE.md`          | Constitution | Immutable rules: stack, paths, naming, git, behavior. |
| `docs/project-tracker.md` | Status     | Scratchpad: what is finished vs pending; checklists. |
| `docs/logs/`         | History     | Timestamped logs for major refactors or debug sessions. |

Do **not** use `CLAUDE.md` for task lists or progress. Update `docs/project-tracker.md` instead. Read the tracker at the start of a task and update it at the end. Logs in `docs/logs/` should capture decisions and debugging steps, not raw transcripts.

---

## 10. Session Hygiene

- Use **`/clear`** when switching to a new, unrelated task (e.g. finishing UI work and starting database work).
- Use **`/compact`** during long debugging/refactor sessions to reduce context bloat; preserve key state, discard raw logs.
- Keep tasks scoped. Avoid mixing multiple unrelated objectives in one session.

---

## 11. Editing + Safety

- Prefer **Plan Mode** for complex or multi-file changes. Always propose a plan before large refactors.
- Do not modify unrelated files. Avoid repo-wide formatting or drive-by edits.
- Be cautious with high-churn files unless explicitly requested: lockfiles (`package-lock.json`, `pnpm-lock.yaml`, etc.), generated build artifacts (`dist/`, `build/`).

---

## 12. Quality Gates

- Keep `main` in a working, deployable state.
- Before merging work into `main`, ensure: tests pass; typecheck/lint passes (if applicable); diff is reviewed and scoped.

---

## 13. Parallel Work (Advanced)

- If running multiple Claude sessions, isolate work using Git worktrees: one branch + one worktree per task; avoid overlapping edits on the same files.

---

## 14. Behavioral Guardrails (#)

These rules are permanent. Follow them unless the user explicitly overrides in the current conversation.

- **# No Diary:** Do not write narrative logs or diary entries into `CLAUDE.md` or the constitution. Use `docs/project-tracker.md` or `docs/logs/` for progress and history.
- **# Project Tracker:** For status and checklists, update `docs/project-tracker.md` only. Do not duplicate that content into `CLAUDE.md`.
- **# Git Safety:** Never commit API keys, `.env` files, or secrets. Do not push to `main` directly; work on `feature/` or `fix/` branches.
- **# Test Before Implementation:** Prefer writing a test (or test contract) before implementing new behavior, when tests are applicable.
- **# Restraint on UI:** Do not change UI or front-end behavior unless explicitly asked; when in doubt, focus on logic and structure.
- **# Context Control:** When the user tags a file with `@` (e.g. `Refactor @auth.ts`), restrict changes to that file or explicitly scoped call sites unless the user asks for broader changes.
- **# Constitution Integrity:** `CLAUDE.md` is the Constitution (immutable rules), not a diary. Do NOT update it with current status, debugging notes, or temporary thoughts. Only update it if the architectural patterns or tech stack explicitly change.
- **# Tracker Discipline:** When executing a complex task, maintain status in `docs/project-tracker.md`. Read it at the start of a task; update the checklist at the end. Read other reference files (e.g. the Mermaid diagram) if confused.
- **# Log Preservation:** For major refactors or debugging sessions, do not overwrite previous notes. Create a new file in `docs/logs/` with the timestamp (e.g. `docs/logs/2026-02-07-auth-debug.md`). Capture decisions and debugging steps, not raw transcripts.

---

## 15. Default Operating Principle

Claude should behave like a careful engineering teammate: small, reversible steps; explicit plans for complex work; branch isolation; human-controlled pushes; repo stays clean and readable.

---

## 16. Day-One Checklist (Human)

When opening this repo for a new work session:

1. Run or review the bootstrap/scaffold (e.g. `./scripts/scaffold.sh`) if the tree is missing `war-room/` or `docs/`.
2. Verify `CLAUDE.md` and that Workflow + Git Rules and Memory Rules are present.
3. Check `docs/project-tracker.md` for current status and next tasks.
4. Create a `feature/<name>`, `fix/<name>`, or `chore/<name>` branch before having Claude make changes.

---

*Last bootstrapped from docs/PRD.md and the Senior Engineer Operating Manual. Keep this file minimal and stable.*

---
> Source: [phillipyan300/Pax-Automata](https://github.com/phillipyan300/Pax-Automata) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
