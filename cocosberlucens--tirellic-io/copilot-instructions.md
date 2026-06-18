## tirellic-io

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**tirellic.io** is Corrado's personal portfolio and knowledge-sharing site — a digital garden and living CV. It publishes selected ML learning notes and healthcare analytics project writeups from the Obsidian vault **psychic-patterns**.

## Technical Stack

- **Static site generator**: Quartz v4 (supports `[[wikilinks]]`, backlinks, graph view, KaTeX)
- **Hosting**: Cloudflare Pages (auto-builds on push to `v4` branch)
- **Domain**: `tirellic.io` — Cloudflare Registrar (DNSSEC enabled, 2FA configured)
- **Theme**: Kinetic Whisper — warm off-white, Space Grotesk headings, Source Serif 4 body, red/blue accent system
- **Content source**: `psychic-patterns/ml-code-companion/notes/` only (gated by `publish: true` frontmatter)

## Commands

| Command | Purpose |
|---------|---------|
| `./scripts/publish.sh` | Sync `publish: true` notes from vault, commit, push |
| `./scripts/publish.sh --preview` | Sync + local preview server |
| `./scripts/publish.sh --sync-only` | Sync without commit/push (used by tests) |
| `./scripts/publish-test.sh` | Run 8 integration tests (17 assertions) |
| `npx quartz build` | Build site to `public/` |
| `npx quartz build --serve` | Build + local dev server |

## Content Management

Use the `/obsidian` skill to prepare notes before publishing. Required frontmatter properties:

| Property | Example | Notes |
|----------|---------|-------|
| `topic` | `"conditional probability and Bayes' rule"` | Becomes `title:` on publish (publish script rewrites) |
| `publish` | `false` | Flip to `true` when ready to go live |
| `created` | `2026-03-28` | Date type |
| `updated` | `2026-03-28` | Date type |
| `tags` | `[probability, bayes]` | List type — drives tag pages on the site |

```bash
OBS="/Applications/Obsidian.app/Contents/MacOS/Obsidian"
$OBS property:set name="topic" value="sum and product rule" file="Untitled" vault="psychic-patterns"
$OBS property:set name="publish" value="false" file="Untitled" vault="psychic-patterns"
$OBS property:set name="created" value="2026-03-28" type=date file="Untitled" vault="psychic-patterns"
$OBS property:set name="tags" value="[probability, combinatorics]" type=list file="Untitled" vault="psychic-patterns"
```

## Editorial Review

Before publishing, review notes with `/editorial-review <note-name>`:

1. **Phase 1**: Classifies note type (atomic, conceptual, pattern, derivation, reference)
2. **Phase 2**: Mechanical checks (subagents) + conversational review (correctness, flow, wikilink suggestions)

The review is grounded in the ml-code-companion curriculum. See `docs/superpowers/specs/2026-03-28-editorial-review-skill-design.md` for the full design.

## Content Pipeline

1. Prepare note in Obsidian — use `/obsidian` skill to set frontmatter properties
2. Review with `/editorial-review <note-name>` — verify quality before publishing
3. Set `publish: true` when ready
4. Run `./scripts/publish.sh` — copies notes to `content/`, rewrites `topic:` → `title:`, commits, pushes
5. Cloudflare Pages auto-builds and deploys

The publish script copies only **referenced** attachments (parses `![[file]]` and `![](attachments/file)` from published notes). Orphaned images from paste-then-delete in Obsidian are excluded.

Content from `refactored-umbrella` (work projects) is handled manually — translated, sanitized, placed in `psychic-patterns/ml-code-companion/notes/` with `publish: true`.

## Key Files

- `quartz.config.ts` — site settings, Kinetic Whisper palette, ExplicitPublish filter
- `quartz/styles/custom.scss` — custom CSS (nav gradient, math blocks, tags, backlinks)
- `content/index.md` — landing page
- `scripts/publish.sh` — vault-to-content sync script

## LaTeX Compatibility

Quartz KaTeX is stricter than Obsidian. Notes must follow these rules:
- `$$` delimiters on their own lines with blank lines above and below
- No `\begin{equation}` inside `$$` — use `\tag{N}` directly
- Inline math with single `$` works normally

## Design Spec

Full design documentation: `docs/superpowers/specs/2026-03-26-quartz-site-design.md`

## Related Repositories

- **psychic-patterns**: Obsidian knowledge vault (source notes)
- **ml-code-companion**: ML learning curriculum that produces the notes
- **refactored-umbrella**: Corrado's work vault (DB/BI at the LHA) — occasional content source

---
> Source: [cocosberlucens/tirellic.io](https://github.com/cocosberlucens/tirellic.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
