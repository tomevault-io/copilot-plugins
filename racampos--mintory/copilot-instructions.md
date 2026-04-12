## mintory

> Enforce REST + SSE backend API and MCP tool contracts during implementation and review.


# API Contracts Enforcement

**Backend REST + SSE:** Must match `docs/context/backend_api_spec.md`.

- `/runs` start, `/runs/{id}` status, `/runs/{id}/stream` SSE, `/runs/{id}/resume` decisions.

**MCP server:** Must match `docs/context/mcp_tools_spec.md`.

- All write calls return **PreparedTx**, never send transactions.

**Validation:** Add request/response validators and integration tests that assert exact shapes.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/racampos)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/racampos)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
