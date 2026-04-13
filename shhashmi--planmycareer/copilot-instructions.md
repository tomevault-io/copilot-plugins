## planmycareer

> - When planning any change that involves UX modifications, **always show the complete page layout** in ASCII mockups or markdown previews — never truncate, abbreviate, or hide sections with `...` or similar placeholders.

# PlanMyCareer — Project Instructions

## UX Planning Rules
- When planning any change that involves UX modifications, **always show the complete page layout** in ASCII mockups or markdown previews — never truncate, abbreviate, or hide sections with `...` or similar placeholders.
- Show every visible element on the page (header, nav, content sections, buttons, footer, etc.) so the full user experience is clear at a glance.
- If a page is too long for a single preview, split it into clearly labeled sections (e.g., "Above the fold", "Below the fold") but still show everything.

## Design System
- **Styling:** CSS custom properties (no Tailwind). Use `var(--token)` — never hardcode colors or spacing.
- **Color tokens:** `--primary` `--primary-dark` `--primary-light` `--secondary` `--accent` `--background` `--surface` `--surface-light` `--text-primary` `--text-secondary` `--text-muted` `--border` `--success` `--warning` `--error` `--gradient-1` `--gradient-2`
- **Component styles:** Inline style objects (not className for layout). Use `className` only for utility classes defined in `index.css`.
- **Animations:** Framer Motion only. All animations ≤500ms. Always respect `prefers-reduced-motion`.
- **Icons:** Lucide React only (`lucide-react`). Size matches context: 14px/sm, 16px/md, 18px/lg.
- **UI primitives:** Reuse `src/components/ui/` — `Button`, `Card`, `Modal`, `FormField`, `RichSelectField`, `LoadingSpinner`, `ErrorAlert`, `StatusBadge`, etc. before writing new ones.
- **Border radius:** 12px for interactive elements (buttons, cards). 8px for inputs.

## UI Development Workflow
- **After every component change:** open `localhost:5000` in Chrome, navigate to the changed page, verify visually, check the browser console for errors.
- **Test all interactive states:** default, hover, focus, active, disabled, loading, error, empty, success.
- **Responsive check:** verify layout at mobile (375px) and desktop (1280px) viewports.
- **Chrome launch:** `claude --chrome` enables browser tool — use it for all UI verification.

## Accessibility Standards
- Use semantic HTML: `<button>` not `<div onClick>`, `<label>` for all inputs, `<nav>` `<main>` `<header>` `<footer>` landmarks.
- All icon-only buttons must have `aria-label`.
- Form inputs must have visible labels or `aria-label` + error state + success feedback.
- Tab order must match visual order. Focus outline must always be visible.
- Minimum touch target: 44×44px.
- Color contrast: 4.5:1 for body text, 3:1 for large text.

## Component Standards
- Functional components + TypeScript only. No class components, no implicit `any`.
- Props interface declared explicitly above the component.
- Cover all states in the component: loading, error, empty, success — never leave a state unhandled.
- Controlled form inputs only (no uncontrolled). Validate on blur + on submit.
- Never duplicate UI logic — check `src/components/ui/` and `src/components/` before writing new components.

## Chrome Browser Verification (Item #1)
- Start UI sessions with `claude --chrome`.
- After any component edit, ask Claude to: "Open localhost:5000, navigate to [page], screenshot the component, check the console for errors, test on mobile viewport (375px)."
- Use Chrome to verify: correct rendering, animations, form validation, error states, responsive layout.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/shhashmi)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/shhashmi)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
