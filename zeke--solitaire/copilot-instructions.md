## solitaire

> This is a small self-contained project that recreates the Windows Solitaire win animation as a web page deployed to Cloudflare Workers.

# Agent instructions

This is a small self-contained project that recreates the Windows Solitaire win animation as a web page deployed to Cloudflare Workers.

## Project structure

- `index.html` — the complete animation with an embedded base64 sprite sheet. This is the source of truth.
- `public/index.html` — copy of `index.html` served by the Cloudflare Worker. Must be kept in sync.
- `src/worker.js` — Hono app that acts as a fallback for non-asset routes. Static assets are served automatically.
- `wrangler.jsonc` — Cloudflare Workers config.
- `extract_cards.py` — Python script that extracts card bitmaps from `cards.dll` and builds the sprite sheet.
- `cards.dll` — the original Windows XP cards DLL containing bitmap resources.
- `cards.png` — the extracted sprite sheet (13 columns x 4 rows, 71x96 per card).
- `cards/` — individual extracted card images (gitignored).

## Key details

- Card dimensions: 71x96 pixels.
- Sprite sheet layout: 13 columns (Ace through King) x 4 rows (Clubs, Diamonds, Hearts, Spades).
- The sprite sheet is base64-encoded inline in `index.html`. If you regenerate `cards.png`, you need to re-encode and update the data URI.
- The trail effect works by never calling `clearRect` on the canvas. Each physics step draws the card at its new position on top of the existing canvas content.
- Physics runs at ~16fps via `setInterval` to get authentic trail spacing between card images.

## Deploying

```sh
npm install
npx wrangler deploy
```

After editing `index.html`, always copy it to `public/index.html` before deploying.

## Important

- The deployed URL is https://solitaire.ziki.workers.dev
- The GitHub repo is private at https://github.com/zeke/solitaire
- Don't push to main without asking.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zeke)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/zeke)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
