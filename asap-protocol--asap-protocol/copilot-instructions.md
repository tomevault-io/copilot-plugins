## architecture-principles

> ASAP Protocol product architecture: Core Python, Web app, and Lite Registry layout. Apply when designing features or choosing where code belongs.


# Product Architecture (v2.0.0+)

The ASAP Protocol uses a **Lean Marketplace** polyglot architecture:

1. **Core Protocol (`src/asap/`)**: Strict Python. Uses Pydantic for validation and `uv` for dependency management. No backend APIs.
2. **Web App (`apps/web/`)**: Next.js 16 (React 19) + TypeScript + Tailwind v4. Static consumer (SSG/ISR) of the Lite Registry.
3. **Lite Registry**: A single `registry.json` hosted on GitHub Pages acts as the database. Registration and updates happen via GitHub PRs (IssueOps), *not* via a write API.

## Where code belongs

| Concern | Location |
|---------|----------|
| Protocol models, transport, auth | `src/asap/` |
| Marketing / registry UI | `apps/web/` |
| Shared UI primitives | `packages/ui/` |
| ADRs and stack decisions | `engineering/architecture/`, `product/decision-records/` |

For clean code standards (SOLID, naming, tests), use `agent-clean-code.mdc`.

---
> Source: [asap-protocol/asap-protocol](https://github.com/asap-protocol/asap-protocol) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
