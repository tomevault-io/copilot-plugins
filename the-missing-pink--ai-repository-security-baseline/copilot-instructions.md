## 04-accessibility

> Accessibility standards for UI components — WCAG 2.1 AA compliance.


# Accessibility Standards (WCAG 2.1 AA)

## Semantic HTML

- Use semantic elements (nav, main, article, section, button, etc.).
- Never use div or span for interactive elements.
- Ensure correct heading hierarchy (h1 through h6 in order).

## ARIA & Screen Readers

- Add ARIA labels and roles where semantic HTML is insufficient.
- All images must have meaningful alt attributes (or alt="" for decorative).
- Form inputs must have associated label elements.
- Dynamic content must use ARIA live regions.

## Keyboard Navigation

- All interactive elements must be reachable via keyboard.
- Maintain a logical tab order.
- Provide visible focus indicators.
- Support Escape to close modals.

## Visual Design

- Minimum 4.5:1 color contrast for normal text, 3:1 for large text.
- Never convey information through color alone.
- Use relative units (rem, em) for text, not px.
- Support prefers-reduced-motion and prefers-color-scheme.

---
> Source: [the-missing-pink/ai-repository-security-baseline](https://github.com/the-missing-pink/ai-repository-security-baseline) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-11 -->
