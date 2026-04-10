## natural-selection

> The extension source lives in `src/`. `manifest.json` defines the Manifest V3 extension (loadable in Chrome, Edge, and Firefox). `content.js` intercepts ChatGPT keyboard events; update it when adjusting core behavior. `popup.html` provides the toolbar info panel, while `icon*.png` provide assets. Release artifacts belong in `dist/` (`chatgpt_shift_arrow_fix-<version>` directory plus zipped bundle). `console-test.md` holds the standalone snippet for quick manual verification; keep it aligned with `content.js`.

# Repository Guidelines

## Project Structure & Module Organization
The extension source lives in `src/`. `manifest.json` defines the Manifest V3 extension (loadable in Chrome, Edge, and Firefox). `content.js` intercepts ChatGPT keyboard events; update it when adjusting core behavior. `popup.html` provides the toolbar info panel, while `icon*.png` provide assets. Release artifacts belong in `dist/` (`chatgpt_shift_arrow_fix-<version>` directory plus zipped bundle). `console-test.md` holds the standalone snippet for quick manual verification; keep it aligned with `content.js`.

## Build, Test, and Development Commands
Use `npx web-ext run --source-dir src --target=firefox-desktop` to launch a temporary Firefox profile. For Chromium-based browsers, load `src/` via `chrome://extensions → Load unpacked`. Generate a release zip with `zip -r dist/chatgpt_shift_arrow_fix-<version>.zip src/*`. When iterating manually, reload via `about:debugging#/runtime/this-firefox → Load Temporary Add-on → src/manifest.json`.

## Coding Style & Naming Conventions
JavaScript files use strict mode, arrow functions, and 2-space indentation. Prefer `const` for immutable bindings and single quotes for strings. Keep console noise minimal and prefix any future diagnostic messages with `[Natural Selection]` for traceability. Name new assets or scripts descriptively within `src/`, and avoid uppercase filenames except for icons.

## Testing Guidelines
There is no automated test harness yet; rely on manual verification. After loading the extension, open ChatGPT, focus any editor, and confirm Shift+Arrow selects text without jumping conversations. Update `console-test.md` whenever API changes require a matching console snippet.

## Commit & Pull Request Guidelines
Follow the short-imperative style seen in `git log` (e.g., `Add guard for focus jumps`). Each PR should describe scope, testing performed, and any ChatGPT UI versions affected. Link related issues and include before/after notes or short screen recordings when UI behavior changes. Attribute authorship to the humans involved—never credit Codex, Claude, or any other AI. Ensure bundled artifacts in `dist/` match the code in `src/` before requesting review.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/banagale)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/banagale)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
