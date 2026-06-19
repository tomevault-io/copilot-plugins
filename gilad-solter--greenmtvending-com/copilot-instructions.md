## greenmtvending-com

> Static single-page marketing website for **Green Mt Vending** (Boulder, CO), deployed via GitHub Pages on the custom domain `greenmtvending.com` (DNS managed at Cloudflare).

# Green Mt Vending — Marketing Site

Static single-page marketing website for **Green Mt Vending** (Boulder, CO), deployed via GitHub Pages on the custom domain `greenmtvending.com` (DNS managed at Cloudflare).

## Project rules
- **Single self-contained file:** `index.html` with all CSS in one `<style>` block in the `<head>`. No build step, no external CSS/JS files, no frameworks.
- **No forms.** Contact is a `mailto:` link only.
- **Content is sourced from the live site** (`greenmtvending.com`) and the original `freehealthyvending.com` was used only as a *visual/structural* reference. Do not invent business facts (services, claims, locations, contact info).
- Clean, modern, health-forward UI. Fonts: **Sora** (headings) + **Inter** (body) via Google Fonts — matching the brand's existing typography.
- Green brand palette (emerald/forest greens).

## Verified business facts (do not change without checking the live site)
- Name: Green Mt Vending · Location: Boulder, CO
- Tagline: "Your premier choice for quality, convenience, and innovation in vending services"
- Email: gilad@greenmtvending.com
- Services: install state-of-the-art credit-card-operated machines, **on-demand restocking** (NOT weekly), routine maintenance
- Focus: healthy + premium snacks and beverages; local small business, personalized service
- Target customers: public & charter schools, YMCAs, recreation & community centers, medical facilities
- Hero uses real machine photo at `images/vending-machine.png` (transparent PNG)

## Deploy
- `index.html`, `CNAME` (custom domain), and `.nojekyll` live at repo root.
- GitHub Pages serves from the default branch root. Cloudflare DNS points the apex/`www` at GitHub Pages.

---
> Source: [gilad-solter/greenmtvending.com](https://github.com/gilad-solter/greenmtvending.com) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-19 -->
