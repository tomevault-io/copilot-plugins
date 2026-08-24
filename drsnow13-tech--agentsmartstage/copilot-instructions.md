## agentsmartstage

> Virtual staging app, live at https://www.smartstageagent.com. The app lives in the `stagesmart.ai/` subdirectory (Vite + React SPA, Vercel serverless functions in `stagesmart.ai/api/`, Neon Postgres via `DATABASE_URL`). Vercel project name: `stagesmart.ai`.

# StageSmart AI (SSA)

Virtual staging app, live at https://www.smartstageagent.com. The app lives in the `stagesmart.ai/` subdirectory (Vite + React SPA, Vercel serverless functions in `stagesmart.ai/api/`, Neon Postgres via `DATABASE_URL`). Vercel project name: `stagesmart.ai`.

Windows: in Git Bash use forward-slash paths (~/repo), no PowerShell cmdlets in bash, use the repo dir as cwd (searches from C:\Users\Darren time out).

## Deploy — NOT automatic
- `git push` does NOT deploy production. Deploy with:
  `cd ~/agentsmartstage/stagesmart.ai && npx vercel --prod`
  (the `.vercel/` link lives inside `stagesmart.ai/`, not the repo root)
- After deploying, VERIFY LIVE: curl https://www.smartstageagent.com and exercise `/api/stage` (endpoint is `api/stage.ts`). Local test results can mislead — the deployed bundle may be older than your working tree. Always verify against production.

## Gemini key rules
- Server-side only (key was removed from client bundle — keep it that way).
- Env var: `GEMINI_API_KEYS` (comma-separated, multiple keys), falls back to `GEMINI_API_KEY`. See top of `api/stage.ts`.
- Keys must be UNRESTRICTED — no HTTP-referrer restriction (referrer-restricted keys fail from serverless).
- The `agentsmartstage` GCP project is SUSPENDED. Keys must come from a healthy GCP project.
- Vercel sensitive env vars CANNOT be read back with `vercel env pull` — only a live endpoint test proves they are set.

## Promo codes
Promo codes (e.g. DFWAGENT2026) are rows in the `promo_codes` DB table (see `api/admin.ts`, `api/redeem-promo.ts`, `api/checkout.ts`). A new code needs ZERO code changes — insert a row via the admin API or SQL.

## Other API endpoints
`api/`: analyze, checkout, webhook (Stripe), send-otp / verify-otp, user, deduct-credit, download, redeem-promo, report, admin.

---
> Source: [drsnow13-tech/agentsmartstage](https://github.com/drsnow13-tech/agentsmartstage) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
