## n-tech-website

> Static HTML/CSS/JS website for N-Tech Energy Solutions LLC, a solar installation company serving North Texas. Deployed via Netlify at `https://n-tech-es.com` and `https://www.n-tech-es.com`.

# N-Tech Energy Solutions — Claude Code Instructions

## Project Overview
Static HTML/CSS/JS website for N-Tech Energy Solutions LLC, a solar installation company serving North Texas. Deployed via Netlify at `https://n-tech-es.com` and `https://www.n-tech-es.com`.

## Business Context
- **Company**: N-Tech Energy Solutions LLC
- **Location**: Based in North Texas; installer partner operates out of Chico, TX (Wise County)
- **Service area**: 60-mile radius of Chico, TX — Wise, Parker, Jack, Montague Counties
- **Contact**: customerexperience@n-tech-es.com | (214) 267-9372
- **Tally forms**: Consultation → `https://tally.so/r/3jdYeQ` | Founding offer → `https://tally.so/r/zxXr4E`
- **Pricing tiers**: Starter $2.40/watt | Standard $2.50/watt | Premium $2.70/watt | Power Plus $2.90/watt
- **Founded**: 2025/2026 — still building first 25 founding customers

## Brand Voice
- **Do not push solar** — always present it as an option, not a prescription
- Say "Solar can be a long-term answer" not "Solar is a long-term answer"
- Transparent, no-pressure, no-commission, no door-to-door salespeople
- Avoid aggressive urgency language; authentic scarcity only
- Do not claim to be "based in Chico" or "based in Wise County" specifically — say "based in North Texas, serving a 60-mile radius of Chico, Texas"

## File Structure
- 28 `.html` files in root — one per page/city/blog post
- `images/` — all images are `.webp` format (converted from originals)
- `logo.jpg` — used as brand icon in popups and success states
- `images/logo.webp` — used in nav header
- `images/hero.webp` — main hero image, preloaded on index.html

## When Editing HTML Files

### Always use WebP images
All `<img>` src and CSS `background-image` must reference `.webp` files, not `.jpg` or `.png`.

### Lazy loading
Add `loading="lazy"` to all `<img>` tags that are below the fold. Do NOT add it to hero/above-fold images.

### Google Fonts
Use the non-blocking pattern — never a plain `<link rel="stylesheet">`:
```html
<link rel="preload" href="https://fonts.googleapis.com/css2?family=Archivo:wght@400;600;700;900&family=DM+Sans:wght@400;500;700&display=swap" as="style" onload="this.onload=null;this.rel='stylesheet'">
<noscript><link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Archivo:wght@400;600;700;900&family=DM+Sans:wght@400;500;700&display=swap"></noscript>
```

### Safari CSS compatibility
- Do NOT use `inset: 0` — use `top:0;left:0;right:0;bottom:0` instead
- Test overlay/modal CSS for Safari compatibility

### Google Analytics
Every page must include this exact tag in `<head>` (correct Measurement ID is `G-50SQZ12XJX`):
```html
<!-- Google tag (gtag.js) -->
<script async src="https://www.googletagmanager.com/gtag/js?id=G-50SQZ12XJX"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());
  gtag('config', 'G-50SQZ12XJX');
  gtag('config', 'AW-17959768934');
</script>
```

## Email / Forms

### Contact form (contact.html)
Uses FormSubmit AJAX endpoint with FormData (NOT JSON — JSON triggers CORS preflight that blocks Safari/iPad):
```javascript
var fd = new FormData();
fd.append('name', name);
fd.append('email', email);
fetch('https://formsubmit.co/ajax/customerexperience@n-tech-es.com', {
  method: 'POST',
  body: fd
})
```

### Popup email capture
- Present on 19 pages (index + city pages + blog posts)
- Uses same FormData pattern as contact form
- SessionStorage key `ntec_popup_seen` — shows once per browser tab session
- Triggers after 5-second delay using `setTimeout(showPopup, 5000)`
- Script must be wrapped in `window.addEventListener('load', function() { ... })`
- Show overlay with `overlay.style.display = 'flex'` (NOT classList.add — specificity issues)
- Success state shows 64px circular logo.jpg, no emojis

## SEO Conventions

### Pages and their target keywords
| File | Target Keyword |
|------|---------------|
| `the-true-cost-of-solar.html` | "how much are solar panels in tx" |
| `solar-north-texas.html` | "solar companies", "solar installation company" |
| `pricing.html` | local pricing, cost FAQs |
| City pages (`solar-[city]-tx.html`) | "[city] solar" variations |

### Schema
- Use `FAQPage` JSON-LD schema on any page with FAQ content
- `LocalBusiness` schema uses `areaServed: {"@type": "City", "name": "Chico, Texas"}`
- Do not expand areaServed beyond what's already there without asking

### City Lists (consistent across all pages)
**Wise County**: Decatur, Bridgeport, Chico, Boyd, Alvord, Rhome, Newark, Paradise
**Parker County**: Weatherford, Aledo, Azle, Springtown, Willow Park
**Jack County**: Jacksboro, Perrin
**Montague County**: Bowie, Nocona, Montague

## Footer
Every page footer includes:
1. Chamber of Commerce badge (do not remove):
```html
<div style="margin-top:1.25rem;">
    <a href='https://www.chamberofcommerce.com/business-directory/texas/chico/solar-energy-company/2034199863-n-tech-energy-solutions-llc?source=memberwebsite' target='_blank' rel='noopener'>
        <img style='width:auto;height:100px;' src='https://coc.codes/images/badge/2034199863' alt='Chamber of Commerce Member - N-Tech Energy Solutions'>
    </a>
</div>
```
2. Social links: Facebook, Instagram, TikTok (SVG icons, no emoji)

## Git Workflow
- Branch for this agent: `claude/check-drive-connection-YtRgb`
- Push to `main` for live deployment via Netlify
- If push fails with 403: `git pull origin main --rebase` then retry
- Commit messages should be descriptive; include what changed and why

## Things NOT to Do
- Do not add `noindex` to any page without asking first
- Do not change the Tally form URLs without confirmation
- Do not add emojis anywhere — use SVG icons instead
- Do not use `localStorage` for popup state — use `sessionStorage`
- Do not use `Content-Type: application/json` for FormSubmit requests — causes CORS issues on iPad/Safari
- Do not reference `IMG_7359.jpeg` or any non-existent image files
- Do not claim N-Tech is "based in Chico" or "based in Wise County"
- Do not push solar as the definitive answer — always frame as an option

---
> Source: [n-tech-es/n-tech-website](https://github.com/n-tech-es/n-tech-website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
