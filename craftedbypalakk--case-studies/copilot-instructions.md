## vercel-production-deploy

> When user says deploy — use npm run deploy; production URL is only https://palakblooms.vercel.app


# Vercel production deploy

When the user asks to **deploy** this project (without asking for a preview):

1. Run **`npm run deploy`** from the repo root. It runs `vercel deploy --prod --yes` against the linked Vercel project and prints the canonical URL.

2. Treat **`https://palakblooms.vercel.app`** as the **only** user-facing production URL. Vercel still prints a unique deployment hostname (e.g. `case-studies-….vercel.app`); do **not** present that as the main link unless debugging that deployment.

3. Do **not** run `vercel` without `--prod` for a normal "deploy" — that creates a preview, not the production domain.

4. **`git push origin main`** also updates production when Git is connected; the live site is still **palakblooms.vercel.app**.

---
> Source: [craftedbypalakk/case-studies](https://github.com/craftedbypalakk/case-studies) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-27 -->
