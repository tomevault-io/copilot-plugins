## focuslock-chrome-extension

> Read this before doing any work in this repo. It's the operating manual for how to work on

# CLAUDE.md — FocusLock Project Instructions

Read this before doing any work in this repo. It's the operating manual for how to work on
FocusLock specifically — not a description of the product (see `FocusLock-Development-Spec.md`,
delivered separately, for that).

---

## 0. Current state of the code — read this first

This is the **v2 codebase**, built fresh per the architecture below — it does not extend the old
v1 files (mandatory password, single Website Blocker list, etc.), which are documented separately
in `FocusLock-Project-Handoff.md` and are no longer what this code does.

**What's actually built right now:** Block Sites only — category toggles (Social Media, Adult Sites,
Shopping Sites, Gambling, News), a custom site list with add/edit/delete/toggle, an optional redirect
URL, and the background service worker that actually enforces it (redirects a matching navigation to
the blocked page or the configured redirect). See `README.md` for setup/run instructions and the full
status table.

**Everything else is a placeholder.** The dashboard shell (sidebar with all 9 pages) exists and is
wired up, but clicking any page other than Block Sites shows a "not built yet" message — except Focus
Mode and Insights, which show the intended "Coming Soon" state per the design. Do not assume any
other page's logic exists until you've checked.

---

## 1. Scope discipline (the most important rule)

When told to work on a specific page — e.g. "work on Reel Blocks" — touch **only** that feature's
files inside `src/features/<name>/`. Do not also refactor, "improve," or touch other pages in the
same pass, even if you notice something that looks related or improvable elsewhere. If a change
genuinely requires touching another feature, stop and say so before doing it, rather than doing it
silently.

The folder boundary enforces this: a file inside `src/features/reel-blocks/` must never import from
`src/features/block-sites/` or any other feature folder — only from `src/core/`. If two features
need the same logic, that logic belongs in `src/core/`, not copy-pasted or cross-imported between
features. `src/options/Sidebar.tsx` and `src/options/App.tsx` are the shell — they import each
feature's page component to render it, but features never import the shell.

Every source file starts with a one-line label comment identifying which feature it belongs to, e.g.
`// FEATURE: Block Sites — page UI`. Follow that convention in new files.

---

## 2. What to build, and how — reference table

Full detail lives in `FocusLock-Development-Spec.md`. Quick reference:

| Page | Mechanism | Status |
|---|---|---|
| Block Sites | Hard redirect via `background/index.ts` + `webNavigation` — categories + custom list | ✅ Built |
| Reel Blocks | Same mechanism as Block Sites, presets scoped to reel-specific paths | ✅ Built |
| Feed Blocks | **Content script**, not a redirect — hides DOM elements per site | ✅ Built |
| Limiter | Hard redirect once a daily time budget runs out | ✅ Built |
| Focus Mode | Coming Soon placeholder | Placeholder (matches design) |
| Insights | Coming Soon placeholder | Placeholder (matches design) |

Password Protection is not on the roadmap right now — no feature folder, no nav entry, not planned.

**Do not conflate the two mechanisms.** Hard-redirect features only need `background/index.ts` +
`webNavigation` — no content script. Feed Blocks needs a content script instead (element-hiding),
and shouldn't be built as a redirect.

---

## 3. Stack & conventions (as actually set up in this repo)

- **TypeScript**, strict mode on. `npm run typecheck` before considering anything done.
- **Preact** for all UI — `.tsx` files, `preact/hooks` for state. No React, no other framework.
- **Vite + @crxjs/vite-plugin** for the build. `npm run build` outputs a loadable `dist/`.
- **Vitest** for unit tests on pure logic — see `src/core/rule-engine.test.ts` for the pattern. Don't
  try to unit-test content-script DOM behavior against real sites; that needs manual QA.
- Plain CSS per component/page (no Tailwind, no CSS-in-JS) — see `src/core/ui/*.css` for shared
  component styles and `src/features/*/*.css` for page-specific styles.
- Design tokens live in `src/core/ui/tokens.css` as CSS custom properties (`--fl-*`) — pulled from
  the Figma file. Extend them, don't hardcode new colors inline.
- All storage reads/writes go through `src/core/storage.ts` — don't invent ad hoc `chrome.storage`
  keys inside a feature file. Add a new top-level key to the `FocusLockData` interface when a new
  feature needs persisted state.
- Shared UI primitives (`Toggle`, `Button`, `Icon`) live in `src/core/ui/` — reuse them, don't build
  a one-off toggle/button inside a feature.

---

## 4. Working style for whoever (human or Claude) picks this up

- **Confirm scope before implementing**, especially for new UI or new mechanics — this person prefers
  a documented plan over jumping straight to code.
- **Be honest about platform limitations.** Don't imply the extension can do something Chrome's
  platform doesn't actually allow.
- **Keep it simple, not gold-plated.** Avoid unnecessary polish or complexity beyond what's specified.
- Communication should be clear and concrete rather than jargon-heavy — English is not the project
  owner's first language.
- **Icon fidelity is explicitly deprioritized for now** — simple placeholder icons are fine until told
  otherwise (see `src/core/ui/Icon.tsx`).

---

## 5. Reference docs (delivered separately, not in this folder)

- `FocusLock-Project-Handoff.md` — original v1 build documentation, historical reference only.
- `FocusLock-Development-Spec.md` — v2 scope, page-by-page spec, target architecture. This file
  enforces *how* to work against that spec.
- `FocusLock-Page-List-v2.md` / `v3.md` — the reasoning behind the Feed Blocks addition, if more
  context is ever needed on *why*.

---
> Source: [mahmodulmakes/focuslock-chrome-extension](https://github.com/mahmodulmakes/focuslock-chrome-extension) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
