## test-25-8-16

> - `index.html`: Main dashboard page rendered with Tailwind (CDN) + DaisyUI.

# Repository Guidelines

## Project Structure & Module Organization
- `index.html`: Main dashboard page rendered with Tailwind (CDN) + DaisyUI.
- `frontend/index.js`: Entry script that loads `Apps/Apps.json` and renders app cards.
- `Apps/Apps.json`: Source of app metadata (name, path, section, colors, icons).
- `Apps/_components/`: Vanilla JS UI utilities (`Header.js`, `Sidebar.js`, `Settings.js`, `ThemeSwitcher.js`).
- `Apps/docs/`: Lightweight docs site built from Markdown pages via `index.js`.
- `tmp/`: Optional helpers (TypeScript/TSX drafts and `build.sh`). Not required for normal usage.

## Build, Test, and Development Commands
- Run locally (recommended): `python3 -m http.server 5173` → open `http://localhost:5173/`.
- Open docs: `http://localhost:5173/Apps/docs/` (or open `Apps/docs/index.html` directly).
Notes: There is no bundler; scripts use standard ES modules and relative paths for GitHub Pages.

## Coding Style & Naming Conventions
- Language: Browser JavaScript (ES modules), no framework.
- Indentation: tabs; Quotes: double (`""`); End statements with semicolons.
- Components: PascalCase filenames in `Apps/_components/` and factory functions like `createHeader()`.
- Data: Keep `Apps/Apps.json` sorted by section; use kebab-case for `slug`/`id` values.
- CSS: Prefer DaisyUI utility classes; avoid inline styles.

## Testing Guidelines
- No formal test suite yet. Validate changes by serving locally and checking:
  - Home page renders cards, sections, and icons without console errors.
  - Docs pages load Markdown and update via hash navigation.
- If adding tests later, place files under `tests/` with `*.spec.js` naming.

## Commit & Pull Request Guidelines
- Commits: Use Conventional Commit style when possible: `type(scope): summary` (e.g., `docs(app): build docs index`). Use imperative mood and concise subjects.
- PRs: Include a clear description, linked issues, before/after screenshots for UI, and a manual test plan (URLs touched, expected behavior).
- Keep diffs focused; update `Apps/Apps.json` and docs together when introducing or renaming apps.

## Security & Configuration Tips
- Do not commit secrets. External libraries are loaded via CDN; pin to known versions when updating.
- Serve over HTTP locally to avoid fetch/CORS issues with `Apps/Apps.json`. Use relative paths for GitHub Pages deployment.

---
> Source: [imarfanc/test-25.8.16](https://github.com/imarfanc/test-25.8.16) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-23 -->
