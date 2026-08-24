## pen-security-quality-gates

> Security defaults and quality gates for Pen contributions


# Security and Quality Gates

- Default to safe boundaries: sanitize untrusted HTML, validate tool inputs, and enforce transport/payload checks.
- Never assume trust for external, network, or process input; validate early and fail safely.
- Keep changes aligned with acceptance criteria in `spec/` wave docs for the touched area.
- For substantive changes, run the monorepo quality checks before completion:
  - `pnpm build`
  - `pnpm typecheck`
  - `pnpm test`
- For perf-sensitive paths, avoid regressions and respect benchmark expectations in `@pen/bench`.

---
> Source: [input-systems/pen](https://github.com/input-systems/pen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
