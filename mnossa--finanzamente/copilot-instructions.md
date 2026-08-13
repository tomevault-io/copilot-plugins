## blade-public

> Public Blade pages — SSR, SEO, Tailwind


# Blade (Public / SSR)

- Public and SSR pages only — not the authenticated Inertia dashboard.
- Files: `kebab-case.blade.php`. Layouts + Blade components (header, footer, alerts).
- Tailwind for styling (match dashboard visual language).
- No business logic in views; use controllers/view models.
- Copy and validation messages: Italian via Laravel lang.
- WCAG 2.1, SEO, performance. Valid HTML.
- Minimal Blade comments; no dead code.

After Blade with behavior impact: full verification gates in `agent-workflow`.

---
> Source: [mnossa/finanzamente](https://github.com/mnossa/finanzamente) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
