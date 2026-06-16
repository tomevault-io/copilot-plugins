## llm-knowledge-bases

> - LLM Knowledge Base project -- a system of skills for maintaining personal knowledge bases as Obsidian wikis.

# CLAUDE.md -- Project Instructions

## Project Identity

- LLM Knowledge Base project -- a system of skills for maintaining personal knowledge bases as Obsidian wikis.
- The wiki is the domain of the LLM. Never manually edit wiki articles outside of the kb skill workflows.

## Configuration

- Always read `kb.yaml` at the project root before operating. It contains paths, output format preferences, and Obsidian config.

## Skills

- `kb-init` -- One-time setup to bootstrap a new knowledge base.
- `kb` -- Main operating skill with four workflows: compile, query, lint, evolve.
- `research` -- Initial research outline generation.
- `research-deep` -- Deep research via independent agents per outline item.
- `research-add-fields` -- Add field definitions to an existing research outline.
- `research-add-items` -- Add items (research targets) to an existing research outline.
- `research-report` -- Summarize deep research results into a markdown report.

## Formatting Rules (Obsidian-Compatible)

- `[[wikilinks]]` for all internal links (never markdown-style links).
- `[[wikilinks|display text]]` when the display name differs from the target.
- YAML frontmatter on every article with: title, aliases, tags, sources.
- `![[image.png]]` for image embeds.

## Model Strategy

- Opus orchestrates all workflows.
- Haiku for mechanical scanning tasks.
- Sonnet for article writing and research subagents.
- Always set `model` parameter when dispatching subagents.

---
> Source: [rvk7895/llm-knowledge-bases](https://github.com/rvk7895/llm-knowledge-bases) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
