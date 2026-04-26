## mo-skills

> Always adhere to the standards defined in `d:/mo_skills/` when generating or refactoring code.

# Mo Skills Standards

Always adhere to the standards defined in `d:/mo_skills/` when generating or refactoring code.

## Python Backend (`d:/mo_skills/python_backend/SKILL.md`)
- **SOLID Principles**: Mandatory for all architecture.
- **Class Organization**: `__init__` -> Static -> Public -> Private -> Magic.
- **ABCs**: Use `Base` prefix (e.g., `BaseAgent`).
- **Type Hinting**: Use `| None` instead of `Optional`.
- **Exports**: `__all__` at the top of `__init__.py`.
- **Docstrings**: Single-line only.
- **Logging**: No `print()` statements.
- **Config**: Strict use of `.env` for secrets and `.yaml` for structural logic.
- **Stack**: FastAPI, Pydantic, Supabase, Stripe, Resend, LangChain/LangGraph, uv.

## Frontend (`d:/mo_skills/frontend/SKILL.md`)
- **Architecture**: Component-Driven / Container-Presentational Pattern.
- **Component Organization**: Imports -> Types -> Component -> Hooks -> Derived State -> Handlers -> Return.
- **Strict Typing**: No `any` types. Explicit interfaces for props/state.
- **Exports**: Named exports only (unless framework explicitly requires default).
- **Styling**: CSS Modules only.
- **Config**: `.env` for public keys, `.yaml` for app settings/i18n.
- **Localization**: STRICTLY NO hardcoded user-facing strings. Must use an i18n dictionary context.
- **Stack**: React, TypeScript, React Query, Zustand, Supabase SDK, Stripe, yarn (Mandatory).

## UX UI Pro Max (`d:/mo_skills/ux_ui_pro_max/SKILL.md`)
- **Aesthetics**: Premium, vibrant, modern. No generic defaults.
- **Depth**: Utilize Glassmorphism (backdrop-filter) and layered soft shadows.
- **Interactivity**: All components must have distinct hover, focus-visible, and active states.
- **Animations**: Use strictly `transform` and `opacity` with cubic-bezier easing for 60fps performance.
- **Typography**: Modern Sans Serif (Inter, Roboto). Strictly no #000 on #fff.

## Git & Commits (`d:/mo_skills/git_workflow/SKILL.md`)
- **Format**: `<type>[optional scope]: <description> (<EPIC_ID>)`
- **Allowed Types**: `feat`, `fix`, `build`, `ci`, `docs`, `perf`, `refactor`, `style`, `test`, `ta`, `chore`.
- **Branch Naming**: `<EPIC_ID>-<DESCRIPTIVE_NAME>` (Must stem from `Test` branch).
- **MR Rules**: One commit per branch. Always rebase & squash before merging.

---
> Source: [Mohamednajdawi/mo_skills](https://github.com/Mohamednajdawi/mo_skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-26 -->
