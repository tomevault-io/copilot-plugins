## 03-ui

> Keep the premium studio UI. Never restore the old utility layout.


# UI

The desktop UI is the premium studio in `src/studio/StudioApp.tsx`, with `src/ui/`, `src/views/`, `src/styles/`.

`src/main.tsx` must import `StudioApp`, not the old single-file utility screen.

Never restore the old layout: centered `.page` / `.hero` / `.dropzone` / `.video-list` / brown utility form. That screen is retired.

If `src/App.tsx` is touched, it must re-export `StudioApp`.

---
> Source: [AndreaZero/subber](https://github.com/AndreaZero/subber) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
