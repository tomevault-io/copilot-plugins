## logogame

> Czech speech recognition game for kids practicing consonant clusters (logopedie).

# Logogame — "Říkej nahlas!"

Czech speech recognition game for kids practicing consonant clusters (logopedie).
Words fall from the sky; the player says them aloud to score points.

## Tech stack

- React 19 + Vite 8, single-page app (no router)
- Bun for package management (`bun install`, not npm)
- Deployed to GitHub Pages via `.github/workflows/deploy.yml` on push to `master`
- Base path: `/logogame/`

## Project structure

```
src/
  main.jsx        — entry point, renders <RickyR />
  RickyR.jsx      — entire game (menu, playing, game-over screens + styles)
  utils.js        — pure functions (normalize, calcStars, pickRandom)
  utils.test.js   — tests for utils (bun test)
index.html        — shell HTML (lang="cs")
vite.config.js    — Vite config with React plugin and base path
ricky-r.jsx       — legacy/backup copy (not used in build)
```

## Commands

```sh
bun install       # install dependencies (also sets up git hooks)
bun run dev       # local dev server
bun run lint      # ESLint — zero warnings allowed
bun test          # run tests
bun run build     # lint + production build → dist/
bun run preview   # preview production build
```

## Quality gates

- **ESLint** runs on every commit (pre-commit hook) and as part of `bun run build` (CI)
- Zero warnings policy (`--max-warnings 0`) — fix warnings, don't accumulate them
- Pre-commit hook is in `.githooks/pre-commit`, auto-configured via `postinstall`
- **CI** (`.github/workflows/ci.yml`) runs lint + build + tests on every PR and push to master
- Pure logic goes in `src/utils.js` with tests in `src/utils.test.js`

## Conventions

- All user-facing text is in Czech — use proper UTF-8 characters (á, č, ř, š, ž, ů…), never Unicode escapes
- Speech recognition uses `cs-CZ` locale
- Word matching normalizes away diacritics (NFD + strip combining marks) so voice input doesn't need exact accents
- Game levels are defined in the `LEVELS` array at the top of `RickyR.jsx`
- Styles (`S`), keyframes, and helper components (`HudItem`, `StatBox`) are defined **before** the main component — never reference a `const` before its declaration
- **Hook ordering in React components**: define all `useCallback` functions before any `useEffect` that references them in its dependency array. Violating this causes a runtime ReferenceError (temporal dead zone).
- No CSS files, no external component libraries

---
> Source: [mojzis/logogame](https://github.com/mojzis/logogame) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-16 -->
