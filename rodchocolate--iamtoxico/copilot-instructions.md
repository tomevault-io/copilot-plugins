## iamtoxico

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Local dev server (Python/Flask valet backend)
./start.sh                  # http://localhost:8080
                            # preview at http://localhost:8080/preview.html

# Python deps
pip install flask flask-cors requests python-dotenv Pillow

# Tests
cd tests && pytest          # or: pytest from repo root (configured in pytest.ini)

# Shopify/Printify tooling (from shopify-app/)
python shopify-app/scan_printify.py          # scan all Printify products + image URLs
python shopify-app/_dump_staging.py          # dump staging product data
python shopify-app/add_amazon_items.py       # add affiliate items to catalog.json
python scripts/resolve_printify_sources.py   # map Printify artwork IDs back to local source assets
python scripts/export_rename_brief.py        # build Gemini-friendly rename brief with mockups + source candidates
python scripts/publish_apres_tops.py         # clone/publish Apres tops from live Harem families and sync catalog
python scripts/refresh_preview.py            # prune deleted preview entries and refresh mockup URLs from Printify
```

## Architecture

### Pages

| File | Purpose | Access |
|------|---------|--------|
| `index.html` | Public storefront — Printify originals via `data/catalog.json` | Public |
| `collection.html` | Product grid — fetches `data/catalog.json` filtering `source: "printify"` | Public |
| `preview.html` | Staging grid — hardcoded `P` array, front/back hover, live/staging filter | Internal |
| `shop.html` | Static shop page | Public |
| `landing.html` | Landing/marketing page | Public |

### Backend — `server.py` (Flask)

Development-only local server. Endpoints:
- Static file serving for all HTML
- `/spotify/auth`, `/spotify/callback`, `/api/spotify/*` — Spotify OAuth + playlist import
- `/api/valet/catalog` — serves `data/catalog.json` (Printify originals + affiliate items)

In production, `index.html` and `collection.html` fall back to fetching `data/catalog.json` directly (no server needed for static deploy).

### Data — `data/catalog.json`

Central product catalog. Two item types:
- `source: "printify"` — live Printify products (synced via `shopify-app/`)
- `source: "affiliate"` — Amazon/partner items added via `shopify-app/add_amazon_items.py`

Top-level `meta`, `activities`, and `vibes` keys power the "valet/shopping guide" feature.

### Shopify/Printify integration — `shopify-app/`

- `printify_connector.py` — Printify REST API wrapper (products, images, orders)
- `shopify_connector.py` — Shopify Admin API wrapper (products, webhooks, OAuth)
- `scan_printify.py` — scans shop and outputs product/mockup data
- `_dump_staging.py` — extracts staging product data for preview.html updates
- `add_amazon_items.py` — adds affiliate items to `data/catalog.json`

API credentials in `shopify-app/.env` (gitignored).

### Printify shops

- Shop `26651009` — `iamtoxico.myshopify.com` (live Shopify-connected, 6 published products)
- Shop `22994552` — disconnected staging store (~39 products in development)

### preview.html — how to add new products

The `P` array in `preview.html` is manually maintained. Each entry:
```js
{t:"Product Name", y:"category", s:"live|staging",
 f:"<front mockup URL>", b:"<back mockup URL>"}
```
Get mockup URLs from: `python shopify-app/scan_printify.py` or the Printify API directly.
Keep entries sorted: LIVE alphabetically first, then STAGING alphabetically.

## Current product line

**Live (Shopify):** Arctic River, Cyan Glow, Frozen Sky, Hold Cards, Hot Oil, Toxico — all Harem Pants.

**Staging categories:** harem pants, zip hoodies, cover/pullover hoodies, shorts, blanket, kimono robe, long sleeve tee, boxy tee, women's thong, windbreaker, puffer jacket, comforter, full-zip hoodie.

## Key planning docs

- `TOXICO_LIFE_PLANNING.md` — toxico-life.html third page (affiliates, resellers, collabs)
- `README.md` — underwear capsule spec (Aug 2025–Jan 2026)
- `agents.md` — 7-agent architecture and scope assignments
- `docs/` — brand, design, legal, marketing, printify, shopify sub-plans
- `PRODUCTS_TO_ADD.md` — affiliate/reseller queue (Zimmerli, Grailed, Reverb, etc.)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rodchocolate) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
