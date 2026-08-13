## core

> Finanzamente core — stack, locale, naming, dev environment


# Finanzamente Core

## Output style
Default: **caveman full** (skill `caveman`). Keep full technical accuracy; drop fluff only. Off: `stop caveman` | `normal mode`. Levels: `/caveman lite|full|ultra`. Security or irreversible actions: clear standard warning first, then resume caveman.

## Project
Italian personal finance webapp (ages 18–45). Mobile-first, responsive. All user-facing text: **Italian**. Code, DB, API names: **English**.

## Stack
| Layer | Tech |
|-------|------|
| Infra | Docker + Nginx (Apache only if required). Commands: **`make *` only** (UID/GID) |
| Backend | Laravel web routes, session auth, policies, Form Requests, Service layer |
| Dashboard | Inertia + React + TypeScript — **no Blade** |
| Public | Blade + Tailwind, SEO/SSR |
| DB | MySQL, Laravel migrations, scalable `snake_case` schema |
| CSS | Tailwind only (custom CSS rarely, with reason). React: `clsx`, `className` prop on components |

No REST API unless explicitly requested. External integrations only when asked.

## Naming
`camelCase` vars/functions · `PascalCase` classes · `snake_case` DB · `kebab-case` routes/files · `is*` booleans · acronyms uppercase in identifiers.

## Locale
`lang/it/validation.php`. Dates `dd/mm/yyyy`, `,` decimals, `.` thousands. EUR default. No multi-language i18n.

## Domain
- User selects active household.
- Registration emails unverified until validated.
- Reuse existing code before adding. Household: `isDebtBalancingMode()`, `isSharedWalletMode()` on `app/Models/Household.php`.

## Quality
DRY/KISS, WCAG 2.1, valid HTML, minimal comments, prefer mature libraries.

## On-demand docs
`docs/agent/makefile.md` · `docs/agent/e2e-conventions.md` · `docs/agent/architecture.md` · `AGENTS.md` (Cloud Agent)

---
> Source: [mnossa/finanzamente](https://github.com/mnossa/finanzamente) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
