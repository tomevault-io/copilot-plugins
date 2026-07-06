## shakespeare-writing-studio

> This repository is designed to be readable by any agent or CLI tool, even when the tool does not support plugin marketplaces yet.

# Agent Entry Point

This repository is designed to be readable by any agent or CLI tool, even when the tool does not support plugin marketplaces yet.

## Primary Skill

Read this first:

- `SKILL.md`

If your agent expects skills under a `skills/` directory, use:

- `plugins/shakespeare-writing-studio/skills/shakespeare-writing-studio/SKILL.md`

Both files define the same workflow.

## What This Agent Should Do

Use Shakespeare Writing Studio when the user asks for:

- Chinese WeChat articles
- English Medium/Substack essays
- research-backed articles
- source-bound writing
- human voice editing
- AI-feel reduction
- cover design
- statistical graphics
- data storytelling
- publish-ready HTML or Markdown packages

## Required Reading Order

1. `SKILL.md`
2. `references/editorial-lessons.md`
3. `references/source-quality.md` when research is needed
4. `references/human-voice.md` before rewriting
5. `references/visual-production.md` before creating images or charts
6. `references/release-gate.md` before final delivery

## Hard Rules

- Do not invent facts, numbers, quotes, or sources.
- Do not invent strategic or product concepts to make the article sound deeper.
- Do not use obvious AI prose scaffolds.
- Do not default to dark visuals or generic tech graphics.
- Do not use literal mountain, flag, rocket, or cheap up-arrow metaphors for abstract market momentum.
- Use top-tier sources only when background search is needed.
- Make the cover and layout improve the article's perceived quality.

## Plugin Paths

Codex/OpenAI-compatible manifest:

- `plugins/shakespeare-writing-studio/.codex-plugin/plugin.json`

Claude-compatible manifest:

- `plugins/shakespeare-writing-studio/.claude-plugin/plugin.json`

Marketplace files:

- `.agents/plugins/marketplace.json`
- `.claude-plugin/marketplace.json`

## Chat-Only Fallback

For DeepSeek, Doubao, Kimi, Qwen, ChatGPT web, or any assistant that cannot install plugins, use:

- `prompts/deepseek-doubao.md`

The user should paste that prompt into the chat first, then paste the topic, draft, notes, links, or source material.

---
> Source: [ZCZHAO-1999/shakespeare-writing-studio](https://github.com/ZCZHAO-1999/shakespeare-writing-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-06 -->
