## whoamiiiport

> This portfolio is mobile-first for now.

# Portfolio Working Rules

## Current Product Focus

This portfolio is mobile-first for now.

Until Quentin explicitly says the mobile version is finished, all design,
layout, visual polish, screenshot review, and UX validation should prioritize
the mobile experience over desktop.

## Mobile-First Validation

- Default screenshot and visual QA viewport: mobile, around `390x844` or
  `390x1200`.
- When checking the live site, inspect `http://localhost:3000/` in a mobile
  viewport first.
- Validate the first mobile viewport before judging lower sections.
- Check text wrapping, header alignment, touch target size, image framing,
  scroll feel, and visual hierarchy on mobile.
- Desktop screenshots are optional regression checks unless Quentin asks for
  desktop work directly.

## Desktop Scope

- Do not redesign or optimize desktop-first right now.
- If a mobile change affects desktop, keep desktop from obviously breaking, but
  do not spend time polishing desktop unless asked.
- Avoid making choices that only look good on wide screens if they weaken the
  mobile version.

## Design Direction

- Preserve the psychedelic artist portfolio identity.
- Keep changes small, visible, and easy to review.
- Prefer real browser/mobile screenshots over guessing from code.
- If there is a conflict between desktop aesthetics and mobile clarity, choose
  mobile clarity for now.

## UI/UX Skill Requirement

- For any design, UI, UX, layout, visual polish, motion, typography, color,
  spacing, or frontend presentation work, use the Codex skill
  `design-taste-frontend` from `/Users/quentinthiessen/.codex/skills/taste-skill`.
- Apply the Taste Skill methods as a design-quality layer, but keep this
  portfolio's project rules higher priority: mobile-first validation,
  psychedelic artist identity, small reviewable changes, and real browser
  screenshots.
- Before adding design dependencies or animation libraries suggested by the
  skill, check `package.json` first and only install new packages when the
  change truly needs them.

---
> Source: [whoamaiii/whoamiiiport](https://github.com/whoamaiii/whoamiiiport) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
