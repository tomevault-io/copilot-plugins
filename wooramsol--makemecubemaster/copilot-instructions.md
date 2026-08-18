## auto-deploy

> Deploy to GitHub Pages after completing user-facing changes


# Auto-deploy

This repo deploys to GitHub Pages on every push to `main`.

After completing user-facing work:

1. Merge or commit changes on `main` (or merge feature branch into `main`).
2. Run `npm run build` to verify the build passes.
3. `git push origin main` — this triggers `.github/workflows/deploy.yml` automatically.
4. Do **not** wait for the user to ask for deployment.

Live URL: https://wooramsol.github.io/makemecubemaster/

Users may need **Ctrl+Shift+R** after deploy (PWA cache).

---
> Source: [wooramsol/makemecubemaster](https://github.com/wooramsol/makemecubemaster) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-17 -->
