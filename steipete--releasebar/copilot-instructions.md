## releasebar

> - Canonical prod: Cloudflare Worker + Worker Assets from `wrangler.toml`.

Work style: terse.

## Deploy

- Canonical prod: Cloudflare Worker + Worker Assets from `wrangler.toml`.
- `release.bar` is not GitHub Pages. Do not trust Pages deploys for prod.
- Push to `main` runs `.github/workflows/deploy.yml`, then `npm exec --yes --package wrangler -- wrangler deploy`.
- Required GitHub secret: `CLOUDFLARE_API_TOKEN`.
- Post-deploy smoke compares live JS/CSS hashes against local `dist/index.html`, then retries live asset fetches while Cloudflare propagates.
- Local prod deploy: `npx wrangler deploy`.
- Static CI/proof: `npm run check:static`.

---
> Source: [steipete/ReleaseBar](https://github.com/steipete/ReleaseBar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-17 -->
