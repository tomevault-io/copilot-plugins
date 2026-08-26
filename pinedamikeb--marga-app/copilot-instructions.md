## deploy-handoff

> MARGA production deploy workflow from HANDOFF/MASTERPLAN


# Deploy And Release

- Production repo: `/Volumes/Wotg Drive Mike/GitHub/Marga-App`, branch `main`, staff URL `app.marga.biz`.
- After verified fixes: commit and push to `main` for Netlify deploy unless the user explicitly says not to push in that message.
- Verify live on `app.marga.biz` with hard refresh (service worker). Sync to `Marga-App-staging` only after production is confirmed.
- Bump `service-worker.js` cache name and script `?v=` versions on deploy.

---
> Source: [PinedaMikeB/marga-app](https://github.com/PinedaMikeB/marga-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-26 -->
