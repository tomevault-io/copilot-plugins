## bizappweb

> Full project context, architecture, deployment, and planning docs live in the

# This repo is part of BizFlow

Full project context, architecture, deployment, and planning docs live in the
sibling **[`biz_app-planning-hub`](../biz_app-planning-hub)** repo — read
[`../biz_app-planning-hub/CLAUDE.md`](../biz_app-planning-hub/CLAUDE.md)
first. That's the source of truth; don't duplicate it here.

## Quick facts (this repo only)

- React 19 + TypeScript + Vite SPA (no SSR)
- Dev: `npm run dev` → `http://localhost:5174`
- Build: `npm run build` → static output in `dist/`
- Talks to the backend (`../biz_app-backend`) over HTTP via `VITE_API_URL`
- Live in production at **https://app.gbicloud.in** (served as static
  files by nginx — see `deploy/README.md` in this repo, and the hub's
  `deploy/production.md`)
- Deploy scripts: `deploy/deploy.sh`, `deploy/deploy-frontend.sh` (this repo)

---
> Source: [itskeerthivasan/bizappweb](https://github.com/itskeerthivasan/bizappweb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-17 -->
