## cotton-market-intelligence

> - **NEVER push feature branches to trigger Vercel previews.** Feature branch previews are disabled in `vercel.json`.

# CLAUDE.md — CMI Project Rules

## Deployment rules (CRITICAL)

- **NEVER push feature branches to trigger Vercel previews.** Feature branch previews are disabled in `vercel.json`.
- Dev deployment: merge into `develop` → auto-deploys to `cmi-notebooks-dev.vercel.app`
- Prod deployment: merge `develop` → `main` → auto-deploys to `cmi-notebooks.vercel.app`
- The only branches that trigger Vercel deployments are `main` and `develop`.

## Git workflow

- Branch from `develop`, not `main`
- Branch naming: `feature/<issue-id>-<slug>`, `fix/<issue-id>-<slug>`
- PR flow: `feature/*` → `develop` → validate on dev → `develop` → `main`
- Commit and push to `develop` for dev testing without asking user permission
- Always ask before merging to `main`

## Dev URLs

- **Dev**: https://cmi-notebooks-dev.vercel.app
- **Prod**: https://cmi-notebooks.vercel.app

## Tech stack

- Next.js 16 (App Router), React 19, TypeScript 5, Tailwind CSS 4, Recharts, Zod, Vitest
- Stateless (no database), localStorage for scenario persistence
- Strategy engine: HF AI → OpenAI fallback → heuristic fallback
- Rate limiting: in-memory per-IP/per-user buckets

## Testing

- Run `npm test` before every commit
- Run `npm run build` to verify type safety
- All tests must pass before pushing

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zayansalman) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
