## sidgangal-website

> `public/roughwork/` contains a **separate React app** (Aerem Demo) that is served at `/roughwork/`. It is completely isolated from the main Astro site.

# Personal Website (theusefulstack.com)

## Roughwork subdirectory

`public/roughwork/` contains a **separate React app** (Aerem Demo) that is served at `/roughwork/`. It is completely isolated from the main Astro site.

### Rules

- **Never modify files inside `public/roughwork/` directly.** They are build output copied from the Aerem Demo project (`/Users/sidgangal/Code/4. Aerem Demo`).
- **Never include `/roughwork/` in the sitemap, SEO, or any main site navigation.**
- Preserve the `_headers` rule that adds `X-Robots-Tag: noindex` for `/roughwork/*`.
- Preserve the `Disallow: /roughwork/` rule in `robots.txt`.
- Preserve the `_redirects` SPA fallback for `/roughwork/*`.

## Deploying after publishing an article

The site is hosted on Cloudflare Pages, project `sidgangal-website` (serves `theusefulstack.com`). Cloudflare is **not connected to GitHub**, so deploys are manual via Wrangler.

### Standard flow (PR-then-deploy)

```bash
# 1. Merge your PR into main
gh pr merge <PR#> --merge

# 2. Make sure local main is up to date (if main is checked out here)
git checkout main && git pull origin main
# If main is checked out in another worktree (e.g. Conductor), instead run:
#   git fetch origin main
# and build from any branch whose tree matches origin/main.

# 3. Build
npm run build

# 4. Deploy to production
npx wrangler pages deploy dist --project-name=sidgangal-website --branch=main

# 5. Verify
npx wrangler pages deployment list --project-name=sidgangal-website | head -5
curl -sI https://theusefulstack.com/<new-article-slug>/
```

### Notes

- `--branch=main` is what makes this a **production** deploy. Omitting it produces a preview deployment at a `*.pages.dev` URL that does not affect `theusefulstack.com`.
- Wrangler uploads the contents of `dist/`, so always run `npm run build` first — `dist/` is gitignored and can be stale.
- If you see `--commit-dirty` warnings, that's normal for manual deploys from a clean working tree; pass `--commit-dirty=true` to silence them.
- The `public/_headers` and `public/_redirects` files (including the `/roughwork/*` rules) are copied into `dist/` by the build and uploaded with the deploy — no separate step needed.

---
> Source: [sidgangal/sidgangal-website](https://github.com/sidgangal/sidgangal-website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
