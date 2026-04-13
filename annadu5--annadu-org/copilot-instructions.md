## annadu-org

> - `docs/` holds the live site files served in production.

# Repository Guidelines

## Project Structure & Module Organization
- `docs/` holds the live site files served in production.
  - `docs/index.html` contains the page template and Vue bindings.
  - `docs/js/app.js` stores the content data (news, videos, competitions, research).
  - `docs/js/main.js` includes jQuery UI interactions (menus, scrolling, carousels).
  - `docs/css/style.css` is the custom theme styling.
  - `docs/img/` and `docs/pdf/` store static assets.
- `archive/` keeps the previous/archived website.
- `.github/workflows/` contains GitHub Actions deployment workflows.

## Build, Test, and Development Commands
- No build tooling or package manager is used.
- Local preview: open `docs/index.html` directly in a browser.
- Deploy workflow (branch-based):
  - Push to `github-pages` to publish the testing site.
  - Merge to `master` to sync to the production S3 site.

## Coding Style & Naming Conventions
- Keep HTML, CSS, and JS simple and readable; follow existing formatting.
- Indentation appears to be 2 spaces in HTML/CSS/JS—match the surrounding file.
- Content entries in `docs/js/app.js` use consistent object shapes; preserve key names and ordering.
- Use descriptive, sentence-case labels for new content entries (e.g., “Author Talks with Anna Du”).

## Testing Guidelines
- There are no automated tests in this repository.
- For changes, manually verify in a browser by opening `docs/index.html` and checking layout, links, and interactive elements.

## Commit & Pull Request Guidelines
- Recent commits use short, descriptive, sentence-case messages (often tied to content updates).
- When opening a PR, include:
  - A brief summary of the change.
  - The target branch (`github-pages` for testing, `master` for production).
  - Screenshots or a short note for any visual or content changes.

## Deployment Notes
- Testing site: deploys from `github-pages` to the GitHub Pages preview.
- Production site: deploys from `master` to the S3 bucket backing `annadu.org`.

## Agent-Specific Instructions
- Update site content via `docs/js/app.js`; avoid introducing new tooling unless required.
- Keep edits minimal and scoped to the requested content or visual changes.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/annadu5) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
