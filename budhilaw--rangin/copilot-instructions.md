## rangin

> Theme bootstrap lives in `functions.php`, which wires feature modules from `inc/` (setup, enqueue, widgets, portfolio CPT, optimization hooks). Page, archive, and single templates sit in the theme root (`front-page.php`, `page-*.php`, `single-portfolio.php`) and follow standard WordPress hierarchy. Source styles reside in `src/` (Tailwind inputs for front end and admin). Compiled assets are written to `assets/css` and `assets/js`; images and performance screenshots live under `assets/img`. The canonical stylesheet header that WordPress reads is `style.css`, so keep metadata in sync when versioning.

# Repository Guidelines

## Project Structure & Module Organization
Theme bootstrap lives in `functions.php`, which wires feature modules from `inc/` (setup, enqueue, widgets, portfolio CPT, optimization hooks). Page, archive, and single templates sit in the theme root (`front-page.php`, `page-*.php`, `single-portfolio.php`) and follow standard WordPress hierarchy. Source styles reside in `src/` (Tailwind inputs for front end and admin). Compiled assets are written to `assets/css` and `assets/js`; images and performance screenshots live under `assets/img`. The canonical stylesheet header that WordPress reads is `style.css`, so keep metadata in sync when versioning.

## Build, Test, and Development Commands
- `npm install`: installs Tailwind, Terser, and tooling.
- `npm run dev`: watches `src/` and emits unminified CSS for frontend/admin.
- `npm run build`: produces minified CSS variants in `assets/css`.
- `npm run build:js`: compresses `assets/js/main.js` → `main.min.js` with terser.
- `npm run build:prod`: convenience alias for `npm run build && npm run build:js`.
- `composer install`: optional; pulls `matthiasmullie/minify` so the Asset Optimization panel can serve cached minified bundles.

## Coding Style & Naming Conventions
Follow WordPress PHP standards: 4-space indentation, early returns, and text domain `rangin` for translation functions. Prefix helper functions and hooks with `rangin_` to avoid collisions (e.g., `rangin_register_widgets`). Sanitize all output using `esc_html()`, `wp_kses_post()`, or context-appropriate helpers. Tailwind utilities drive layout—custom classes belong in `src/input.css` and should mirror component intent (`.card-portfolio`, `.badge-category`). Template filenames follow WordPress conventions (`single-portfolio.php`, `page-contact.php`) so add new templates with the same pattern.

## Testing Guidelines
No automated test suite exists; rely on manual QA inside a local WordPress instance. After PHP or template changes, flush permalinks and confirm each template (Front, About, Portfolio, Contact, Blog) renders without notices in `wp-content/debug.log`. For CSS/JS edits, run `npm run build:prod`, clear the Theme Options “Purge Minified Cache,” and smoke test in Chrome + Firefox responsive modes. Performance-sensitive changes should be validated with Lighthouse or GTMetrix, keeping TBT < 100 ms and CLS ≈ 0.01.

## Commit & Pull Request Guidelines
Commits use Conventional Commit syntax (`type(scope): summary`), as seen in `fix(): comment form is not showing`. Keep scopes short (`assets`, `templates`, `widgets`) and write imperative summaries. Pull requests must describe the motivation, list testing done (commands + browsers), and link any WordPress issue/ticket. Include before/after screenshots for UI-facing tweaks and mention if activation routines, Theme Options, or cached assets need re-running. Avoid bundling compiled `node_modules`; only commit files under version control today.

## Security & Configuration Tips
Guard every PHP entry point with `defined('ABSPATH')` checks (already present—mirror that style). Never commit real API keys to Theme Options defaults; prefer environment-driven fallbacks. When touching asset URLs, keep them relative to `THEME_URL` to support subdirectory installs. If you enable optional Cloudflare image rewriting, document the expected environment variables inside the PR description for future agents.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/budhilaw)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/budhilaw)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
