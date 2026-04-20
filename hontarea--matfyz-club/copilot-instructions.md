## matfyz-club

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the Project

No build tools or package manager — open `index.html` directly in a browser, or serve locally:

```bash
python3 -m http.server 8080
# then open http://localhost:8080
```

Deploy by pushing to GitHub Pages (works as-is, no build step needed).

## Architecture

Single-page static site. Three files do all the work:

- **`index.html`** — full page markup with all sections inline (Hero, About, Principles, Focus Areas, Structure, Join, Footer)
- **`css/style.css`** — all styles; uses CSS custom properties defined in `:root` for the design system
- **`js/main.js`** — five independent modules (Lucide icon init, tsParticles hero animation, navbar scroll effect, hamburger menu, IntersectionObserver fade-ins)

External CDN dependencies loaded in `index.html`:
- **tsParticles** (`tsparticles.bundle.min.js`) — animated particle background in the hero `#vanta-bg` div
- **Lucide** (`lucide.min.js`) — icon library; icons declared as `<i data-lucide="icon-name">` and activated via `lucide.createIcons()` on `DOMContentLoaded`
- **Google Fonts** — Satoshi (display), JetBrains Mono (mono), DM Sans (body)

## Design System

CSS variables in `:root` (`css/style.css:35`):

| Variable | Value | Use |
|---|---|---|
| `--bg` | `#011627` | Hero background |
| `--section-bg` | `#020c16` | Page/footer background |
| `--card-bg` | `#0d2540` | Cards, pillars, section containers |
| `--accent-teal` | `#2ec4b6` | Primary accent |
| `--accent-green` | `#20bf55` | Secondary accent |
| `--gradient` | teal → green | Buttons, highlights |
| `--font-display` | Satoshi | Headings, titles |
| `--font-mono` | JetBrains Mono | Labels, code-style accents |
| `--font-body` | DM Sans | Body text |

Section containers use `var(--card-bg)` background applied to `.container` children (not the `<section>` itself), keeping sections transparent with the dark `--section-bg` as the page background.

## Key Patterns

**Scroll fade-ins**: Add class `fade-in` to any element. `IntersectionObserver` in `js/main.js` adds `visible` when it enters viewport. CSS handles the opacity/translateY transition (`css/style.css:77`).

**Card variants**: `.card` for standard cards; `.pillar--teal` / `.pillar--green` for the two-pillar structure section; `.card-icon--teal` / `.card-icon--green` for colored icon backgrounds.

**Responsive breakpoints**: `768px` (tablet — stacks all grids to 1 column, shows hamburger) and `480px` (mobile — tighter padding).

**Hero animation graceful degradation**: tsParticles is wrapped in `if (window.tsParticles)` — if CDN fails, the hero shows the solid `#011627` background defined directly on `#hero` in CSS.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hontarea) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
