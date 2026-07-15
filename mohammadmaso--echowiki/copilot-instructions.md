## echowiki

> A full reimplementation of [VectifyAI/OpenKB](https://github.com/VectifyAI/OpenKB) — an open-source LLM Knowledge Base compiler — built with the **Vercel AI SDK** (TypeScript/Node) instead of OpenKB's original Python stack (OpenAI Agents SDK + LiteLLM).

# Project Context & Rules — OpenKB-for-Obsidian (Vercel AI SDK)

## What this project is
A full reimplementation of [VectifyAI/OpenKB](https://github.com/VectifyAI/OpenKB) — an open-source LLM Knowledge Base compiler — built with the **Vercel AI SDK** (TypeScript/Node) instead of OpenKB's original Python stack (OpenAI Agents SDK + LiteLLM).

A single `raw/` folder is the universal ingestion point: voice recordings (transcribed via STT) and manually authored/dropped Markdown/text files both land there and are compiled identically. The wiki compiler compiles everything in `raw/` into a structured, interlinked `wiki/` — summaries, concept pages, entity pages, cross-links — with OKF-schema frontmatter, viewable natively as an Obsidian vault.

See `01-PRD.md` and `02-TECH-SPEC.md` for full product/technical context. Follow `03-TASKS.md` task-by-task; do not skip ahead or bundle multiple tasks into one change.

## Reference Implementation
Treat [VectifyAI/OpenKB](https://github.com/VectifyAI/OpenKB) as the architectural reference. Key concepts to preserve when porting to Mastra:
- The `raw/ → wiki/` compilation pipeline, and the `wiki/` folder shape: `index.md`, `log.md`, `AGENTS.md`, `sources/`, `summaries/`, `concepts/`, `entities/`, `explorations/`, `reports/`.
- The compilation steps per new document: summary → read existing concepts/entities → create/update concepts → create/update entities → update index/log.
- `wiki/AGENTS.md` as the LLM's own instruction manual for wiki structure/conventions, read from disk at runtime (so edits take effect immediately, no redeploy).
- OKF-schema YAML frontmatter on every generated page.
- Obsidian-native output: plain Markdown + `[[wikilinks]]`, no proprietary format.

## Tech Stack
- Vercel AI SDK (`ai` + `@ai-sdk/openai`) for LLM calls — compilation runs in-process inside the Obsidian plugin (no child process / HTTP server).
- Node-native file watching (`chokidar` or `fs.watch`) as the functional analog of upstream's `watchdog`-based `openkb watch`.
- HTTP calls to user-configured, OpenAI-compatible STT and LLM endpoints — no hardcoded provider.
- Voice capture UI: Obsidian plugin (Obsidian Plugin API) if integrated into Obsidian directly, or a standalone companion app if decoupled — this is still an open decision (see PRD open questions); do not assume one without confirming.

## Core Conventions
- **`raw/` is source-agnostic.** Voice transcripts and manually authored notes must be treated identically by the compilation pipeline — never special-case "this came from voice" inside compilation logic.
- All AI provider config (LLM and STT: base URL, API key, model) must be read from settings/config — never hardcode a provider or endpoint.
- Treat "local-first" as a hard constraint: raw files and wiki output live on disk first; external calls only happen for STT/LLM steps the user explicitly triggers.
- The approval gate (`requireApproval`) must be respected uniformly across both ingestion paths.
- Compilation must **update existing concept/entity pages**, not create duplicates, when new documents touch the same topic/entity — page identity/matching logic is a first-class concern, not an afterthought.
- `wiki/AGENTS.md` defines the compilation agent's instructions and must be read from disk at runtime, not baked into code.
- All generated wiki pages must carry OKF-schema YAML frontmatter and use `[[wikilinks]]` for cross-references.
- Never silently overwrite a manually edited wiki page — surface a conflict instead (exact handling is still an open decision — flag it rather than guessing silently).

## Code Style
- Idiomatic TypeScript, strict mode.
- Keep these concerns in separate modules, not one file: (1) ingestion (voice capture + raw file handling), (2) folder watcher/workflow, (3) compilation agent logic (summary/concept/entity steps), (4) wiki file I/O.
- Prefer small, testable functions per compilation step.

## What NOT to do
- Don't build voice-only special paths — every ingestion path must converge on the same `raw/` → compilation flow.
- Don't hardcode a specific AI provider — must stay compatible with any OpenAI-compatible endpoint, including self-hosted.
- Don't implement PageIndex-style long-document tree indexing, non-Markdown format support (PDF/Word/PPT/Excel/URLs), or Query/Chat/Skill Factory generators — all out of scope for MVP (see PRD §7 and Tasks "Stretch Goals").
- Don't build multi-user/team features — single local vault only for MVP.
- Don't skip the manual test step listed under each task in `03-TASKS.md` before moving to the next task.

## Working Style for This Session
- Implement one task from `03-TASKS.md` at a time, in order, unless told otherwise.
- After each task, briefly state what was implemented and how to manually verify it (matches the "Manual test" line in the task list).
- Ask before making an architectural decision that isn't already settled in `02-TECH-SPEC.md` (e.g., voice-UI integration point, note-collision handling, wiki-folder-structure scope).

---
> Source: [mohammadmaso/echowiki](https://github.com/mohammadmaso/echowiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-15 -->
