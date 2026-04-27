## linxule-com

> transforms.js           # Image optimization + deep-link definition transforms

# CLAUDE.md - AI Assistant Context

Extended reference docs in `.claude/docs/` (local-only, gitignored). Rules in `.claude/rules/`.

## Project Philosophy

A "book in the algorithmic age" — paper/ink colors, marginalia, spine navigation, bloom-on-hover. One cyan "accident" per page (intentional wrongness). Content lists human + AI authors as arrays. Design system in `.claude/docs/design.md`.

## Repo Structure

The website repo is `xule-site/`, not the parent `personal-website/` directory. If Claude Code opens from the parent, all git commands need `git -C xule-site/`.

## Architecture

- **Eleventy (11ty)** + **Nunjucks** + **Pagefind** — no framework (vanilla CSS/JS)
- **Inline `<style>` per page** — intentional, not debt. Global styles in `main.css`
- Image pipeline details in `.claude/rules/architecture.md`
- Concept propagation and departure infrastructure in `.claude/rules/concept-propagation.md`

### Key Files
```
eleventy.config.js        # Main config (ESM, imports from eleventy/)
middleware.ts             # Vercel Edge Middleware (Accept: text/markdown → .md rewrite)
eleventy/
  collections.js          # Collections (writing, portraits, artifacts, tags, concepts)
  transforms.js           # Image optimization + deep-link definition transforms
src/
  _includes/layouts/      # Page templates (writing, portrait, artifact, concepts, etc.)
  assets/css/main.css     # Global styles
  writing/                # Blog posts    concepts/           # Concept territory page
  making/                 # Portraits + artifacts
  talks/                  # Presentations
```

### Content Types

Writing, Portraits, Artifacts, Thinking, Concepts, Teaching, CV, Talks. Each has its own layout. Frontmatter patterns in `.claude/docs/content-patterns.md`.

## Key Gotchas

Full list in `.claude/docs/gotchas.md`.

1. **Do NOT use `| reverse`** on collections.writing — already newest-first
2. **Footnotes** render as marginalia on desktop, endnotes on mobile (≤1100px)
3. **Text shaping is mandatory** — prompts and contextExcerpts must be arrays with stagger pattern and one accident
4. **Writing images are auto-optimized** — use standard `![](path)`, the transform handles AVIF/WebP conversion
5. **Guard undefined arrays** — `contextExcerpt`, `prompt`, `images` can be undefined
6. **ESM project** — all JS uses `import`/`export default`
7. **`keywords` is the standard** frontmatter field (not `tags`), both feed `tagPages`

## Commands

```bash
bun run start  # Dev server with hot reload
bun run build  # Production build + Pagefind index
```

## Deployment

Vercel, `linxule.com`. `git push` auto-deploys. Details in `.claude/docs/infrastructure.md`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/linxule) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
