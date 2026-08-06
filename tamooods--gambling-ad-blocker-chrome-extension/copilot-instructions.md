## gambling-ad-blocker-chrome-extension

> Chrome Extension (Manifest V3) that hides gambling ads on any webpage by scanning for Thai + English gambling keywords.

# Gambling Ad Blocker — AGENTS.md

## What this is

Chrome Extension (Manifest V3) that hides gambling ads on any webpage by scanning for Thai + English gambling keywords.

## How to load / develop

No build step, no npm, no bundler. The extension is ready to use as-is:

1. Open `chrome://extensions`
2. Enable **Developer mode** (toggle top right)
3. **Load unpacked** → select this repo root
4. Reload the extension from the same page after every code change

## Architecture

- **`manifest.json`** — Manifest V3; content script runs on `<all_urls>` at `document_start` with `all_frames: true` (runs inside iframes too)
- **`content.js`** — IIFE that scans `img`, `a`, `iframe`, `div/span/p` for keywords; uses `MutationObserver` for lazy-loaded ads; cluster-detection groups link+img banners
- **`hide.css`** — `.gab-hidden` class with `display: none !important`
- **`popup.html` / `popup.js`** — toggle on/off + blocked count via `chrome.storage.local`
- **`icon.png`** — extension icon

## Key conventions

- All scanned elements get `data-gab-checked="1"` to avoid re-scanning
- Hidden elements get `data-gab-hidden="1"` + class `gab-hidden`
- Keywords list lives in `content.js:10-25`
- Toggle state stored under `gabEnabled` key; count under `gabBlockedCount`
- Popup ↔ content script communication via `chrome.tabs.sendMessage` with `{ type: "GAB_TOGGLE", enabled }`

## No tooling

No package.json, no lockfile, no test runner, no linter, no CI, no formatter. Files are plain JS/CSS/HTML with no pre/post-processing.

---
> Source: [tamooods/gambling-ad-blocker-chrome-extension](https://github.com/tamooods/gambling-ad-blocker-chrome-extension) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-31 -->
