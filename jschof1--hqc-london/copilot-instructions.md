## hqc-london

> - Use non-www HTTPS as the canonical host for this site: `https://highqualityclean.co.uk`.

## Learned User Preferences

- Use non-www HTTPS as the canonical host for this site: `https://highqualityclean.co.uk`.
- Keep discount form URL params and internal values compatible as `home` / `commercial`, but send webhook payload `type` values as `Residential` / `Commercial`.

## Learned Workspace Facts

- This Astro 5 site deploys to Cloudflare Pages with `@astrojs/cloudflare`, `output: 'server'`, `pages_build_output_dir = "dist"`, and GitHub pushes to the connected branch auto-deploy.
- Canonical URLs should be lowercase, trailing-slash, non-www HTTPS URLs, with duplicate legacy area and service paths redirected directly to final canonical URLs where possible.
- Public phone details should show the GBP-aligned landline `0208 870 3925` / `+44208703925`; calls forward to `07427 925266`, but the mobile should not be displayed publicly.
- The discount form posts to `/api/discount`, which reads the Cloudflare Pages secret `DISCOUNT_FORM_WEBHOOK`; changing that secret requires a fresh Pages deployment before production uses the new value.
- Location landing pages are prerendered static HTML under `/locations/london/[area]/` and `/locations/surrey/[area]/`; the sitemap injection script should skip URLs already emitted by Astro sitemap to avoid duplicates.

---
> Source: [jschof1/hqc-london](https://github.com/jschof1/hqc-london) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-15 -->
