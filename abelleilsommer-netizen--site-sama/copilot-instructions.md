## site-sama

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

This is a **static site export from Framer** (framer.com), not a hand-authored codebase. It's the published output for a marketing site for SAMA, a French consulting firm specializing in strategy and management for multi-site restaurants (restauration multisite, collective, commerciale et rapide).

There is no source project, build tooling, package manager, or framework config here — only the generated static output:

- `index.html`, `a-propos.html`, `actualites.html`, `contact.html`, `expertises.html`, `mention-legales.html`, `politique-confidentialite.html` — the site's pages (French).
- `js/` — Framer's minified/bundled runtime (`.mjs` chunks with hashed filenames), plus `rerouter.js` and `init.mjs` which remap `framerusercontent.com` / `app.framerstatic.com` URLs to local `images/`/`js/` paths so the export works standalone.
- `images/` — all site assets (fonts as `.woff2`, images, icons), renamed to hashed filenames matching the URL remapping table in `js/rerouter.js`.

## Working in this repo

- **There is no build/lint/test command.** There's no `package.json`, no bundler config, and no test suite — this is flat HTML/CSS/JS meant to be served as-is by a static file server.
- **Do not hand-edit the `.mjs` files in `js/`.** They are minified Framer framework output tied to hashed filenames referenced from every HTML page and from the URL-remapping table embedded in `js/rerouter.js`. Any real content or design change belongs upstream in the Framer project and gets re-exported — treat this directory as a build artifact, not source.
- Each HTML page is a full, self-contained, largely duplicated document (inline `<style data-framer-font-css>` blocks, same font-face declarations, same `<script src="js/rerouter.js">` bootstrap). Expect near-total duplication of `<head>` boilerplate across pages — that's how Framer exports work, not an issue to "fix".
- If asked to change visible copy, layout, or styling, edit the relevant page's HTML/inline CSS directly — the structure is verbose generated markup (framer-generated class names, inline styles), so search for the specific French text string first rather than trying to map out component structure.
- To preview the site locally, serve the directory root with any static file server (e.g. `npx serve .` or `python -m http.server`) from this folder — opening `index.html` directly via `file://` will break the base-href-relative asset paths in some cases.

---
> Source: [abelleilsommer-netizen/site-sama](https://github.com/abelleilsommer-netizen/site-sama) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-05 -->
