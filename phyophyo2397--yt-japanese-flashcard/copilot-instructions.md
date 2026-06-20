## yt-japanese-flashcard

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

YouTube Japanese Flashcard Generator — a 4-layer Claude Code pipeline that accepts a YouTube URL, extracts Japanese transcript via MCP, identifies JLPT N5–N3 vocabulary via subagent, transforms into flashcard JSON via skill, and displays in a static HTML UI. No auth, no database, session-based only.

Full specification, plan, data model, contracts, and tasks live in `specs/001-youtube-flashcard-generator/`. Read `specs/001-youtube-flashcard-generator/plan.md` for the canonical architecture reference.

## Commands

### MCP Transcript Extraction

Two YouTube MCP tools are available:

```bash
# Primary (language-aware):
mcp__youtube-transcript__get_transcript --url "<URL>" --lang "ja" --include_timestamps true --strip_ads true

# Fallback (legacy):
mcp__youtube__download_youtube_url --url "<URL>"
```

Always try `youtube-transcript` first with `lang: "ja"`, fall back to `youtube` MCP on failure. Both produce subtitle text with timestamps.

### yt-dlp CLI Transcript Download (separate from MCP)

`yt-dlp` is a command-line tool (`brew install yt-dlp`), NOT an MCP server. It downloads subtitles to `.srt` files on disk. Two ways to use it:

```bash
# Quick one-off:
./yt-ja "<YouTube URL>"
# → writes {title}[{video_id}].ja.srt to project root

# Batch / skill-driven:
# Invoke the subtitle-downloader skill — searches YouTube and downloads .ja.srt for top-N results
# Writes to subtitles/{channel_name}/{title}.ja.srt with manual→auto fallback per video
```

See `skills/subtitle-downloader/SKILL.md` for the full skill definition. The subtitle-downloader is a standalone tool — it is NOT part of the 4-layer flashcard pipeline. The flashcard pipeline uses MCP for in-memory transcript extraction.

### Flashcard Pipeline

The entry point is `skills/flashcard-generator/SKILL.md`. Invoke this skill to run the full pipeline:
1. Validate YouTube URL
2. Extract transcript (MCP with fallback)
3. Invoke vocabulary-extractor subagent
4. Generate FlashcardDeck JSON → `output/flashcards-{video_id}.json`

### View Flashcards

Open `ui/index.html` in a desktop browser. By default it loads `ui/sample-data/live-test.json`. Swap the `src` attribute on the `<script id="deckData">` tag to load a different FlashcardDeck JSON file.

## Architecture

```
YouTube URL
  → L1: YouTube MCP (transcript extraction, contracts/transcript-contract.md)
        Primary: youtube-transcript MCP (lang: "ja")
        Fallback: youtube MCP
  → L2: vocabulary-extractor subagent (vocabulary identification, contracts/vocabulary-contract.md)
  → L3: flashcard-generator skill (card generation, contracts/flashcard-contract.md)
  → L4: static HTML UI (interactive study, ui/index.html + app.js + flashcards.css)
```

Each layer boundary has a formal contract in `specs/001-youtube-flashcard-generator/contracts/`. The full workflow diagram is at `specs/001-youtube-flashcard-generator/workflow.md`.

**Separate tool:** The `subtitle-downloader` skill (`skills/subtitle-downloader/SKILL.md`) uses `yt-dlp` CLI (not MCP) to batch-download `.ja.srt` files. It is standalone — not part of the 4-layer pipeline above.

### Data entities (see `data-model.md` for full schemas)

- **TranscriptBundle**: `{ video: {...}, segments: [{text, start_time, end_time, segment_index, source}] }`
- **VocabularyList**: `{ video_id, video_title, vocabulary: [{word, reading, meaning, part_of_speech, jlpt_level, occurrence_count, timestamps}] }`
- **FlashcardDeck**: `{ meta: {video_id, video_title, youtube_url, generated_at, total_cards, jlpt_breakdown}, cards: [{id, front, back, jlpt_level, youtube_link, secondary_links, context_snippet}] }`

## Key Files

| File | Role |
|------|------|
| `skills/flashcard-generator/SKILL.md` | Main pipeline orchestrator — URL validation, MCP invocation, error handling, subagent invocation, flashcard transformation, JSON output |
| `skills/subtitle-downloader/SKILL.md` | Standalone batch subtitle downloader — uses yt-dlp (CLI, not MCP) to download .ja.srt files by search query or direct URL |
| `subagents/vocabulary-extractor/SUBAGENT.md` | Subagent definition — language-agnostic vocabulary extraction from transcript segments, deduplication, stopword filtering, occurrence counting |
| `ui/index.html` | Single-page flashcard viewer shell — loads JSON, renders card deck with flip/filter/sort |
| `ui/app.js` | All UI logic — card rendering, flip animation, navigation (arrow keys + buttons), JLPT filter/sort, progress tracking, YouTube deep links |
| `ui/flashcards.css` | Dark-themed card deck styling — 3D flip animation, CJK fonts, JLPT color coding (green/orange/red for N5/N4/N3) |
| `ui/sample-data/sample-deck.json` | Sample Japanese FlashcardDeck (12 cards: 4 N5, 4 N4, 4 N3) for UI development |
| `ui/sample-data/live-test.json` | Real output from pipeline — loaded by default in `index.html` |

### Specs directory structure

```
specs/001-youtube-flashcard-generator/
├── spec.md              # Feature specification (user stories, requirements, success criteria)
├── plan.md              # Implementation plan (architecture, layer design, project structure)
├── research.md           # Phase 0 research output
├── data-model.md         # Entity schemas and validation rules
├── quickstart.md         # End-to-end validation scenarios
├── tasks.md              # Full task breakdown by user story
├── progress.md           # Current completion status per task
├── workflow.md           # Mermaid flowchart diagram
└── contracts/            # Layer boundary contracts
    ├── transcript-contract.md
    ├── vocabulary-contract.md
    └── flashcard-contract.md
```

## Current Progress

See `specs/001-youtube-flashcard-generator/progress.md` for per-task status. Key state:
- Phase 1 (Setup): done
- Phase 2 (Foundational): sample data not yet created
- US1 (Transcript): SKILL.md complete with all error paths; MCP runtime testing blocked
- US2 (Vocabulary): SUBAGENT.md written; empty-vocabulary handling pending
- US3 (Flashcards): transformation logic pending
- US4 (UI): `app.js` core logic done, CSS and HTML shells pending
- US5 (Filter/Sort): pending
- Phase 8 (Polish): pending

## Conventions

- **No external dependencies**: No package.json, no npm install, no build step. Everything runs inside Claude Code or a browser.
- **Contracts-first**: Layer boundaries are defined by contract files before implementation. Every handoff (MCP → subagent → skill → UI) must match its contract schema.
- **Structured JSON output**: The vocabulary-extractor subagent uses the `schema` option for validated structured output.
- **MCP fallback strategy**: `youtube-transcript` MCP first (explicit `lang: "ja"`), `youtube` MCP as fallback.
- **Error messages**: User-facing error messages are exact strings from `contracts/transcript-contract.md`. Never pass raw MCP or subagent errors directly to the user.
- **Japanese validation**: After transcript extraction, verify text contains CJK characters (hiragana U+3040–U+309F, katakana U+30A0–U+30FF, kanji U+4E00–U+9FFF) before proceeding. If none found, surface the "Japanese subtitles not available" error.
- **Desktop only**: UI targets desktop browsers. Mobile responsiveness is explicitly out of scope.
- **Pipeline idempotent**: Each run is stateless. Data is ephemeral (in-memory or temp files). The only persistent artifact is `output/flashcards-{video_id}.json`.

---
> Source: [PHYOPHYO2397/YT-Japanese-flashcard](https://github.com/PHYOPHYO2397/YT-Japanese-flashcard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-20 -->
