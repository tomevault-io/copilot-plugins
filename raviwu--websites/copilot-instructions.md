## websites

> Hugo monorepo for 4 personal blogs deployed on Cloudflare Pages.

# CLAUDE.md

Hugo monorepo for 4 personal blogs deployed on Cloudflare Pages.

## Structure

- `sites/{tempo,lw-studio,draft-zero,purmuurfuuur}/` — each is an independent Hugo site
- `scripts/sync-content.sh` — syncs content from Obsidian vault to this repo
- Content source of truth is the Obsidian vault, not this repo

## Key commands

- `make sync` — sync all sites from vault
- `make serve-tempo` / `serve-lw` / `serve-pur` / `serve-dz` — local preview
- `make build-all` — build all sites
- `git push` — triggers Cloudflare Pages auto-deploy

## Conventions

- Do not edit post content in this repo directly — edit in Obsidian vault, then sync
- New posts default to `draft: true`
- All posts use frontmatter: title, date, tags, image, draft, blog

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/raviwu) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
