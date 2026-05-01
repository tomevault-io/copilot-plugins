## cursor-markdown-file-links

> Review markdown in Cursor — path-only file links (no :line in link targets)


# Markdown file links in Cursor

In **Cursor**, `path:line` and `path:start-end` in markdown link targets do not open the file at the line. Prefer:

- `[path/to/file.ts](path/to/file.ts)` — lines 12–34

Detect Cursor when `CURSOR_AGENT`, `CURSOR_TRACE_ID`, or `CURSOR_CLI` is set. Full detail: `.agents/skills/drive-pr-walkthrough/SKILL.md` (linking conventions).

---
> Source: [prisma/prisma-next](https://github.com/prisma/prisma-next) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
