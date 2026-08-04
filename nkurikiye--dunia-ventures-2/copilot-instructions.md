## dunia-ventures-2

> Context for anyone (incl. Claude Code) working on this project.

# CLAUDE.md — Dunia Ventures website

Context for anyone (incl. Claude Code) working on this project.

## What this is
The marketing website for **Dunia Ventures** — an early-stage fund backing tech-enabled
creative businesses across Africa (film, music, fashion-tech, gaming, animation, creative-tech).
It is a **static, multi-page site**: hand-written HTML/CSS/JS with **no build step**.
The only runtime dependency is React + Babel loaded from a CDN, used *solely* to render the
optional in-page "Tweaks" panel — the site content itself is plain HTML.

## Pages
| File | Purpose |
|------|---------|
| `index.html` | Home — hero, sector marquee, proof ("Proven Models"), thesis, opportunity, investment-themes teaser, team teaser, newsletter |
| `Portfolio.html` | **Investment Themes** — six sectors as dark cards (nav label "Themes") |
| `Approach.html` | Investment Approach (ticket size/stage/geography + "Missing Middle"), the model, and the "How We Invest & Create Value" timeline |
| `Team.html` | The four partners with bios |
| `About.html` | Company hero, thesis story, opportunity metrics |

Nav across all pages: **Home · Themes · Approach · Team · About**.

## Shared files
- `dunia.css` — the whole design system: CSS custom properties (`:root`) for color/type/spacing,
  plus all component classes (`.nav`, `.btn`, `.eyebrow`, `.display`, `.theme-card`, `.timeline`, etc.).
- `dunia.js` — behaviour: palette/font/grain persistence (localStorage), mobile nav, **scroll reveal**
  (`.reveal` → `.in`, rect-based so it works in any render context), marquee duplication, newsletter form.
- `tweaks-panel.jsx` — reusable Tweaks shell (starter component; host protocol + form controls).
- `dunia-tweaks.jsx` — mounts the Tweaks panel; lets the user switch palette / display font / paper grain.
- `assets/` — all images (team headshots, proof-point photos, hero image).

## Design system
- **Type:** display `Bricolage Grotesque`; accent italics `Instrument Serif`; body `Hanken Grotesk`;
  labels/numbers `Space Mono`. Loaded via Google Fonts `@import` at the top of `dunia.css`.
- **Color:** warm cream paper + warm near-black ink, with ONE swappable accent.
  Default accent is **plum `#6C3FB6`** (matches the logo). Alternates (vermillion / marigold / jade)
  live in `window.DUNIA_PALETTES` (`dunia.js`) and the Tweaks panel.
- **Accent is themeable at runtime** via `--accent` / `--on-accent` / `--accent-deep` on `:root`,
  set by `duniaApplyTheme()` and persisted to `localStorage` (`dunia.theme`). A tiny inline script in
  each page's `<head>` re-applies the saved theme before paint to avoid a flash.
- **Logo:** the three-circle mark is inline SVG (purple), in every nav and footer — no image file.

## Conventions (please keep)
- **Canonical HTML:** close every element, double-quote attributes, no self-closing non-void tags.
- Sections carry `data-screen-label="Page / Section"` and numbered mono eyebrows, e.g. `(01)`.
- Entrance animation = add class `reveal` (+ optional `d1`–`d6` stagger); `dunia.js` adds `.in` when scrolled into view. The hidden state is gated behind `html.js` so content is never stuck invisible if JS fails.
- Use existing tokens/classes before inventing new ones. Define new colors with `color-mix(... var(--accent) ...)` rather than hard-coding.
- Keep all paths **relative**; keep images in `assets/`.

## Run / preview
No build. Open `index.html` directly, or serve the folder:
```bash
python3 -m http.server 8000   # then visit http://localhost:8000
```

## Deploy
Any static host. See `README.md` for GitHub Pages and Netlify steps. The published site needs
internet access (React/Babel + Google Fonts come from CDNs). No server or database.

## Contact / brand
Email: `info@wajenzi.fund` · Footprint: Côte d'Ivoire · Algeria · Luxembourg.

---
> Source: [nkurikiye/Dunia-Ventures-2](https://github.com/nkurikiye/Dunia-Ventures-2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-30 -->
