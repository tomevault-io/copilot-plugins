## site

> After site changes, commit and push so Vercel deploys


# Git: commit and push for Vercel

This repo deploys to Vercel from GitHub. After making substantive changes the user cares to see live:

1. `git add` the relevant files (never commit `__pycache__` or `.pyc`).
2. `git commit` with a clear message.
3. **`git push origin main`** (or the current branch that Vercel tracks) — **always**, not only commit. A local-only commit does not update production.

If push fails (auth/network), say so and what to fix; do not assume commit alone is enough.

**Catalog edits:** Running `admin_realtime.py` locally commits and pushes `products.json` automatically (~4s debounce after each save) unless `ADMIN_AUTO_GIT_PUSH=0`. Manual git is still needed for changes to other files (e.g. `js/app.js`).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/xtrrrxx-create) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
