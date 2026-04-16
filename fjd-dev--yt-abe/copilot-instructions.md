## yt-abe

> Maintain a Deno-first TypeScript scraper that uses `youtubei.js` to collect:

# Project context

## Goal

Maintain a Deno-first TypeScript scraper that uses `youtubei.js` to collect:

- all community posts for a channel
- top-level comments for each post
- all reply pages for each top-level comment
- a compact JSON output with only relevant fields

## Engineering constraints

- Runtime is Deno.
- Keep `youtubei.js` as the primary integration library.
- Prefer public library APIs first.
- Avoid introducing new dependencies unless there is a concrete need.
- Preserve the output schema unless the user explicitly asks to change it.
- Preserve readable, small functions and explicit naming.
- Favor pure helpers for serialization and argument parsing.

## Important implementation detail

The scraper must accumulate comment and reply continuation batches explicitly. Do not assume a continuation object still contains earlier pages.

## Output expectations

- Do not dump raw InnerTube payloads into the JSON output.
- Keep only stable, relevant fields.
- Continue scraping other posts when one post's comments fail.
- Represent per-post comment failures with `loadError` instead of aborting the whole run.

## Style

- Use strict equality.
- Keep functions focused.
- Make errors actionable.
- Keep CLI UX simple and explicit.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fjd-dev) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
