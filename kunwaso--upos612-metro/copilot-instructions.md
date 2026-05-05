## blade-refactor-clean-architecture

> Refactor Blade files with heavy @php logic into clean architecture (Controller/Util or composer/minimal Blade)


# Blade Refactor — Clean Architecture

When refactoring a Laravel Blade file that contains heavy PHP logic inside `@php` blocks, use **one** of these approaches (choose the most professional and scalable):

- **Proper Controller structure** — prepare all data in the controller and pass to the view.
- **Controller + Util/view composer** — move shaping logic into repo-standard Util/presenter logic or a view composer, then pass prepared data to the view.
- **Minimal Blade-only** — only the cleanest possible Blade with no data prep (data still prepared before render).

---

## Hard Rules (Must Follow)

1. **No business logic in Blade** — Blade may only contain presentation logic.
2. **No data normalization in views** — no transforming or assembling data structures in the view.
3. **No config assembly in views** — all data preparation happens before rendering (Controller, Util/presenter logic, ViewModel when already established, or view composer).
4. **Routes** — do not repeatedly generate routes inside Blade; pass them from Controller or prepared view data.
5. **Translations** — centralize; avoid large translation arrays inside Blade; prepare before view rendering.
6. **Blade stays clean** — display variables only; use `@json()` only for passing **prepared** config.
7. **Laravel best practices** — separation of concerns, Single Responsibility, clean structure, type safety where possible (Laravel 9+).

---

## Expected Output When Refactoring

1. **Controller refactor** — show full controller method(s) and the updated Blade.
2. **Prepared-data option** — show the controller/composer/Util shaping logic and the simplified Blade.
3. **Minimal Blade** — show the cleanest possible Blade (no large `@php` blocks, no mixed data prep and UI).

---

## Do NOT

- Reinsert the large `@php` block.
- Mix data preparation and UI in the same file.
- Return partial refactors or leave placeholder comments like `// TODO` or `// ...`.

---

## Agent Enforcement

When generating code for a Blade refactor:

- Validate against the Hard Rules; self-correct if any rule is violated.
- Prefer **maintainability** over compactness.
- Prefer **explicit structure** over inline logic.
- Assume: Laravel 9+, standard MVC, routes already defined; `$composition`, `$componentCatalog`, `$fabric` (or equivalent) come from prepared backend data.

---
> Source: [kunwaso/upos612-metro](https://github.com/kunwaso/upos612-metro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->
