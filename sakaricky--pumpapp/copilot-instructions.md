## lint-and-checks

> Run lints and formatting checks after coding; fix any issues before finishing.


# Lint and checks after coding

- **After you finish implementing or editing code**, always run the project’s lint and format checks and fix any reported issues.
- From the repo root, run:
  - `pnpm lint` — runs ESLint and TypeScript checks in all workspace packages (api, web, shared).
  - `pnpm format` — runs Prettier and formats the codebase (or `pnpm format:check` to only verify).
- Fix any lint errors and apply formatting so the branch stays clean and the build stays green.
- Do not consider the task complete until these commands succeed and any new issues introduced by your changes are resolved.

---
> Source: [SakaRicky/PumpApp](https://github.com/SakaRicky/PumpApp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
