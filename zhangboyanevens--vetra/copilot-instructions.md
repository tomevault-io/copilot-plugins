## md3-hig-design

> MD3 + Apple HIG hybrid UI baseline (Google/Meta/Apple frontend standards)


# MD3 + Apple HIG Design Baseline

Default for all frontend UI work unless the user explicitly overrides a parameter or replaces the whole system.

## Tokens

Use CSS variables from `src/styles/tokens.css` — never hardcode colors/spacing when a token exists.

## Typography

- Font stack: `system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif` + `Noto Sans SC`, `PingFang SC`, `Microsoft YaHei`
- Weights: 400 / 500 / 700 only
- Scale: 12 / 14 / 16 / 20 / 24 / 28 / 32px; ≤6 sizes per page
- Body: 16px desktop, 17px mobile; `line-height: 1.5`; `html { font-size: 16px }`
- Code: `Consolas, Menlo`

## Color (WCAG AA)

- Text: `#202124`; secondary `#5f6368`; no pure `#000` body text
- Primary: `#1a73e8`; semantic success/warning/error only
- ≤8 total colors; lowercase hex

## Spacing & layout

- 8px grid: 4 / 8 / 16 / 24 / 32 / 48px
- Touch targets ≥48×48px; icons 24px
- Prose `max-width: 65ch`; page padding ≥24px desktop / ≥16px mobile

## Shape

- Border radius: 4px or 8px only (two tiers)
- Single-layer soft shadow; no glassmorphism or stacked shadows

## CSS rules

- Lowercase selectors/properties; ≤3 nesting levels; avoid `!important`
- Ant Design theme synced via `src/theme/antdTheme.ts`

## Override priority

1. User specifies one parameter → change only that
2. User replaces whole design system → follow new system
3. No instruction → enforce this baseline

---
> Source: [ZhangBoyanEvens/Vetra](https://github.com/ZhangBoyanEvens/Vetra) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
