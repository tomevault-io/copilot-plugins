## cartly

> UI and styling rules for Cartly

- Use DaisyUI components/classes for consistency.
- Keep admin UI layouts under core/admin and use existing layouts/partials.
- Keep form fields labeled with matching for/id and include CSRF tokens.
- Do not nest `<form>` inside another `<form>`; use sibling forms for separate submit actions (e.g. Save settings vs Verify domain).
- Prefer simple, readable layouts; avoid excessive nesting.
- Use Alpine.js (x-data, x-show, x-model) for conditional UI where the project already uses it; keep behavior server-agnostic where possible.
- Maintain hover styles on tables and consistent spacing utilities.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SanmishaTech) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
