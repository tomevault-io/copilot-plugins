## code-style

> JoyWork API formatting, naming, imports, and typing style


# JoyWork API Code Style

- Match existing backend formatting: `single quotes`, semicolons, and `@/` alias imports inside `src`.
- Keep filenames aligned with the current backend pattern such as `<feature>.<part>.ts` and kebab-case sub-feature names like `user-profile.service.ts`.
- Prefer named exports for new backend files; keep route files consistent with existing `export async function ...Routes(...)` style.
- Use `PascalCase` for classes and interfaces, `camelCase` for functions and variables, and clear domain-oriented names over generic helpers.
- Respect the current strict TypeScript posture; avoid adding unused locals, casual `any`, or broad type escapes unless there is a justified local reason.

---
> Source: [vanchuong201/joywork-api](https://github.com/vanchuong201/joywork-api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
