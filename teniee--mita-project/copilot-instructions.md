## mita-project

> Landing page + waitlist for **Mita Finance** — a daily budget redistribution app (iOS + Android, coming soon).

# Mita Finance — Project Guide for Claude

## What is this project
Landing page + waitlist for **Mita Finance** — a daily budget redistribution app (iOS + Android, coming soon).
The site is a **single-file** `index.html` — no framework, no build step, pure HTML/CSS/JS.

## Live site
https://mitafinance.com

## Key file
`F:/adata/MITA FINANCE/index.html` — the only file you edit for the website.

## Backend API
`https://mita-production-production.up.railway.app`
- `POST /api/waitlist/join` — join waitlist, returns `{ effective_position, total_signups, referral_link, ref_code }`

## Brand guidelines
All brand files are in `F:/adata/MITA FINANCE/brand/` (26 PNG files).

### Brand colors (EXACT — do not deviate)
| Variable     | Hex       | Role                        |
|--------------|-----------|-----------------------------|
| `--navy`     | `#193C57` | Primary text & anchor color |
| `--yellow`   | `#FFD25F` | PRIMARY accent (CTA, energy)|
| `--teal`     | `#14B8A6` | Success, savings, positive  |
| `--purple`   | `#6B73FF` | Secondary accent            |
| `--red`      | `#F44336` | Alerts, logo quadrant       |
| `--teal-l`   | `#CCFBF1` | Light teal backgrounds      |
| `--bg`       | `#FFF8F0` | Page background (warm cream)|

### Approved gradients
- Navy → Yellow: `#193C57 → #FFD25F`
- Red → Teal: `#F44336 → #14B8A6`
- Purple → Yellow: `#6B73FF → #FFD25F`

### Typography
- `'Manrope'` — PRIMARY font (UI headlines, financial figures, dashboards)
- `'Sora'` — SECONDARY font (marketing body text, section content)
- `'Space Mono'` — financial data, numbers, position counters

### Logo mark
4-quadrant grid (2×2), each quadrant a solid rounded square:
- Top-left: Red `#F44336` (M)
- Top-right: Purple `#6B73FF` (I)
- Bottom-left: Teal `#14B8A6` (T)
- Bottom-right: Yellow `#FFD25F` (A)

### Background
The site is **LIGHT** — warm cream `#FFF8F0`. NOT dark. Glass cards use white `rgba(255,255,255,0.72)` with navy borders.

### Brand personality
Confident but not loud. Structured. Calm. Forward-thinking. Never financial jargon or fear language.

## Preserved JS IDs (never rename or remove)
These IDs are wired to the waitlist API and phone engine:
- Forms: `heroFormWrap`, `heroForm`, `heroBtn`, `heroSuccess`, `ctaFormWrap`, `ctaForm`, `ctaBtn`, `ctaSuccess`
- Success data: `hPos`, `hTotal`, `hLink`, `cPos`, `cTotal`, `cLink`
- Counters: `totalCount`
- Phone mockup: `phBc`, `phCal`, `phPanel`
- UI: `nav`, `confetti`, `cur`, `curRing`, `heroSocial`

## Preserved JS globals
- `window.mitaDay(d)` — show day detail in phone
- `window.mitaBack()` — return to calendar in phone

## Key features to never break
1. **Waitlist API** — hero form + CTA form both POST to the backend
2. **Referral system** — `?ref=CODE` URL param, referral link after signup
3. **Confetti** — triggers on successful signup (`boom()`)
4. **Interactive phone** — MITA Interactive Phone Engine IIFE at bottom of `<script>`; the calendar shows March 2026, days are clickable, shows budget redistribution detail
5. **Custom cursor** — `.cur` + `.cur-ring`
6. **Scroll reveal** — `.reveal` + IntersectionObserver

## Figma integration
- Figma capture script is already in the HTML: `<script src="https://mcp.figma.com/mcp/html-to-design/capture.js" async></script>`
- To push to Figma: start `npx http-server . -p 8765 --cors -s`, then use `mcp__plugin_figma_figma__generate_figma_design`
- Latest Figma file: https://www.figma.com/design/hGPeeY99LUn1Q2MSpmHBQu

## Local dev server
```bash
cd "F:/adata/MITA FINANCE"
npx http-server . -p 8765 --cors -s
```
Then open http://localhost:8765

## Design decisions made
- Hero accent gradient: Red → Teal (brand-approved)
- CTA button: Navy background + Yellow text (Navy→Purple gradient)
- Step numbers use `Space Mono` font
- Stat numbers use `Space Mono` font
- Feature icon backgrounds: solid brand color tints (not gradients)
- Phone mockup exterior stays dark (phone frames are conventionally dark)
- Dot grid on body uses navy dots `rgba(25,60,87,0.06)` instead of white dots

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/teniee)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/teniee)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
