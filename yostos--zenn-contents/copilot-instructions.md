## zenn-contents

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Project Name:** zenn-contents
**jrnl Project Tag:** zenn-contents

Repository for managing Zenn articles and books.

- `articles/` — Markdown files for articles
- `books/` — Book directories

## Commands

```bash
npx zenn preview              # Local preview (http://localhost:8000)
npx zenn new:article          # Create new article
npx zenn new:book             # Create new book
```

## References

See the following for details.

- Frontmatter & Emoji selection: `docs/article-frontmatter.md`
- Japanese writing rules & textlint: `docs/writing-rules.md`

## Writing Rules (always applied)

- Body text uses polite form (ですます調), headings/lists use plain form (である調)
- End sentences with "。" (colon "：" is prohibited)
- Do not use level-3 headings (###) without user permission
- Do not use bold (`**`) as a substitute for headings
- Do not repeat the same particle in one sentence

## Linting

- **Pre-commit hook** runs textlint automatically
- Do not run textlint manually from Claude Code (only when explicitly requested by user)
- Adding textlint suppression comments requires user permission (also blocked by Hooks)
- Skipping with `--no-verify` is prohibited (also blocked by Hooks)

## Git Operations

- **Main branch**: `main` (PR required, direct push prohibited; also blocked by Hooks)
- **Branch naming**: `article/<slug>` (articles), `docs/<topic>` (documentation)
- Verify `published: true` before creating a PR

## Commit Message Prefixes

| Prefix | Usage |
|--------|-------|
| `content:` | Add or update articles/books (`articles/`, `books/`) |
| `chore:` | Config, CI, dependencies |
| `docs:` | Documentation (`README.md`, `CLAUDE.md`) |

## Requirements

- Node.js >= 22.0.0 (managed via mise.toml)

---
> Source: [yostos/zenn-contents](https://github.com/yostos/zenn-contents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-17 -->
