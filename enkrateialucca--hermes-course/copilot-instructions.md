## hermes-course

> > Example agent instruction file. Paths use `~` and placeholders — replace with your own.

# AGENTS.md

> Example agent instruction file. Paths use `~` and placeholders — replace with your own.

## Paths

- `~/Desktop/automated` — main Python/shell automation scripts.
- `~/Desktop/projects/<trainings-repo>` — repos for course/training material.
- `~/Desktop/projects/<landing-page-repo>/docs/blog/posts` — blog posts (MkDocs blog). If asked to write a blog post, write it there and follow the guidelines in the root of that project.
- `~/Desktop/projects/<ref-docs-repo>/ui-design/brand-guidelines.pdf` — brand style guidelines.
- `~/notes/` — Obsidian vault.
- `~/Library/Application Support/Anki2/<profile>/collection.anki2` — Anki deck (media folder: `~/Library/Application Support/Anki2/<profile>/collection.media/`). Note: profile names are device-local, so the same synced profile can appear under a different name on mobile.
- `~/Library/CloudStorage/GoogleDrive-<account>/My Drive/Meet Recordings` — meeting recordings. If asked to transcribe "the latest meeting" with no other context, use the most recent file there.

## Commands

Transcription:

- `transcribe file.mp4` — JSON only
- `transcribe file.mp4 --process` — readable txt
- `transcribe file.mp4 --all-formats` — json + srt + vtt
- `transcribe file.mp4 --pt` — Portuguese

## Conventions

- Always ask before deleting multiple files.
- When generating Python scripts, unless instructed otherwise, use `uv` inline script metadata (`requires-python`, dependencies, etc.).
- Every project has a central note starting with `Project - ...` in the Obsidian vault.
- When reporting information back, be extremely concise.

## Wiki Knowledge Base

Path: `~/notes`

When you need personal/project context not already in the current project:

1. Read `wiki/hot.md` first (recent context cache, ~500 words)
2. If not enough, read `wiki/index.md` (full catalog)
3. If you need domain details, read the relevant `wiki/domains` page or sub-index
4. Only then drill into specific wiki pages

Do NOT read the wiki for general coding questions or tasks unrelated to personal knowledge/projects.

---
> Source: [EnkrateiaLucca/hermes-course](https://github.com/EnkrateiaLucca/hermes-course) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
