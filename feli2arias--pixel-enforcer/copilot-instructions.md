## pixel-enforcer

> Use when implementing any UI component, styling, or CSS to prevent hardcoded values and enforce design token usage.


# Pixel Enforcer

**Core rule:** no hardcoded values in UI code. Every visual property references a token or variable.

---

## What's Banned

**Colors:** No hex (`#3b82f6`), no rgb/hsl, no named colors in component files

**Spacing:** No arbitrary px/rem values. Use the 4-point scale: `4, 8, 12, 16, 24, 32, 48, 64px`

**Typography:** No hardcoded font sizes or weights. Reference the type scale.

**Shadows, radii, transitions:** Reference tokens only.

**Inline styles:** No `style={{ color: '#fff' }}` — ever.

## What's Required

- All values reference CSS variables or Tailwind config tokens
- Dark mode variables defined alongside light mode
- New visual properties → add to token file first, then reference

## The Check

Before finalizing any UI code:
- Search for `#`, `rgb(`, `hsl(` in component files → fix any found
- Search for `style={{` with hardcoded values → replace with tokens
- Confirm all new colors exist in the token file

---

## Anti-Patterns

| Pattern | Why It's Harmful |
|---------|----------------|
| `color: #3b82f6` in component CSS | Breaks theming, can't be changed globally |
| `margin: 13px` | Magic number, breaks the spacing system |
| `style={{ fontSize: '14px' }}` | Bypasses the type scale |
| Hardcoded dark mode values | Dark mode becomes inconsistent |

---

## What Stays the Same

Visual quality and design precision are never compromised.

---

## Manual Activation

Invoke with `/pixel-enforcer` before any UI implementation session.

---
> Source: [Feli2arias/pixel-enforcer](https://github.com/Feli2arias/pixel-enforcer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
