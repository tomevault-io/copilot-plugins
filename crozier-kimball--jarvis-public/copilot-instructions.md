## jarvis-public

> This repository is a **Minimum Viable Jarvis** workspace: an agentic personal business OS for the owner.

# AGENTS.md

This repository is a **Minimum Viable Jarvis** workspace: an agentic personal business OS for the owner.

## Purpose

Maintain the owner's operational truth across relationships, strategy, decisions, and state by keeping markdown files accurate, current, and cross-referenced.

## Startup Orientation (Run On Load)

1. Read `CLAUDE.md` first.
2. Build read-context from the Obsidian link graph within these parent folders only: `.obsidian/`, `artifacts/`, `meeting-transcripts/`, `people/`, `skills/`, `user/`.
3. Use folder paths only to decide write destinations after context is selected via graph traversal.

## Default Behavior: Brain Dump First

Any owner input — question, thought, debrief, update, decision — should be treated as a brain dump first. Route it before (or alongside) responding. No command required.

**Write routing:**
- Person mentions → `people/<First Last>.md` (update or create)
- Strategic thoughts, decisions, status → `artifacts/` (date-prefix when relevant)
- Tasks or commitments → `artifacts/Tasklist.md`
- Transcript content → direct owner to `meeting-transcripts/raw/`, run `/readmeeting`; never process manually

**How to propose writes:**
- One line per file: "I'll update X with Y — okay?" Keep it lightweight unless changes are complex.
- Write after approval. Flag anything you noticed but didn't route.

**Skills are for complex SOPs only.** If the owner must remember a command to do something routine, that's a gap to fix in routing rules, not a reason to build another skill.

## Graph-Augmented Routing

Before routing a brain dump, use the Obsidian link graph to load only the minimum context needed:
1. Identify seed nodes from the input (named people, projects, dates, topics).
2. Load seed files + neighbors via wiki-link traversal.
3. Use that graph context to determine which files to update and what connections to add.
4. Write routing decisions are still folder-based (see above); graph traversal is for read context only.

## Graph-First Context Policy

- Read scope is restricted to graph-addressable notes under: `.obsidian/`, `artifacts/`, `meeting-transcripts/`, `people/`, `skills/`, `user/`.
- Use folders to decide where to write; use the link graph to decide what to read.
- Load minimum context. Start from seed nodes in the owner's input, expand via wiki-link traversal, stop when you have enough to act.
- Preserve graph quality: stable canonical links, no duplicate person notes, every new file linked from at least one existing note.
- If `AGENTS.md` or `CLAUDE.md` is updated, update the other file in the same change so routing rules stay aligned.

## Skills Behavior

When owner invokes `/[skillname]`, read and execute `skills/[skillname].md` as an SOP.
Skills are invocation-gated: they only apply during that explicit run and must not retroactively govern prior files or decisions.
Do not apply rules from a skill to historical outputs unless the owner explicitly asks for a retrofit.

Current known skills:
- `/today`
- `/readmeeting`
- `/buildskill`
- `/wrap` — end-of-session git checkpoint: stage, commit, optional push
- `/endday` — end-of-day review: collect notes, repair wikilinks, review tasklist, write daily summary
- `/auditproject` — audit CLAUDE.md and AGENTS.md for drift against original MVJ baseline
- `create-user-profile` (automatic in first session)

## Meeting Transcript Rules

- Do not manually process raw transcripts in chat.
- Raw inputs belong in `meeting-transcripts/raw/`.
- `/readmeeting` creates processed notes in `meeting-transcripts/meetings/` and handles lifecycle.

## File Standards

- Mixed files will be uploaded. Only ingest .md files unless otherwise instructed by user or a skill. .md outputs should be set up to use in Obsidian.
- Use descriptive kebab-case file names for artifacts, transcripts, and skills.
- People files use Title Case with spaces matching the person's name (e.g., `Jane Smith.md`).
- Use `YYYY-MM-DD-descriptor.md` for date-relevant artifacts/transcripts.
- Keep docs concise and high-signal.
- Maintain cross-references (especially meeting mentions -> people files).

## Link Integrity Rules

- Before and after large ingest/update tasks, run a quick unresolved/ambiguous wiki-link scan.
- Do not introduce basename-ambiguous canonical notes (for example two different `Tasklist.md` files).
- For people mentions in strategic docs, prefer linking to an existing note instead of plain text names.
- Keep canonical hub links stable (`Tasklist`, workstreams, core people) so graph routing stays reliable.

## Operating Principles

- Owner is source of truth: propose changes, owner approves truth.
- Ask when uncertain; do not fabricate details.
- Optimize for compounding memory: each update should improve future usefulness.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/crozier-kimball) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
