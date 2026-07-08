## ai-context-hierarchy

> > This file is itself the pattern this repository documents. Treat it as a working example of a Level 1 project context file.

# ai-context-hierarchy — CLAUDE.md (Level 1)

> This file is itself the pattern this repository documents. Treat it as a working example of a Level 1 project context file.

## Status: ACTIVE
Public template repository — three-level context hierarchy for AI coding agents. Featured in the [Graphify v5.0 roadmap](https://github.com/safishamsi/graphify/issues/425).

## Architecture
- `README.md` — main entry, problem/solution/test results, multi-platform support table
- `templates/level-0.md.template` — global project map skeleton (goes in `~/.claude/CLAUDE.md` or equivalent)
- `templates/level-1.md.template` — per-project context skeleton (goes in each project root)
- `templates/desktop-prompt.md` — Claude Desktop system prompt for hierarchy-aware behaviour
- `scripts/parse-sessions.py` — Claude Code JSONL → markdown with YAML frontmatter (date, project, topics, files_touched). Handles Claude Code's `msg.message.content` nesting and mixed string/list-of-blocks content
- `scripts/parse-desktop-export.py` — Claude Desktop export → markdown
- `scripts/vps-sync.md` — slash-command template for syncing remote production code to a local mirror
- `platforms/*.md` — per-platform setup guides (Claude Code, Cursor, Codex, Gemini CLI, Claude Desktop)
- `examples/multi-project-setup.md` — anonymised real-world Level 0 example (15 projects across 4 VPS)
- `article/draft.md` — companion article draft (published versions linked from README)

## CRITICAL RULES — when editing this repo
- **NEVER** introduce links from `README.md` to files that do not exist in the tree. The CI `validate` workflow fails on broken internal links.
- **ALWAYS** update both `README.md` and `README.ru.md` when changing customer-facing surface area. They are mirrors.
- **ALWAYS** update `CHANGELOG.md` for any change that affects users (templates, scripts, platform support).
- Test results (T1–T4) are anchored to specific scenarios — do not renumber without updating both README locales.

## Commands
- `python scripts/parse-sessions.py` — parse `~/.claude/projects/*.jsonl` into `~/conversations/claude-code/*.md`
- `python scripts/parse-desktop-export.py <export.json>` — parse Claude Desktop export JSON
- (no build step — this is a template repository)

## Key Patterns
1. **Hierarchical loading**: agent reads Level 0 (~2 KB, always) → Level 1 (~5 KB, on-demand) → Level 2 (source, only when needed). Graphify GRAPH_REPORT.md sits between Level 1 and Level 2.
2. **YAML frontmatter convention**: every parsed conversation file starts with `--- date / project / topics / files_touched ---`. This format maps 1:1 to Graphify graph nodes (per [issue #425](https://github.com/safishamsi/graphify/issues/425)).
3. **Cross-platform parity**: every platform listed in the support table has a corresponding `platforms/*.md` file. Adding a platform means adding both a row in the table and a setup file.

## External validation
- [Graphify v5.0 roadmap](https://github.com/safishamsi/graphify/issues/425) — Sash Hamsi (Graphify author) confirmed all four enhancement suggestions are on the v5.0 roadmap; `conversation ingestion` maps directly to scripts in this repo.
- [Habr (RU)](https://habr.com/ru/articles/1024878/), [dev.to (EN)](https://dev.to/creatman/the-context-problem-nobody-talks-about-why-ai-coding-agents-waste-80-of-tokens-on-files-they-mp1) — companion articles.

## Recent Changes
See [CHANGELOG.md](CHANGELOG.md).

---
> Source: [CreatmanCEO/ai-context-hierarchy](https://github.com/CreatmanCEO/ai-context-hierarchy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
