## learn-harness-engineering

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Learn Harness Engineering is a project-based course on building reliable coding environments for AI agents. The repo contains a VitePress documentation site plus hands-on project code.

## Commands

```sh
# Documentation site
npm install
npm run docs:dev        # Dev server with hot reload (VitePress)
npm run docs:build      # Production build
npm run docs:preview    # Preview built site

# Run lecture code examples
npx tsx docs/lectures/<lecture-dir>/code/<file>.ts

# Project Electron apps (from each project directory)
cd projects/project-NN/starter  # or solution/
npm install
npm run dev              # Build + launch Electron (via scripts/dev.js)
npm run check            # Type-check both tsconfig.json and tsconfig.node.json
npm run test             # Vitest run (single run)
npm run test:watch       # Vitest watch mode
```

## Repository Structure

- `docs/` — VitePress documentation site (lectures, projects, resources)
- `docs/.vitepress/config.mts` — Nav/sidebar config for both EN and ZH locales
- `docs/lectures/` — 12 lectures, each with `index.md` + `code/` examples
- `docs/projects/` — 6 project descriptions
- `docs/<lang>/resources/` — localized templates, references, OpenAI advanced pack
- `projects/shared/` — Shared Electron + TypeScript + React foundation
- `projects/project-NN/` — Per-project `starter/` and `solution/` directories

## Architecture

The course revolves around an Electron knowledge-base desktop app that evolves across 6 projects:
- **Main process** (`src/main/`): Window management, IPC handlers, service initialization
- **Preload** (`src/preload/`): contextBridge exposing typed API to renderer
- **Renderer** (`src/renderer/`): React UI with document list, Q&A panel, status bar
- **Services** (`src/services/`): DocumentService, IndexingService, QaService, PersistenceService
- **Shared types** (`src/shared/types.ts`): Cross-boundary interfaces and IPC channel constants

Each project's starter/solution is a complete copy of the Electron app at that evolutionary stage. P(N+1) starter is derived from P(N) solution. The shared foundation is in `projects/shared/`.

## Key Patterns

- IPC channels defined as constants in `src/shared/types.ts` (IPC_CHANNELS) — single source of truth
- All data stored locally as JSON/text files (no database)
- Mock Q&A returns structured answers with citations (no real LLM API)
- Harness files in project roots: AGENTS.md, CLAUDE.md, feature_list.json, init.sh, claude-progress.md
- Progressive disclosure: short AGENTS.md entrypoint linking to focused docs
- Each project has two tsconfigs: `tsconfig.json` (renderer) and `tsconfig.node.json` (main/preload)

## Multilingual Content

Course documentation is organized by locale under `docs/<lang>/`. Keep English as the structural source of truth, keep localized directories in sync, and preserve runnable code examples across languages.

### EN ↔ zh-TW sync check

Whenever `README.md` or anything under `docs/en/**` changes, verify the paired zh-Hant files still match section-for-section (translated content, not identical text). In scope:

- `README.md` ↔ `docs-readme/zh-TW/README.md`
- `projects/project-0N/README.md` ↔ `projects/project-0N/README-ZH-TW.md` (all 6 projects)
- `docs/en/**` ↔ `docs/zh-TW/**`

Compare heading structure with a code-fence-aware extractor (plain `grep -n '^#'` false-positives on shell comments inside ```sh blocks):

```sh
extract_headings() { awk '/^```/{incode=!incode; next} incode{next} /^#/{print}' "$1"; }
diff <(extract_headings <en-file> | sed 's/^#* //') <(extract_headings <zh-TW-file> | sed 's/^#* //')
```

A pass means both sides have the same heading count and the same `#`/`##`/`###` nesting order (content differs by language, structure should not). If they diverge, update the zh-TW side before considering the change done.

## Slides Pack (cached index)

`slides-pack/` holds the consolidated PPTX bundle of all 18 zh-TW decks (12 lectures + 6 projects). Source `.md` lives next to each lecture / project under `docs/zh-TW/lectures/<L>/slides.md` and `docs/zh-TW/projects/<P>/slides.md` (Marp Markdown).

- **`slides-pack/snapshot.json`** — single source of truth for the pack. Contains generation timestamp, deck structure conventions, homework conventions, style rules, regenerate command, and per-deck records (`id`, `kind`, `slug`, `title_zh`, `source_md`, `pptx`, `slide_count`, `size_bytes`, `sha256`). **Read this first** when you need any slide metadata — never re-derive it.
- **`slides-pack/build-snapshot.sh`** — rebuilds `snapshot.json` from current source `.md` and PPTX files. Idempotent.
- **Naming**: `L01..L12-<slug>.pptx` for lectures, `P01..P06-<slug>.pptx` for projects.
- **Regenerate PPTX** (run after editing any `slides.md`):
  ```sh
  find docs/zh-TW \( -path "*/lectures/*" -o -path "*/projects/*" \) -name slides.md ! -path "*/code/*" \
    | while read f; do npx -y @marp-team/marp-cli --allow-local-files --pptx "$f" < /dev/null; done
  # then refresh the pack and snapshot:
  for src in docs/zh-TW/{lectures/lecture,projects/project}-*/slides.pptx; do
    dir=$(basename $(dirname "$src"))
    new=$(echo "$dir" | sed -E 's/^lecture-0?([0-9]+)-/L\1-/; s/^project-0?([0-9]+)-/P\1-/; s/^L([0-9])-/L0\1-/; s/^P([0-9])-/P0\1-/')
    cp "$src" "slides-pack/${new}.pptx"
  done
  bash slides-pack/build-snapshot.sh
  ```
- **Deck conventions** (lifted from `snapshot.json` — do NOT re-design): 12–17 slides; cover → 30-sec TL;DR → dual-audience scenarios (vibe coder + engineer) → core concept + metaphor → official-source data → failure modes → step-by-step → team SOP → agent self-improvement loop → tools/templates → recap → **homework (4 parts: 🪞 reflection / 🛠 hands-on / 🤝 pair-with-agent / 🏢 team SOP)** → further reading.
- **Marp style settings**: see `style:` block at top of any `slides.md` — use the same block for new decks (PingFang TC / Microsoft JhengHei, `#D95C41` headings, full-width Chinese punctuation, 「」 quotes, half-width space between Chinese and English/numbers, `你` not `您`).

---
> Source: [clonn/learn-harness-engineering](https://github.com/clonn/learn-harness-engineering) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
