## testing

> Vitest testing; unit, nuxt, and e2e projects; mock consola when needed


# Testing

- Use Vitest. Tests live under `test/`: `test/unit/` (unit), `test/nuxt/` (Nuxt), `test/e2e/` (e2e). Config in `vitest.config.ts`.
- In tests that touch code using consola, mock `consola` (e.g. `vi.mock("consola", () => ({ ... }))`). Prefer existing patterns under `test/`.

---
> Source: [Wetzel402/Skylite-UX](https://github.com/Wetzel402/Skylite-UX) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
