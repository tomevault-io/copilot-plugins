## manual-dev-reload

> Dev server does not auto-reload; refresh the game explicitly when a task finishes.


# Manual Vite reload

`npm run dev` has **auto-reload disabled** so mid-task file writes do not restart an
in-progress sandbox / multiplayer session. The open page shows a **stale code** badge
when client-relevant files change; the user can click it to refresh themselves.

When a coding task that changed client (`src/`, `index.html`, Vite-served assets) or
shared sim code is **finished** and the user may be playtesting, you may also trigger
one explicit refresh:

```bash
npm run reload
```

That hits `POST http://127.0.0.1:5173/__reload`. If curl fails (dev server not running),
ignore it — do not start Vite just to reload.

The endpoint sends the opt-in `hard-lines:reload` event, not Vite's global
`full-reload`. The game opts in. `/asset-viewer` deliberately does not:
preserve live workshop state until the user manually refreshes that browser page.

Do **not** reload after every file write. Skip reload for docs-only or other
non-client noise. Opt back into Vite's default auto-reload with
`HARD_LINES_AUTO_RELOAD=1 npm run dev`.

---
> Source: [yegors/hard-lines](https://github.com/yegors/hard-lines) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-27 -->
