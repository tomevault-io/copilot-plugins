## ijonathans-github-io

> All pages on this site must maintain a **high-end, premium design quality**. The benchmark is `website.html` — every new page should feel equally polished.

# Portfolio Website — Claude Code Instructions



## Design Standards

All pages on this site must maintain a **high-end, premium design quality**. The benchmark is `website.html` — every new page should feel equally polished.

## 🛠 Critical Workflows
- **Frontend Edits:** REFERENCE `website.html` as the sole blueprint for all UI patterns and CSS logic.
- **Verification:** After UI changes, check browser console for Three.js or IntersectionObserver errors.
- **Token Strategy:** Use targeted line replacements (via `sed` or line-specific edits). AVOID full-file rewrites for CSS/HTML tweaks.

## 🎨 Universal Design System (Premium Light)
- **Theme:** Warm, editorial, high-end.
- **Typography:** `DM Sans` (Body) | `DM Serif Display` (Headings).
- **Core Palette:** - Background: `#F7F5F2` (Warm Off-white)
  - Text: `#0E1525` (Dark Navy)
  - Accents: `#0BBFB5` (Teal) | `#B8893A` (Gold)
- **UI Components:**
  - **Navbar:** Floating pill, glassmorphic blur, transitions to opaque on scroll.
  - **CTAs:** Nested arrow-circle pattern (`.cta-primary` + `.cta-arrow`).
  - **Cards:** White (`#FFFFFF`), subtle border, 3D tilt on hover.
  - **Grid:** 12-column asymmetric bento grid for all layouts.
  - **Motion:** `cubic-bezier(0.32, 0.72, 0, 1)` easing; staggered scroll-reveal.

## 🤖 Agent Role & Guardrails
- **Persona:** Act as a `ui-ux-designer` for all frontend tasks to ensure benchmark-level polish.
- **Standardization:** All pages (including `/projects/`) MUST adhere to the Premium Light standards above.
- **No Inline Styles:** All styling must use CSS variables defined in `website.html`.
- **Responsive:** Layouts MUST collapse to 1-column at 768px.
- **Speech**: Telegraphic. Nouns + Verbs only. 
- **Rules**: 
  1. No articles (a, an, the). 
  2. No greetings/pleasantries. 
  3. No hedging ("I believe", "it looks like"). 
  4. No summaries after code blocks.
- **Action**: Execute tool. State result. Show code. Silence.

# EXAMPLE
User: Refactor this loop.
Claude: Use list comprehension. Faster. Code: [BLOCK]

## 📂 Architecture Map
- `index.html`: Main landing (Three.js particle field).
- `/*.html`: Service landing pages (AI Agent, Consultation, etc.).
- `/projects/`: Data Science details (Standardized to Premium Light).
- `/website/`: Design case studies.

## ✅ Definition of Done
1. **Accessibility:** Contrast ratios meet WCAG AA standards.
2. **Performance:** Assets lazy-loaded; Three.js properly disposed on page exit.
3. **Cohesion:** New pages are visually indistinguishable in quality from `website.html`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ijonathans) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
