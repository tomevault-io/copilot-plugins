## astrology

> After code changes, commit, push to GitHub, and publish to Vercel


# Ship changes (Git + Vercel)

When you finish a **substantive change** (features, fixes, config that affects production):

1. Run **`npm run build`** if you touched app code, routes, or dependencies; fix failures before shipping.
2. **`git add`** the relevant files, **`git commit`** with a clear message, **`git push origin main`** to [github.com/sreenuti/Astrology](https://github.com/sreenuti/Astrology).
3. **Publish to Vercel**: run **`npm run vercel:prod`** (or `npx vercel deploy --prod --yes` from the repo root) so production updates even if Git webhooks are slow. If the project is Git-linked, this may queue a second build—that is acceptable per project owner preference.

Do **not** commit secrets (`.env.local`, API keys in source). Use `.env.example` for placeholders only.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sreenuti) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
