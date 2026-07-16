## portfolio

> - `src/index.js` boots the React app; `src/App.js` wires routing and layout styles (`app.css`, `index.css`).

# Repository Guidelines

## Project Structure & Module Organization
- `src/index.js` boots the React app; `src/App.js` wires routing and layout styles (`app.css`, `index.css`).
- Route-level views live in `src/pages` (e.g., `homepage.jsx`, `projects.jsx`, `readArticle.jsx`); shared UI lives in `src/components` (`homepage`, `projects`, `about`, `articles`, `common`).
- Content/config data sits in `src/data`; testing scaffold is in `src/App.test.js` with helpers in `src/setupTests.js`.
- Static assets and the HTML shell live under `public/`; Tailwind and global styling tokens are defined in `tailwind.config.js`.

## Build, Test, and Development Commands
- `npm install` — install dependencies (React 18, react-scripts 5, Tailwind, FontAwesome, GA4).
- `npm start` — run the dev server with hot reload at `http://localhost:3000`.
- `npm run build` — produce a production build in `build/` via `react-scripts`.
- `npm test` — run Jest/React Testing Library in watch mode.
- `npm run deploy` — publish the `build/` output to GitHub Pages (ensure `npm run build` succeeds first).

## Coding Style & Naming Conventions
- Prettier defaults: tabs enabled, `tabWidth: 4`. Keep JSX/JS files formatted with these settings.
- Use functional components with hooks; name components and files in PascalCase (`ProjectCard.jsx`), data/constants in `camelCase`, and assets with kebab-case.
- Prefer Tailwind utility classes for layout/spacing; fall back to `app.css`/`index.css` for globals and to `styled-components` only where already used.
- Keep imports ordered: React/vendor, utils/data, components, styles. Avoid unused exports and dead CSS.

## Testing Guidelines
- Testing stack: Jest with React Testing Library and `@testing-library/jest-dom`.
- Co-locate tests as `*.test.js[x]` near the component under test; keep assertions user-focused (text, roles, labels) rather than implementation details.
- Run `npm test` before sending a PR; for CI-like runs, use `CI=true npm test -- --watch=false`.

## Commit & Pull Request Guidelines
- Commit messages: short imperative summary (e.g., `Add project card animation`, `Fix article route guard`); group related changes per commit.
- PRs: include a brief description, screenshots/GIFs for UI updates, and link any related issues. Note new dependencies/config changes (Tailwind tokens, analytics, env vars) explicitly.
- Ensure `npm test` passes and `npm run build` succeeds locally; flag any flaky or skipped tests with context.

---
> Source: [Apetun/portfolio](https://github.com/Apetun/portfolio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-16 -->
