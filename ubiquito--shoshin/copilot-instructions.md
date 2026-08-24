## shoshin

> Static HTML/CSS/JS. No framework. No build step. No tracking.

# Shoshin — A Practice in Beginner's Mind

Static HTML/CSS/JS. No framework. No build step. No tracking.

## Architecture
- Each practice tool is a self-contained HTML file
- Module pages (module-01 through module-12) link to practice tools
- index.html is the landing/journey page
- enter.html is the license key portal (public, not gated)
- Progress tracked in localStorage (key: haiku-lab-zen-progress)
- Sound via Web Audio API (singing bowls, tones, water noise)
- Calligraphy via canvas (stroke.html)

## Access gate (Gumroad + Cloudflare)
Sold via Gumroad. Access controlled by license key + HttpOnly cookie.

- `functions/_middleware.js` — auth gate on all pages except /enter and /api/*
- `functions/api/webhook.js` — receives Gumroad purchase/refund pings, stores/revokes keys in KV
- `functions/api/verify.js` — validates key from enter.html form, sets cookie
- KV binding: `SHOSHIN_KEYS` (keys stored as `key:<license_key>`)
- Cookie: `shoshin_key`, HttpOnly, Secure, SameSite=Lax, 1 year
- Direct link flow: `?key=TOKEN` in URL → middleware validates → cookie set → redirect to clean URL
- Env vars: `GUMROAD_PRODUCT_ID` (required, used to verify keys against Gumroad API), `GUMROAD_SELLER_ID` (optional, early rejection of spoofed pings)

## Serving locally
```
npx wrangler pages dev . --kv SHOSHIN_KEYS --port 8080
```
Falls back to `bunx serve . -l 8080` for static-only dev (no auth).

## Deploying
Cloudflare Pages. Point at the repo root. No build command. Static files + Functions.

## Design principles
- Rice paper (#f8f6f1) and ink (#1a1a1a). Nothing else.
- No progress bars. No gamification. No encouragement.
- Each tool asks something of the body or of time — not just reading.
- The tool gets out of the way. The practice is what remains.

---
> Source: [ubiquito/shoshin](https://github.com/ubiquito/shoshin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
