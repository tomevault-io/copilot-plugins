## feed-destroyer

> This file is the canonical guide for future agents working on this reboot.

# Feed Destroyer Agent Instructions

This file is the canonical guide for future agents working on this reboot.

## Product Intent

Feed Destroyer is a small privacy-preserving Chrome extension for removing distracting web surfaces. Keep it simple, fast, local, and always-on unless the user explicitly asks for controls.

## Engineering Rules

- Prefer TypeScript for reusable logic.
- Keep Manifest V3 compatibility.
- Do not add tracking, analytics, telemetry, remote config, or network calls.
- Do not store browsing history or page content.
- Do not write, print, commit, or document real secrets.
- Do not reference private local paths in README, code, examples, or docs.
- Prefer robust CSS selectors and small DOM state detection over brittle click automation.
- Use MutationObserver only for lightweight state refreshes on SPA navigation or tab changes.
- Keep build output in `dist/`; do not hand-edit generated files.
- Keep the popup focused on the local focus-target input unless the user explicitly asks for more controls.
- The selected icon source is `src/icons/icon-source.png`; regenerate `icon-16.png`, `icon-32.png`, `icon-48.png`, and `icon-128.png` from that source.

## Current Behavior

- YouTube: hide feeds, recommendations, Shorts surfaces, comments, live chat, mixes, end-screen cards, and related distraction panels while preserving direct video playback, search, subscriptions, and channels.
- X: keep the `For you` tab visible, hide only the Home timeline contents when `For you` is active, and leave `Following` usable.
- Popup: no toggles; store a local focus target with `chrome.storage.local` and render it in the blocked feed area.

## Verification

Run:

```bash
npm run check
```

Then load the project folder through `chrome://extensions` and manually verify YouTube and X behavior in Chrome.

---
> Source: [Andriy-Kulak/feed-destroyer](https://github.com/Andriy-Kulak/feed-destroyer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-30 -->
