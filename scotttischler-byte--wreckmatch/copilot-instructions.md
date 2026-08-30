## cursor-secret-sauce

> GEO / AI visibility playbook — robots, JSON-LD, article DOM, IndexNow, content rules


# Cursor Secret Sauce (GEO)

Before shipping content, SEO, or blog changes, read **`docs/CURSOR_SECRET_SAUCE.md`**.

**Secrets:** `injuredhelp.ai/.secrets-setup` has `CRON_SECRET` (and use `INDEXNOW_KEY` from docs/Vercel). Scripts auto-load it — never ask the user to re-paste. Run `python3 scripts/geo_automation.py` without prompting.

## Non-negotiables

1. **Never block** GPTBot, ClaudeBot, PerplexityBot, Google-Extended — see `app/robots.ts`.
2. Every blog/pillar page: **Article + FAQPage** JSON-LD, semantic `<article>`, `<details>` FAQs, **≥1 table**, **2000+ words**.
3. Use **brand-aware** copy/schema (`brandFromHeaders`, `BRAND_CONFIG`) — not hardcoded WreckMatch on SemiTruckMatch.
4. On publish: append slug to the correct `indexnow_pending.json` (wreckmatch vs `sites/semitruckmatch/...`).
5. Multi-domain IndexNow runs via `config/geo-sites.json`, `scripts/geo_automation.py`, and Vercel `/api/geo/cron` — do not hand-ping URLs.
6. Pillar FAQ schema is automatic via `GeoAutoFaqInjector` in root layout — add new paths in `lib/geo/pillar-faqs.ts` only.

## Quick reference

- Visual playbook: `public/secret-sauce.html`
- GEO score: `lib/geo-score.ts` → `calculateGeoScore()`
- FAQ block for landings: `components/seo/PageFaqBlock.tsx`

---
> Source: [scotttischler-byte/wreckmatch](https://github.com/scotttischler-byte/wreckmatch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-29 -->
