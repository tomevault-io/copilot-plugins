## my-creator-landing

> - Next.js Pages Router lives in `pages/` (e.g., `pages/index.js`, `pages/_app.js`). Dynamic routes under folders like `pages/creator/[id].js`.

# Repository Guidelines

## Project Structure & Module Organization
- Next.js Pages Router lives in `pages/` (e.g., `pages/index.js`, `pages/_app.js`). Dynamic routes under folders like `pages/creator/[id].js`.
- Reusable UI in `components/` (e.g., `components/layout/Header.jsx`, `components/public/Hero.jsx`).
- Global styles in `styles/globals.css` (Tailwind entry). Static assets in `public/` (e.g., `/favicon.ico`, `/manifest.json`).
- Legacy CRA/Vite artifacts in `src/` and `legacy/`; not used at runtime. Avoid edits unless migrating.
- Root config: `next.config.js`, `tailwind.config.js`, `postcss.config.js`. Legacy: `vite.config.js`, `index.html`.

## Build, Test, and Development Commands
- `npm install` — install dependencies (Node 18+).
- `npm run dev` — start Next dev server at http://localhost:3000.
- `npm run build` — production build via `next build`.
- `npm start` — serve built app with `next start`.
- `npm run lint` — run ESLint with `eslint-config-next`.

## Coding Style & Naming Conventions
- Language: JavaScript/JSX with 2‑space indentation.
- Components use `PascalCase.jsx` (e.g., `CreatorProfile.jsx`). Utilities/hooks use `camelCase.js`.
- Use clear relative imports (no path aliases). Keep components small and composable.
- Fix ESLint warnings before commit; prefer simple, focused diffs.

## Testing Guidelines
- No test runner is currently wired. Legacy Jest files may exist under `src/`.
- If adding tests, use Jest + React Testing Library; name files `*.test.js` colocated or under `__tests__/`. Keep tests deterministic and offline.

## Commit & Pull Request Guidelines
- Commits follow Conventional Commits, e.g., `feat(dashboard): add analytics card`.
- PRs include: concise description, linked issues, and screenshots/GIFs for UI changes.
- Gate: `npm run lint` must pass and `npm run build` should succeed locally before review.

## Security & Configuration Tips
- Never commit secrets. Use `.env.local`; see `.env.example` for required vars. Rotate exposed keys immediately and redeploy.

## Migration Notes (Legacy CRA/Vite)
- Move any remaining UI from `src/` into `pages/`/`components/` and serve assets from `public/`.
- Replace Vite/CRA-specific imports with Next-compatible ones; update relative paths. Validate with `npm run dev`, then `npm run build`.

## Agent-Specific Instructions
- Prefer surgical changes aligned to the Pages Router. Avoid editing `src/` or Vite artifacts unless migrating. Reference exact paths and npm scripts in explanations.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tdiorio2323)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/tdiorio2323)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
