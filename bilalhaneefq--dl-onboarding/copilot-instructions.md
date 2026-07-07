## dl-onboarding

> This repository follows the Disrupt Labs venture-building standards: **STD-002 Venture Knowledge Foundation (VKF)** for knowledge, **STD-001 Spec-Driven Development (SDD)** for change. The sections below tell Claude Code when and how to use each command.

# Project Conventions for Claude Code

This repository follows the Disrupt Labs venture-building standards: **STD-002 Venture Knowledge Foundation (VKF)** for knowledge, **STD-001 Spec-Driven Development (SDD)** for change. The sections below tell Claude Code when and how to use each command.

> **Note:** This is the onboarding template. As you customize it for your venture (filling in the constitution, shipping features), update this CLAUDE.md to reflect your project's specifics — tech stack, conventions, tracker integration, etc.

---

## Standards Stack

| Standard | Governs                                              | Skill                  |
|----------|------------------------------------------------------|------------------------|
| STD-002  | Knowledge (constitution, specs layout, freshness)    | `venture-foundation`   |
| STD-001  | Change (proposals, spec-deltas, tasks, archival)     | `disrupt-sdd`          |
| STD-003  | Venture metrics                                      | (adopted as-needed)    |

**STD-002 is a prerequisite for STD-001** — you need a constitution and a specs/features layout before you can manage changes against them. A fresh fork of this template should run `/vkf/init` first, fill in the Core constitution files (mission, pmf-thesis, principles), and only then start SDD cycles.

---

## Knowledge Operations (STD-002)

### Command Routing Table

When the user asks about or attempts something, consult this table and suggest the right command:

| User says / situation | Command | What it does |
|----------------------|---------|-------------|
| "Initialize" / "Set up VKF" | `/vkf/init` | Bootstrap STD-002 structure |
| "Draft the mission/personas/..." | `/vkf/constitution` | Interactive constitution drafting |
| "We need to change our positioning/mission/..." | `/vkf/amend` | Tiered amendment process (C0–C3) |
| "I have a doc/sheet/notes to add" | `/vkf/ingest` | Classify and place external content |
| "What's missing?" / "What don't we know?" | `/vkf/gaps` | Scan for knowledge gaps with AI-proposed answers |
| "Are our docs up to date?" | `/vkf/freshness` | Check freshness + source staleness |
| "Are we compliant?" / "Check everything" | `/vkf/validate` | Full STD-002 audit |
| "Research competitors/market/..." | `/vkf/research` | Market research (exa.ai) for constitution sections |
| "Here's a meeting recording/transcript" | `/vkf/transcript` | Extract insights, generate meeting brief |
| "Where did this info come from?" | `/vkf/audit --trace` | Trace any section back to its sources |
| "What are our goals this quarter?" | `/vkf/okrs` | View/update quarterly objectives |
| "What needs attention?" | `/vkf/workflow status` | Show document lifecycle and pending actions |
| Pasting content in chat without context | Suggest `/vkf/ingest --inline` | Route pasted content through proper ingestion |
| Editing constitution files directly | Suggest `/vkf/amend` | Ensure proper change governance |

### Before Modifying Knowledge Base Files

Before editing any file in the specs tree, evaluate this decision tree:

1. **Is this a constitution file?** (any `.md` in `specs/constitution/`)
   - YES → Is the file still in Draft state (has `[REQUIRED]` placeholders)?
     - YES → Use `/vkf/constitution` for initial drafting
     - NO → Use `/vkf/amend` — announce: "This is an active constitution section. I'll use the amendment process."
   - NO → Continue normally (feature specs follow SDD)

2. **Is the user providing external content?** (pasting text, sharing a file reference)
   - YES → Announce: "I'll route this through `/vkf/ingest` to classify and place it properly."

3. **Is the user sharing a meeting transcript or recording notes?**
   - YES → Announce: "I'll use `/vkf/transcript` to extract and classify insights."

### VKF — Always / Ask First / Never

**Always:**
- Log every ingestion and transcript extraction to `specs/ingestion-log.yaml`
- Announce the amendment tier (C0 / C1 / C2 / C3) before making constitution changes
- Track "we don't know" as explicit knowledge state, not absence
- Update `Last amended` / `Last reviewed` dates on every document change
- Update `.claude/state/vkf-state.yaml` after significant operations
- Follow commit conventions: `[ingest]`, `[gaps]`, `[constitution]`, `[transcript]`, `[okr]`, `[workflow]`, `[foundation]`, `[validate]`

**Ask first:**
- Applying gap resolution answers that change active constitution content (routes through `/vkf/amend`)
- Archiving OKR quarters
- Transitioning documents from Active to Archived
- Overwriting an existing constitution file that has been filled out
- Running exa.ai research that consumes API credits

**Never:**
- Never overwrite audit log entries (append-only)
- Never delete gap reports (they are historical records)
- Never bypass amendment tiers — even if the user says "just change it", announce the tier
- Never auto-resolve gaps without user review
- Never mark a `[REQUIRED]` section as complete without content
- Never fabricate market data — use `/vkf/research` or clearly label assumptions

---

## Change Operations (STD-001 / SDD)

### Command Routing Table

| User says / situation | Command | What it does |
|----------------------|---------|--------------|
| "I want to work on X" (rough idea) | `/sdd:backlog add "X"` | Queue a lightweight backlog item |
| "Let's dig into X before committing" | `/sdd:explore {slug}` | Deep investigation — creates `wip/{slug}/` |
| "Start a cycle for X" | `/sdd:start {slug}` | Scaffold `changes/{slug}/` with proposal / spec-delta / tasks |
| "Pull the next backlog item" | `/sdd:start --from backlog:{N}` | Promote backlog item to an active cycle |
| "What's the current state?" | `/sdd:status` | Show cycle phase, task progress, commits |
| "Work the next task" | `/sdd:implement` | Execute the next unchecked task |
| "We're done" | `/sdd:complete` | Merge spec-delta → specs/, archive cycle, push, open PR |
| "Process the whole backlog" | `/sdd:run-all` | End-to-end for all queued items |

### Before Modifying Feature Specs

1. **Is there an active cycle in `changes/`?** If yes, write the change into that cycle's `spec-delta.md`, not directly into `specs/features/`. The `spec-delta` merges into canonical spec on `/sdd:complete`.
2. **No active cycle, but a behavior change?** Start one with `/sdd:start {slug}`. Do not edit `specs/features/**/spec.md` directly for behavior changes.
3. **Bug fix that restores specified behavior?** No cycle needed. Fix directly, reference the spec in the commit.
4. **Config / dependency / doc-only change?** No cycle needed.

### SDD — Always / Ask First / Never

**Always:**
- Follow Given/When/Then in spec-deltas — including error scenarios
- Mark tasks `[x]` as they complete; don't batch
- Update `.claude/state/sdd-state.yaml` via the commands, not by hand
- Commit conventions: `[spec]` for spec changes, `[impl]` for code, `[archive]` for cycle archival

**Ask first:**
- Starting a cycle on a branch that isn't clean
- Skipping the proposal for something that looks like a refactor but may change observable behavior
- Completing a cycle when not all Given/When/Then scenarios hold in the implementation

**Never:**
- Never commit implementation before the spec-delta exists
- Never merge a spec-delta into `specs/features/` by hand — use `/sdd:complete`
- Never edit `archive/` contents — cycles are historical once archived

---

## Passive Behaviors

Things to do automatically without being asked:

- **After any ingestion or constitution change:** Remind the user that `/vkf/gaps` can identify what's still missing.
- **When freshness scan shows STALE specs:** Suggest specific actions (re-review, ingest new data, or run gap analysis).
- **When editing constitution files:** Always check amendment tier and announce it — *"This is a C2 (substantive) change because it alters the meaning of the positioning statement."*
- **When OKRs exist and a related spec changes:** Note: *"This change relates to OKR [objective]. Consider updating progress."*
- **When the user asks a question the constitution should answer but doesn't:** Note: *"The constitution doesn't address this yet. Want me to run `/vkf/gaps` to surface this formally?"*
- **When a behavior change is proposed outside an active cycle:** Suggest `/sdd:start` before writing code.

---

## Path Resolution

All VKF commands resolve paths from `.claude/state/vkf-state.yaml`:
- `constitution_root` (default: `specs/constitution`)
- `features_root` (default: `specs/features`)
- `docs_root` (default: `docs`)

Override these only if your repo deviates from the defaults.

---

## Tech Stack (customize for your venture)

> **Replace this section with your venture's stack once chosen.** Example scaffolding:
>
> - Language / runtime: [e.g., TypeScript + Node.js 20]
> - Framework: [e.g., Next.js, FastAPI, Go + Chi]
> - Database: [e.g., Postgres via Supabase]
> - AI/LLM provider: [e.g., Anthropic Claude via Vercel AI SDK]
> - Deploy target: [e.g., Vercel / Fly.io / AWS]
> - Dev port: [e.g., 3000; document any reserved ports]

---

## Tracker Integration (customize)

- **Default:** `tracker_preference: "local"` in `.claude/state/sdd-state.yaml` — backlog lives in `backlog/items.yaml`.
- **Linear / GitHub Issues via MCP:** switch `tracker_preference` to `"linear"` or `"github"` when a tracker is configured. Backlog items then live in the tracker; `/sdd:start --from {issue-id}` pulls them.

---

## Onboarding (new engineer? start here)

1. Read [`README.md`](README.md) for the 30-second overview.
2. Work through [`ONBOARDING.md`](ONBOARDING.md) — day-1 to first shipped cycle.
3. Score yourself against [`CHECKLIST.md`](CHECKLIST.md) — that's the bar.
4. Capture what surprised you in `learnings.yaml` via `/sdd:complete`.

---
> Source: [BilalHaneefQ/dl-onboarding](https://github.com/BilalHaneefQ/dl-onboarding) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
