## long-term-agent-memory

> This Obsidian vault is a long-term external memory system for an AI agent and the user.

# AGENTS.md

This Obsidian vault is a long-term external memory system for an AI agent and the user.

## Purpose
- Preserve durable knowledge beyond the model context window.
- Store distilled knowledge, not raw chat dumps.
- Optimize for retrieval, linking, and maintenance.

## Memory layers

### 1. Working memory
- Folder: `inbox/`
- Temporary captures, scratch notes, partial ideas, session staging.
- Can be reorganized, merged, or deleted after promotion.

### 2. Episodic memory
- Folder: `sessions/`
- One note per meaningful work session.
- Record what was attempted, decisions made, failures, outcomes, and follow-ups.

### 3. Semantic memory
- Folders: `projects/`, `concepts/`, `entities/`, `references/`
- Stable facts, definitions, project knowledge, user preferences, tool conventions.

### 4. Procedural memory
- Folder: `procedures/`
- Reusable workflows, setup guides, debugging playbooks, operating instructions.

### 5. Decision memory
- Folder: `decisions/`
- Architecture decisions and important policy choices.

## Rules
- Prefer updating an existing canonical note over creating duplicates.
- Use Obsidian wikilinks like `[[Home]]` and `[[index]]`.
- Create atomic notes when possible.
- Keep source-derived claims linked to supporting notes in `references/`.
- Use `log.md` as append-only operational history.
- Update `index.md` when adding a durable top-level knowledge area.
- Do not store secrets in plaintext.
- Do not use this vault as a raw transcript dump.

## Recommended note types
- Project notes: `projects/`
- Concept notes: `concepts/`
- Entity notes: `entities/`
- Reference/source notes: `references/`
- Procedure notes: `procedures/`
- Session notes: `sessions/`
- Decision notes: `decisions/`
- Analysis notes: `analyses/`

## Note quality standard
A durable note should answer some of these:
- What is this?
- Why does it matter?
- How does it relate to other notes?
- Where did this come from?
- How confident are we?
- What should be updated when this changes?

## Maintenance workflow
1. Capture in `inbox/` or `sessions/`.
2. Distill into durable notes.
3. Link to related notes.
4. Merge duplicates.
5. Archive or delete low-value temporary notes.

## Current policy
- This vault starts as a general persistent memory system for an AI agent and the user.
- The canonical entry points are `[[Home]]`, `[[index]]`, and `[[log]]`.

---
> Source: [eslamgenio/long-term-agent-memory](https://github.com/eslamgenio/long-term-agent-memory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-27 -->
