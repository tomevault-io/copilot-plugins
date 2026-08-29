## dragonvstigerbot-com

> Single static page (`index.html`) implementing a Dragon Tiger prediction/signal tracker. No framework, no build step, no backend, no database — all state (result history) lives in a JS array in memory for the current browser tab only.

# AGENTS.md

## Project

Single static page (`index.html`) implementing a Dragon Tiger prediction/signal tracker. No framework, no build step, no backend, no database — all state (result history) lives in a JS array in memory for the current browser tab only.

## Structure

- `index.html` — everything: markup, `<style>` CSS, and `<script>` logic in one file.
- `assets/bg.jpg` — user-supplied background image, also reused as favicon/apple-touch-icon.
- `netlify.toml` — publishes the repo root as-is (no build command needed).

## Conventions

- Keep this as a single self-contained HTML file unless the app grows enough to justify a build step — don't introduce a framework for what is fundamentally a static tool.
- Result values are single-letter codes throughout: `D` (Dragon), `T` (Tiger), `A` (Tai).
- The signal algorithm in `getSignal()` runs a fixed set of five named "tricks," each optionally casting one vote for D/T/A; the majority vote wins, ties/no-votes fall back to the last entered result, and a final `A` result is always converted to `D` (per product rule — Tai is never presented as the final bet suggestion).
- No persistence layer exists on purpose: history resets on page reload. If persistence is ever requested, use Netlify Blobs or Netlify Database (see the `netlify-blobs`/`netlify-database` skills) rather than localStorage, to keep behavior consistent across devices.

---
> Source: [dilawarkhanje1-beep/dragonvstigerbot.com](https://github.com/dilawarkhanje1-beep/dragonvstigerbot.com) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-29 -->
