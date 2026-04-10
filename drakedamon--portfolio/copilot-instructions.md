## portfolio

> - Root CRA app: source in `src/`, static assets in `public/`, build output in `build/`.

# Repository Guidelines

## Project Structure & Module Organization
- Root CRA app: source in `src/`, static assets in `public/`, build output in `build/`.
- Next.js site: lives in `portfolio-nextjs/` with its own `src/`, `content/`, and config (`next.config.mjs`). Exports static files for GitHub Pages.
- Docs and assets: see `README.md`, `src/SRC_OVERVIEW.md`, and `src/images/`.

## Build, Test, and Development Commands
- Root (CRA):
  - `npm start`: run dev server at `http://localhost:3000`.
  - `npm test`: run Jest + Testing Library in watch mode.
  - `npm run build`: production build to `build/`.
  - `npm run deploy`: build and publish to GitHub Pages (via `gh-pages`).
- Next.js (`cd portfolio-nextjs`):
  - `npm run dev`: local Next.js dev server.
  - `GITHUB_PAGES=true npm run build`: static export with sitemap.
  - `npm run deploy`: export to `out/` and push to `gh-pages` (adds `.nojekyll`).
  - `npm run lint` / `npm run typecheck`: ESLint and TypeScript checks.

## Coding Style & Naming Conventions
- Indentation: 2 spaces; prefer semicolons and ES modules.
- React components: PascalCase (`MyComponent.jsx/tsx`). Hooks: camelCase with `use` prefix.
- Files: CRA uses `.js/.jsx`; Next.js uses `.ts/.tsx` and MDX (`.mdx`).
- Lint/format: CRA extends `react-app` ESLint; Next.js uses `eslint-config-next` and Prettier (`portfolio-nextjs/.prettierrc`). Run `npm run lint` where available.

## Testing Guidelines
- Frameworks: Jest + React Testing Library (CRA). No required tests in Next.js unless touching logic.
- Location/patterns: `**/*.test.js(x)` near sources or under `__tests__/`.
- Expectations: write focused unit tests for components and utils; prefer queries by role/label.
- Run: `npm test` (root). Consider adding smoke tests for critical pages.

## Commit & Pull Request Guidelines
- Commits: imperative mood, concise subject, optional scope (e.g., `feat(home): add hero CTA`).
- PRs: clear description, linked issue, screenshots/GIFs for UI, steps to verify, and notes on accessibility/perf where relevant.
- Keep diffs focused; include updates to docs and tests.

## Security & Configuration Tips
- Do not commit secrets. Use environment vars locally; avoid adding `.env` to VCS.
- GitHub Pages: root uses `homepage` in `package.json`; Next.js requires `GITHUB_PAGES=true` for proper `basePath`/`assetPrefix`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DrakeDamon)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/DrakeDamon)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
