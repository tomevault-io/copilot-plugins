## harmonext

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm install          # Install dependencies
npm run dev          # Start dev server (http://localhost:5173)
npm run build        # Production build → /dist
npm run preview      # Preview production build locally
```

There are no test or lint scripts configured yet.

## Tech Stack

- **React 18** + **Vite 5** (no CRA)
- **React Router DOM v6** — client-side routing
- **CSS Modules** — all component styles are scoped (`.module.css`)
- **No external UI libraries** — custom CSS only
- **Google Fonts** — Playfair Display (headings) + DM Sans (body), loaded via `<link>` in `index.html`

## Architecture

```
src/
  App.jsx                    # BrowserRouter + AnimatedRoutes (keyed page transitions)
  main.jsx                   # ReactDOM.createRoot entry
  styles/
    variables.css            # All CSS custom properties (colors, spacing, fonts, etc.)
    global.css               # Reset, body defaults, .container, .gold-bar, .section-label utilities
  components/
    Navbar/                  # Fixed nav, transparent→opaque on scroll, mobile drawer
    Footer/                  # Dark bg footer with logo, nav links, social icons
    AnimatedText/            # Phrase-cycling component using key remount to retrigger CSS animation
    SectionReveal/           # IntersectionObserver wrapper that adds .visible class on scroll entry
  pages/
    Home/                    # Video hero + services grid + methodology snapshot + industries + CTA
    About/                   # Mission/Vision + narrative + core values + leadership cards + org diagram
    Services/                # 6 service blocks (problem / solution / outcomes / callout)
    Methodology/             # 6-phase vertical timeline + SAFe principles callout
    Portfolio/               # Fabel featured case study + 2 placeholder case studies (before/after)
    Industries/              # 5 industry sections (icon + description + pain points card)
    Contact/                 # Intake form with client-side validation + contact info + next-steps
```

### Key patterns

**Page transitions** — `AnimatedRoutes` in `App.jsx` keys a wrapper div on `location.pathname`, causing remount which re-plays the `pageEnter` CSS animation in `global.css`.

**Scroll animations** — Wrap any section in `<SectionReveal delay={ms}>` to get a fade-up on scroll entry. Uses IntersectionObserver; animates once per mount.

**AnimatedText** — Accepts a `phrases` string array and `interval` (ms). Cycles by updating state index; the `key={index}` on the span forces React remount which retriggers the `phraseEnter` keyframe animation in `AnimatedText.module.css`.

**Navbar transparency** — Transparent over the hero (white text), switches to `--color-bg` background with shadow after 56px scroll. Controlled by `scrolled` state in `Navbar.jsx`.

**CSS custom properties** — All brand colors, spacing tokens, and fonts live in `variables.css`. Reference them with `var(--color-gold)`, `var(--color-bg)`, etc. Do not hardcode brand values.

## Brand tokens

| Token | Value | Usage |
|---|---|---|
| `--color-bg` | `#FAF9F6` | Page background |
| `--color-gold` | `#BBA14F` | CTAs, icons, active states, accents |
| `--color-surface` | `#F0EEE9` | Card / section backgrounds |
| `--color-text` | `#000000` | Body text |
| `--font-heading` | Playfair Display, serif | All h1–h6 |
| `--font-body` | DM Sans, sans-serif | All body copy |

## Content TODOs

The following require real content before publishing:

- **Hero video** — add stock footage to `public/videos/hero-bg.mp4` (see comment in `Home.jsx`)
- **Logo image** — place logo file in `public/assets/logo.png` and swap the text logo in `Navbar.jsx` and `Footer.jsx`
- **Team headshots** — add photos in `public/assets/team/` and replace the initial avatar in `About.jsx`
- **Contact form backend** — wire up `handleSubmit` in `Contact.jsx` to Formspree, EmailJS, or a custom API
- **Real contact details** — update email, LinkedIn URL, and location in `Footer.jsx` and `Contact.jsx`
- **Case study details** — replace placeholder metrics and client names in `Portfolio.jsx` with approved real data
- **Government/Nonprofit case studies** — replace placeholder client names in `Portfolio.jsx` with approved names

---
> Source: [daniyellaharmonext/harmonext](https://github.com/daniyellaharmonext/harmonext) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
