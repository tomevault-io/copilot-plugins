## english-file-elementary-5th-u3-and-u4-a1

> This repository currently contains two standalone lesson pages at the root:

# Repository Guidelines

## Project Structure & Module Organization
This repository currently contains two standalone lesson pages at the root:
- `ENGLISH_FILE_ELEMENTARY_5TH_U3.html`
- `ENGLISH_FILE_ELEMENTARY_5TH_U4.html`

Keep related unit files in the project root using the same naming pattern (`ENGLISH_FILE_ELEMENTARY_5TH_U{N}.html`) for consistency. If shared assets are added later, place them in clear top-level folders such as `assets/` (images, audio) and `styles/` (shared CSS).

## Build, Test, and Development Commands
There is no build pipeline configured; files are static HTML.
- `start ENGLISH_FILE_ELEMENTARY_5TH_U3.html` opens Unit 3 in the default browser (Windows).
- `start ENGLISH_FILE_ELEMENTARY_5TH_U4.html` opens Unit 4 in the default browser.
- `npm test` should be run after modifying JavaScript (project working agreement), even if tests are added later.

If Node-based tooling is introduced, prefer `pnpm` for dependency installation.

## Coding Style & Naming Conventions
- Use 2 spaces for indentation in HTML, CSS, and inline JavaScript.
- Prefer semantic HTML (`header`, `main`, `section`, `button`, `label`) and meaningful `id`/`class` names.
- Keep filenames descriptive and unit-scoped (example: `ENGLISH_FILE_ELEMENTARY_5TH_U5.html`).
- Keep inline scripts minimal; extract repeated logic when JavaScript files are introduced.

## Testing Guidelines
No automated test framework is currently configured. Before submitting changes:
- Open each edited HTML file in a browser.
- Verify interactions, media, and layout at desktop and mobile widths.
- If JavaScript is changed, run `npm test` and include results in the PR notes.

## Commit & Pull Request Guidelines
Git history is not available in this workspace, so adopt a clear convention:
- Commit format: `type(scope): short summary` (example: `feat(u4): add frequency adverb drag-and-drop check`).
- Keep commits focused on one change set.
- PRs should include: purpose, files changed, manual test steps, and screenshots/GIFs for UI changes.
- Link related issue/task IDs when available.

## Agent-Specific Notes
- Ask for confirmation before adding new production dependencies.
- Workspace-write edits should stay scoped to the requested task.

---
> Source: [rbisposspp/ENGLISH_FILE_ELEMENTARY_5TH_U3_and_U4_A1](https://github.com/rbisposspp/ENGLISH_FILE_ELEMENTARY_5TH_U3_and_U4_A1) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-27 -->
