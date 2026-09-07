## landing-page

> <!-- BEGIN:nextjs-agent-rules -->

<!-- BEGIN:nextjs-agent-rules -->

# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` (resolved from this file's directory; in monorepos the `next` package may not be visible from the repo root) before writing any code. Heed deprecation notices.

This block is written and re-added by `next dev` — verify at `node_modules/next/dist/server/lib/generate-agent-files.js`. Removing it from a diff only re-creates the uncommitted change; committing it with your work keeps the tree clean.

<!-- END:nextjs-agent-rules -->

## Responsive verification

After making any frontend/UI change in this repo (anything under `src/app`,
`src/components`, or `src/app/globals.css`), invoke the `mobile-view` skill
(`.claude/skills/mobile-view/SKILL.md`) before considering the change done.
It audits the changed files for mobile/tablet/desktop responsiveness against
this repo's Tailwind v4 breakpoint conventions and reports pass/fail per
component.

## Scrolling

This app's real scroll container is `AppScrollArea`
(`src/components/app-scroll-area.tsx`), not the browser window — `<html>`/`<body>`
are pinned with `overflow-hidden`. Any new component that needs internal
scrolling (dropdown, table, modal body, long list) must use `ScrollArea`/
`ScrollBar` from `src/components/ui/scroll-area.tsx`, not raw `overflow-auto`.
Code that needs the app's real scroll position, or that needs to lock/unlock
app scrolling, must go through `useScrollViewport()` from
`src/components/scroll-viewport-context.tsx` — never read/write
`window.scrollY` or `document.body.style.overflow` directly.

---
> Source: [desmark-twintra/Landing-page](https://github.com/desmark-twintra/Landing-page) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-07 -->
