## mattlifetech-github-io

> - **URL:** https://mattlifetech.github.io/

# MattLifeTech Blog — Claude Context

## Site Overview
- **URL:** https://mattlifetech.github.io/
- **Stack:** Jekyll + Minimal Mistakes theme (`contrast` skin) + GitHub Pages
- **Tone:** Honest, no-BS product reviews and practical tech how-tos
- **Author:** Matthew Choo — 20 years market research (Telco/Media), Asia-based

## Affiliate Setup
Affiliate links are managed in `_data/affiliate_links.yml`.
To add a buy card to any post:
```liquid
{% include affiliate-card.html product="product_key" %}
```

### Active Affiliate Programs
| Platform | Status | Notes |
|---|---|---|
| Shopee MY | Active | Links use `sub_id: blog` for tracking |
| Lazada MY | Pending approval | Add links to YAML once approved |
| AliExpress | App created (MattLifeTech) | App Key/Secret to be wired up |

### Adding a New Product
1. Add entry to `_data/affiliate_links.yml` with: `title`, `tagline`, `image`, `shopee`, `lazada`, `note`
2. Generate Shopee affiliate link at https://affiliate.shopee.com.my/offer/custom_link
3. Add `{% include affiliate-card.html product="key" %}` before `## Support Me` in the post

## Key Files
| File | Purpose |
|---|---|
| `_config.yml` | Site config, GA tracking (G-9CQYF1JHKV), GSC verification |
| `_data/affiliate_links.yml` | All affiliate product links and metadata |
| `_includes/affiliate-card.html` | Product card component (image + title + buy buttons) |
| `assets/css/main.scss` | Custom CSS including `.aff-card` styles |
| `_posts/` | All blog posts — 72 total |

## GSC & SEO
- GSC property: `https://mattlifetech.github.io/` — verified (siteOwner)
- Sitemap: `https://mattlifetech.github.io/sitemap.xml` — submitted June 2026
- GA: `G-9CQYF1JHKV`
- Verification file: `googlee1bdd1508645a7fa.html` — do NOT delete

## Gallery Posts — Hidden
17 AI-generated image gallery posts are hidden (`published: false`).
They live in `_posts/gallery*.md` — keep them, just don't publish.

## Deployment
GitHub Pages — auto-deploys on push to `main`. Build takes ~90 seconds.
Check status: https://github.com/mattlifetech/mattlifetech.github.io/actions

---
> Source: [mattlifetech/mattlifetech.github.io](https://github.com/mattlifetech/mattlifetech.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-15 -->
