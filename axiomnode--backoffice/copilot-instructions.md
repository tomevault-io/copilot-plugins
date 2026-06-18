## backoffice

> React backoffice UI for operations, monitoring, and controlled admin actions.

# AGENTS

## Repo purpose
React backoffice UI for operations, monitoring, and controlled admin actions.

## Key paths
- src/: React + TypeScript app
- public/: static assets
- .github/workflows/ci.yml: build pipeline + infra dispatch

## Local commands
- npm install
- npm run dev
- npm run build

## CI/CD notes
- Push to main dispatches platform-infra build for backoffice image.
- Runtime deployment is managed by platform-infra (dev first).

## LLM editing rules
- Keep role-gated flows explicit and documented.
- Preserve API base URL/token env conventions.
- Update docs when UI behavior or environment setup changes.

---
> Source: [AxiomNode/Backoffice](https://github.com/AxiomNode/Backoffice) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
