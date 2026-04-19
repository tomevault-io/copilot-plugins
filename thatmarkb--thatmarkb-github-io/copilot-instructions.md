## thatmarkb-github-io

> Provide only the **critical** brand constraints (typography, colors, tone, and a few “must nots”) so Claude can explore layout and design freely while remaining on-brand.

# CLAUDE.md — TMBPG Brand Guardrails (minimal, non-prescriptive)

## Purpose
Provide only the **critical** brand constraints (typography, colors, tone, and a few “must nots”) so Claude can explore layout and design freely while remaining on-brand.

If a request conflicts with these constraints, flag it and propose the smallest compliant change.

---

## Non-negotiables

### Typography
- Use **Inter** for body/UI text.
- Use **Lato** for headlines (H1/H2/H3) and short callouts.
- Fallback stacks:
  - Body: `Inter, system-ui, -apple-system, Segoe UI, Roboto, Arial, sans-serif`
  - Headings: `Lato, system-ui, -apple-system, Segoe UI, Roboto, Arial, sans-serif`
- Line-height:
  - Body: **1.45–1.55**
  - Headings: tighter than body (avoid overly airy headlines)

### Color tokens
Use these tokens only unless explicitly instructed otherwise.

- Background (dark): `#1d1f21`
- Primary text: `#c5c8c6`
- Muted text: `#969896`
- Primary blue: `#0D4F7C`
- Secondary blue (hover/accent): `#1E7FB6`

Rules:
- Default to **dark** theme unless explicitly asked for light.
- Ensure readable contrast for text and CTAs.

### Tone + copy
- Calm, confident, high-signal.
- No hype/buzzwords; no emojis.
- Do **not** fabricate client logos, client names, or numeric outcomes.
  - Use `{TODO: ...}` placeholders if proof points are needed.

### CTA labels
- Keep CTA button labels short: **1–2 words**.

### Accessibility baseline
- Keyboard navigable form controls.
- Visible focus states.
- Responsive by default.

---

## Safe freedoms (intentionally open)
Claude may freely choose:
- Page structure (single column, split layout, cards, etc.)
- Spacing scale, grid, section count
- Background treatments (gradients, blur, noise) if subtle and consistent with tokens
- Component styles (buttons, chips, callouts), as long as they stay within tokens/fonts and remain readable

---

## Content constraints for initial v1
No case studies yet. v1 should include:
- A **hero section** that communicates:
  - 15+ years product/program leadership in Fortune 100/500 contexts
  - engineering + project management background
  - specialization in data, BI, APIs, AI enablement, DevOps
- A **contact CTA** and an embedded contact form posting to:
  - `https://formspree.io/f/xkovrzeg`
- Optional: 3–6 capability callouts (short, scannable)

---

## Implementation notes (optional)
- Keep dependencies minimal (plain HTML/CSS/JS is fine).
- Prefer semantic HTML.
- Keep file naming lowercase/underscore-delimited when adding assets.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/thatmarkb) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
