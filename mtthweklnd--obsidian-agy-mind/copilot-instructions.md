## obsidian-agy-mind

> A persistent personal knowledge base and second brain for notes, ideas, projects, and reflections.

# Obsidian Mind — Personal Knowledge Base

A persistent personal knowledge base and second brain for notes, ideas, projects, and reflections.

## Vault Layout

- `brain/` — Living personal context (`North Star.md`, `Key Decisions.md`, `Patterns.md`, `Gotchas.md`, `Memories.md`, `Skills.md`).
- `knowledge/` — Permanent knowledge (`concepts/`, `topics/`, `books-and-media/`, `reference/`).
- `projects/` — Goal-directed personal initiatives (`active/`, `archive/`, `Index.md`).
- `journal/` — Temporal cadence (`daily/YYYY-MM-DD.md`, `weekly/`).
- `thinking/` — Ephemeral drafts in `drafts/` and auto-backed session transcripts in `session-logs/`.
- `templates/` — Markdown note templates with YAML frontmatter schemas.

## Antigravity Hooks Pipeline

The hook scripts in `.agents/scripts/hooks/` are agent-agnostic TypeScript scripts executed natively by Node.js via `--experimental-strip-types`:

| Script | Purpose | Event Trigger |
| :--- | :--- | :--- |
| `session-start.ts` | Injects North Star goals, active projects, and file tree under byte budget | `session_start` / `SessionStart` |
| `classify-prompt.ts` | Classifies prompt intent (`[CONCEPT]`, `[PROJECT]`, `[DECISION]`, `[JOURNAL]`) | `user_prompt_submit` / `BeforeAgent` |
| `validate-write.ts` | Validates YAML frontmatter and `[[wikilinks]]` on markdown writes | `post_tool_execution` / `AfterTool` |
| `pre-compact.ts` | Backs up conversation transcript to `thinking/session-logs/` before compaction | `pre_compact` / `PreCompress` |
| `stop-checklist.ts` | Displays daily wrap-up checklist at end of session | `session_end` / `Stop` |

## Antigravity Skills

Invoke as slash commands (e.g. `/om-daily`, `/om-dump`, `/om-concept`, `/om-wrap-up`, `/om-vault-audit`):
- `/om-daily` — Morning kickoff with active projects, tasks, and North Star focus.
- `/om-dump` — Process unstructured personal brain dumps into atomic concepts, project notes, or decisions.
- `/om-concept` — Create or refine an atomic evergreen concept note with bidirectional links.
- `/om-book-summary` — Ingest book/article takeaways and mental models into `knowledge/books-and-media/`.
- `/om-decision` — Document a significant personal or technical decision in `brain/Decisions.md`.
- `/om-wrap-up` — Evening review, link verification, daily summary.
- `/om-weekly` — Weekly synthesis across active projects and North Star.
- `/om-vault-audit` — Health report for orphan notes, dead links, and unpromoted drafts.
- `/om-tidy` — Fix formatting, wikilinks, and file organization.
- `/om-project-archive` — Move completed projects to `projects/archive/YYYY/`.
- `/om-humanize` — Polish note tone into natural, concise language.

## PKM Subagents

Specialized agents running in isolated contexts:
- `vault-librarian` — Audits vault health, orphan notes, broken links, and unpromoted drafts.
- `cross-linker` — Finds and builds connections between related concepts, projects, and mental models.
- `context-loader` — Retrieves relevant personal notes, principles, and previous decisions.
- `concept-synthesizer` — Distills raw notes and dumps into atomic evergreen concept notes.
- `vault-migrator` — Imports and restructures external notes into this vault format.

---
> Source: [mtthweklnd/obsidian-agy-mind](https://github.com/mtthweklnd/obsidian-agy-mind) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-29 -->
