## article-overload

> Chrome extension (Manifest V3) that analyzes long-form X posts using the Anthropic API.

# Article Overload

Chrome extension (Manifest V3) that analyzes long-form X posts using the Anthropic API.

## What It Does

When a user visits a post URL (`x.com/*/status/*`), a sidebar slides in from the right showing:

- **Word count & read time** — basic stats for the post
- **Slop-o-Meter** — animated gauge scoring 0-100 for how AI-generated the text appears
- **Key Takeaways** — 3-4 concise bullet points summarizing the article

Both the slop score and takeaways are fetched in parallel via the Anthropic Messages API (claude-opus-4-6).

## Architecture

Three files, no build step:

- `manifest.json` — MV3 manifest. Content script runs on x.com. Background service worker handles API calls.
- `background.js` — Message handler for API key storage (chrome.storage.local) and proxying Anthropic API requests.
- `content.js` — All UI and logic. Injects a Shadow DOM sidebar, extracts article text from the page, calls Claude via the background script, caches results per URL.

## Key Details

- Shadow DOM isolates styles from the X page
- Results are cached in `chrome.storage.local` keyed by normalized URL
- Posts under 100 words are skipped (MIN_WORDS constant)
- URL changes are detected via `history.pushState`/`popstate` monkey-patching + MutationObserver
- API key is stored locally and validated with an `sk-` prefix check
- Uses `anthropic-dangerous-direct-browser-access` header for browser-based API calls

## Permissions

- `storage` — Save the user's API key and cached analysis results in `chrome.storage.local`
- `contextMenus` — Right-click menu on the extension icon with a "Remove API Key" option
- `host_permissions` (`https://api.anthropic.com/*`) — Proxy requests from the background service worker to the Anthropic Messages API

## Development

- Always use `source venv/bin/activate` before running any `python` or `pip` commands

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gwintrob) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
