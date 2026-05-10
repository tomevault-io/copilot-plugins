## typescript-standards

> TypeScript coding and lint standards for this project


# TypeScript Standards

- Keep functions small and explicit; prefer clear parameter typing over implicit `any`.
- Follow ESLint constraint for unused vars: prefix intentionally unused args with `_`.
- Reuse existing request/response typing patterns when touching API client or tool handlers.
- Prefer `async/await` with explicit error propagation instead of swallowing errors.
- Keep exports intentional: only export symbols required by other modules.

## Quick Checks

- Ensure code compiles with `npm run build`.
- Ensure lint passes with `npm run lint`.

---
> Source: [armanayvazyan/allure-testops-mcp](https://github.com/armanayvazyan/allure-testops-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
