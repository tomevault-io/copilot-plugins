## slidev-ai-template

> This is a [Slidev](https://sli.dev) slide deck. The entry point is `slides.md` —

# CLAUDE.md

This is a [Slidev](https://sli.dev) slide deck. The entry point is `slides.md` —
every `---` line starts a new slide.

## AI tooling in this repo

- **Skill**: `.claude/skills/slidev` (symlinked from `.agents/skills/slidev`, the
  vendored copy of the official `slidevjs/slidev` skill). It documents Slidev's
  markdown syntax, frontmatter, layouts, animations, code highlighting, diagrams,
  and CLI. Read it before making non-trivial slide changes.
- **MCP server**: configured in `.mcp.json`, runs `slidev mcp slides.md` over
  stdio. It exposes structured tools (`slidev-get-slide`, `slidev-insert-slide`,
  `slidev-update-slide`, `slidev-move-slide`, `slidev-remove-slide`, etc.) for
  editing the deck. Prefer these tools over raw text edits for slide-level
  operations — they handle Slidev's `---` separators correctly and hot-reload
  the running dev server. See `.claude/skills/slidev/references/tool-mcp.md`
  for the full tool list.

## Working on this deck

- `pnpm install` once, then `pnpm dev` to preview at `http://localhost:3030`
  with hot reload.
- Keep one slide's content between its own pair of `---` separators.
- Prefer the MCP tools for inserting/reordering/removing slides; use direct
  file edits for authoring prose/content within a slide.
- `pnpm build` produces a static site in `dist/`; the `deploy.yml` workflow
  publishes it to GitHub Pages on push to `main`.

---
> Source: [narze/slidev-ai-template](https://github.com/narze/slidev-ai-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-23 -->
