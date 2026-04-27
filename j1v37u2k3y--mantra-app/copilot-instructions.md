## mantra-app

> A small React + Express app that displays a random mantra from `src/data/mantras.json`. Single-purpose,

# Mantra App — Agent Instructions

A small React + Express app that displays a random mantra from `src/data/mantras.json`. Single-purpose,
typography-forward, minimal UI. Deploys both as a full-stack Express app and as a static site on GitHub Pages.

## Architecture

- **`server.js`** — Express server on port `3174`. Loads `src/data/mantras.json` once at startup, exposes
  `GET /api/mantra` and `GET /api/mantra/:index`, and serves the Vite `dist/` build in production. Exports `app` so
  tests can hit it with supertest; only calls `.listen()` when run directly.
- **`src/App.tsx`** — Single-component React UI. Tries `/api/mantra` first, falls back to the bundled
  `src/data/mantras.json` when the API is unreachable (this is what makes the GitHub Pages build work without a
  backend).
- **`src/lib/parseMantra.ts`** — Pure parser extracted so it's unit-testable. `App.tsx` imports from here.
- **`src/data/mantras.json`** — Flat JSON array of strings. Single source of truth — imported by both the client
  bundle and `server.js`.
- **`vite.config.ts`** — Dev server on port `5174` with `/api` proxied to `http://localhost:3174`. Reads
  `VITE_BASE` from the environment so the GitHub Pages workflow can build with `/mantra-app/` as the base.
- **`tests/`** — Vitest suite. `parseMantra.test.ts` covers the parser, `server.test.ts` hits the Express app via
  supertest.

## Mantra Format

`parseMantra()` in `src/lib/parseMantra.ts` recognizes three formats:

1. `"Plain string"` — rendered as title only
2. `"Title — subtitle"` (em dash or en dash) — split into two lines
3. `"**Bold title** — subtitle"` — markdown-style bold title with subtitle (also accepts `-` as separator)

When adding new mantras, prefer format 2 or 3 so the UI has both a primary headline and a secondary line. Keep titles
short (under ~50 chars); subtitles can be longer. A bare hyphen inside a plain string (e.g. `co-located services`)
does NOT split — you need an em/en dash, or bold markers, to get a subtitle.

## Running

```bash
npm install
npm run dev        # Express (3174) + Vite (5174) in parallel via concurrently
npm run build      # Vite build → dist/
npm run preview    # Express serves dist/ + API on port 3174
npm run typecheck  # tsc --noEmit
npm test           # vitest run (parser + server tests)
npm run test:watch # vitest in watch mode
npm run ci         # typecheck + test + build, as CI runs it
```

## Target Resolutions

- Desktop/tablet: normal flow with large Playfair headline.
- **480x320 embedded display** — the layout is compact-first. Title font-size uses `clamp(1.1rem, 5vw, 3rem)`,
  subtitle uses `clamp(0.75rem, 2.6vw, 1.25rem)`, gaps shrink at `sm`, and the spacebar hint is hidden on small
  screens so the control row fits. If you edit `App.tsx`, verify the layout still fits inside a 480x320 viewport
  before shipping.

## Code Conventions

- **TypeScript strict mode** — no `any`, `resolveJsonModule` enabled for JSON imports.
- **React 19** with hooks, functional components only.
- **Tailwind** for styling — no CSS modules or styled-components; inline `style` only for keyframe animations.
- **ESM only** — `"type": "module"` in package.json; use `import` syntax everywhere including `server.js`.
- Keep `App.tsx` as the single UI component. If the app grows, split into `components/` — but don't over-abstract
  for a one-screen experience.

## Making Changes

### Adding a mantra

Edit `src/data/mantras.json`. In dev, restart the server so it picks up the change (the file is read once at
startup in `server.js`). The client also bundles this file at build time for the static fallback — run
`npm run build` to refresh that.

### Changing the mantra parser

`parseMantra()` lives in `src/lib/parseMantra.ts`. If you add a new format, update the matching test in
`tests/parseMantra.test.ts` AND the README's "Editing Mantras" section.

### Styling

The look is intentional: dark background (`#050507`), purple accent lines, Playfair Display for headlines,
JetBrains Mono for UI chrome. Avoid bright colors or heavy UI elements — this app is meant to feel quiet and
meditative. Keep the 480x320 constraint in mind for any layout change.

### Adding persistence / database

Not needed. `src/data/mantras.json` is the source of truth. If the list grows beyond ~100 entries, consider
reading it on each request instead of at startup so edits don't require restarts.

## Ports

| Service     | Port | Notes                                           |
|-------------|------|-------------------------------------------------|
| Express API | 3174 | Serves `/api/mantra` and static `dist/` in prod |
| Vite dev    | 5174 | Proxies `/api/*` → `3174` in dev                |

These ports are hardcoded. If you change them, update `server.js`, `vite.config.ts`, the README, and this file.

## CI / Release / Deploy

Three GitHub Actions workflows:

- **`.github/workflows/ci.yml`** — runs on every PR and push to `main`. Does `typecheck`, `test`, `build` (with
  `VITE_BASE=/<repo>/`), and a server smoke test (boot `server.js`, curl `/api/mantra`).
- **`.github/workflows/pages.yml`** — on push to `main`, builds the static bundle with `VITE_BASE=/<repo>/` and
  publishes to GitHub Pages via `actions/deploy-pages`. Requires Pages to be enabled in repo settings with "Source:
  GitHub Actions".
- **`.github/workflows/release.yml`** — on push to `main`, runs `semantic-release` (config in `.releaserc.json`).
  Uses conventional commits to decide the next SemVer bump, writes `CHANGELOG.md`, tags the release, and creates a
  GitHub release. Pushes a `chore(release): x.y.z [skip ci]` commit back to `main`.

### Conventional commits

Since `semantic-release` is driving version bumps, commit messages matter:

- `fix: ...` → patch bump
- `feat: ...` → minor bump
- `feat!: ...` or a `BREAKING CHANGE:` footer → major bump
- `chore:`, `docs:`, `test:`, `refactor:`, `ci:`, `build:` → no release

If you want to change how versioning is decided, edit `.releaserc.json`.

## Non-obvious Gotchas

- **`mantras.json` lives in `src/data/`**, not the repo root. Both `server.js` and the client import from there —
  that's intentional, so there's a single source of truth for both the API and the static fallback.
- **The leading-list-marker strip in `parseMantra`** requires whitespace after the `-` or `*` (`/^[-*]\s+/`). That
  prevents `**Bold**` from getting one of its stars stripped. Don't relax that regex without updating the tests.
- **`VITE_BASE`** defaults to `/` for local dev/preview so the Express server still serves assets correctly. Only
  the CI/Pages workflow sets it to `/mantra-app/`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/j1v37u2k3y) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-12 -->
