## beautifulbitcoinblocks

> - Name: Beautiful Bitcoin Blocks.

# Project overview

- Name: Beautiful Bitcoin Blocks.
- Purpose: For each Bitcoin block, generate:
  - A short poem about that block.
  - An abstract image inspired by that block.
- Storage model:
  - No hosted database.
  - All block data, poems, and image references are stored as JSON files in the repo.
  - Site is statically built and deployed in github pages

# High-level architecture

- Framework: Next.js (App Router, TypeScript)
- Data source:
  - `public/blocks/<height>.json` for each processed block.
  - `public/blocks/index.json` as an index of all processed blocks.
- Generation:
  - A script or small CLI (can be run locally or in CI) fetches new blocks from a Bitcoin API and writes JSON files.
  - Poem and image are generated via external AI APIs in that script, not in the browser.
- Deployment:
  - Static export or SSG build reads JSON from `public/blocks` and generates pages.

# Data model (JSON files)

- `public/blocks/<height>.json`:

  ```json
  {
    "height": 888888,
    "hash": "0000000000000000000...",
    "timestamp": "2026-01-07T12:34:56Z",
    "tx_count": 1234,
    "total_output_btc": 12.3456789,
    "mood_label": "euphoric",
    "prompt_text": "Write a 8-line free verse poem about a busy Bitcoin block...",
    "poem": "…",
    "image_url": "/blocks/images/888888.png",
    "created_at": "2026-01-07T12:40:00Z"
  }

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ankitchiplunkar)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ankitchiplunkar)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
