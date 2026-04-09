## boggart

> When deploying to Railway (`railway up`), always commit and push changes to the git repository too:

# Boggart - Claude Code Instructions

## Deployment

When deploying to Railway (`railway up`), always commit and push changes to the git repository too:

```bash
git add -A
git commit -m "describe changes"
git push origin main
railway up
```

Never deploy without also updating the GitHub repo — this project was previously lost due to not having the source in version control.

## Project Structure

- Monorepo using npm workspaces (`packages/*`)
- `packages/shared` — shared types, must be built/typechecked first
- `packages/server` — Express backend, entry point at `src/index.ts`
- `packages/web` — React + Vite frontend

## Key Services

- **LLM**: Anthropic Claude (Sonnet 4.5 for JSON generation, Haiku 4.5 for batch condensation)
- **Image Generation**: Google Nano Banana 2 (`gemini-3.1-flash-image-preview`)
- **Auth**: Google OAuth 2.0
- **Database**: SQLite via better-sqlite3 (defaults to `./data/boggart.db`)
- **Storage**: Cloudflare R2 (optional, falls back to data URIs)

## Environment

- Environment variables are in `.env` at the project root (never commit this)
- Railway variables are managed via `railway variables`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/willgdjones)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/willgdjones)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
