## booking-agency-design

> When editing booking UI (Boulevard wizard), apply agency-agents design personas with Rella overrides.


# Booking + agency-agents design division

Vendored personas live in `docs/agency-agents/design/` (from [agency-agents](https://github.com/msitarzewski/agency-agents/tree/main/design)). Use them as **process guidance**, not a second color system.

## Which personas apply

| Persona file | Use for booking |
|--------------|-----------------|
| `design-ux-architect.md` | Step flow, layout grid, responsive behavior, progress clarity |
| `design-ui-designer.md` | Component states, spacing rhythm, form field consistency, WCAG-minded targets |
| `design-brand-guardian.md` | Voice, hierarchy, messaging against drift |
| `design-inclusive-visuals-specialist.md` | Contrast, motion preferences, inclusive copy |
| `design-ux-researcher.md` | Optional test hypotheses post-launch |

Do **not** apply whimsy or playful-only patterns to core booking.

## Overrides (non-negotiable)

- **Source of truth**: `rella-brand.mdc` — rose/silver palette, Poppins, left alignment, no blue CTAs.
- **Ignore** upstream defaults that demand a global **light/dark theme toggle** for this site.
- **Ignore** generic blue primary buttons / Inter stacks in upstream examples; map patterns to existing Tailwind tokens in `src/app/globals.css`.
- Luxury med spa tone: confident, warm, expert — not casual or gimmicky.

## Quick QA

- Keyboard path through schedule → details → confirm.
- Focus rings visible (`:focus-visible` in globals).
- Touch targets ≥ 44px on primary choices where possible.

---
> Source: [zwagner19/rella-aesthetics](https://github.com/zwagner19/rella-aesthetics) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
