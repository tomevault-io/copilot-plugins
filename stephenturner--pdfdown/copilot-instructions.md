## pdfdown

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A fully client-side PDF→markdown converter, deployed on GitHub Pages at https://stephenturner.github.io/pdfdown/. The entire app (HTML, CSS, and a `type=module` script) lives in `index.html`. There is no framework, no build step for the app itself, and no server component: PDFs are converted in the browser.

## Commands

```sh
python3 -m http.server 8000   # serve locally; module scripts won't load from file://
npm run build                 # rebuild both vendor bundles (only needed when bumping deps)
```

There is no test framework or linter. Changes have been verified by serving the directory and driving the page in headless Chrome:

```sh
"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" --headless --disable-gpu \
  --virtual-time-budget=15000 --dump-dom http://localhost:8000/<smoke-page>.html
```

where the smoke page loads `index.html` in an iframe, drives the DOM (submit the URL form, poll `#status` until it reports Done/error), and writes results into a report div that the `--dump-dom` output is grepped for. Delete smoke pages afterward; they are not checked in.

## Architecture

- `index.html` — everything. Input paths (file picker, drag-drop, paste, URL form, `#url=` hash) all funnel into `convertBytes(bytes, name)`, which calls `pdf2md()` and renders the split view: editable markdown `<textarea id="out">` on the left, sanitized preview `<div id="preview">` on the right. Editing the textarea re-renders the preview (debounced). Export buttons: copy/download .md on the left, copy rich text (`ClipboardItem` with text/html + text/plain) and download standalone .html on the right.
- `vendor/pdf2md.js` (~1.6 MB) and `vendor/markdown.js` — esbuild browser bundles of `@opendocsg/pdf2md` (conversion; uses pdf.js via unpdf, no separate worker file needed) and marked+DOMPurify (`renderMarkdown(md)`). The bundles are **checked in deliberately** so Pages needs no build step; `npm install` is only for rebuilding them from `bundle-entry.js` / `markdown-entry.js`.
- Shareable links: a successful URL conversion sets `#url=<encoded-pdf-url>` via `history.replaceState`; page load and `hashchange` read it back and auto-convert. Only http/https URLs are accepted from the hash. A local-file conversion clears the hash.

## Constraints

- Must keep working on GitHub Pages from the `/pdfdown/` subpath: static files only, all paths relative, `.nojekyll` must remain.
- All markdown-derived HTML must go through `renderMarkdown()` (DOMPurify) — PDF content is untrusted input.
- URL fetching is subject to CORS; the remote server must send CORS headers. Error messages already explain this, keep that behavior.
- `.claude/` is gitignored because `settings.local.json` holds API credentials; never commit it or weaken that ignore rule.
- A Firefox extension version is a long-term goal (being developed elsewhere); keep the conversion pipeline decoupled from input handling (`convertBytes` as the single entry point) to ease reuse.

---
> Source: [stephenturner/pdfdown](https://github.com/stephenturner/pdfdown) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-25 -->
