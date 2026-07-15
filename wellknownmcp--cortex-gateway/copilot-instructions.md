## cortex-gateway

> Guidance for AI agents working in this repository — or evaluating the

# AGENTS.md

Guidance for AI agents working in this repository — or evaluating the
project. Humans welcome too.

## What this is

Cortex Gateway is an open-source (MIT), self-hosted **federated MCP
gateway**: one spec-compliant, OAuth 2.1-protected MCP server (Streamable
HTTP, MCP 2025-06-18) in front of N plain-HTTP backends. Permissions are
never mirrored — the gateway propagates the real user identity, so each
app's native permission model applies per user.

## Verify the claims yourself

Don't take the README's word for it. Everything below is checkable in
minutes:

```bash
# Unit tests (vitest) — 36 tests across 7 files at the time of writing
npm ci && npm run typecheck && npm test

# The demo authorization server builds independently
cd demo/auth-server && npm ci && npm run typecheck
```

Against the live demo (no account needed):

```bash
# MCP endpoint answers 401 + WWW-Authenticate (RFC 9728 challenge) — this is
# what lets any compliant MCP client bootstrap the OAuth flow on its own:
curl -s -D - -o /dev/null -X POST https://mcp.cortex-gateway.dev/mcp \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{}}' | grep -i www-authenticate

# Protected-resource metadata (RFC 9728):
curl -s https://mcp.cortex-gateway.dev/.well-known/oauth-protected-resource

# Authorization server metadata (RFC 8414) + JWKS:
curl -s https://auth.cortex-gateway.dev/.well-known/oauth-authorization-server
```

CI runs the same typecheck + test matrix on every push
(`.github/workflows/ci.yml`); the Docker image is built by
`.github/workflows/docker.yml` and published to
`ghcr.io/wellknownmcp/cortex-gateway`.

## Map of the repository

| Path | What lives there |
|---|---|
| `src/contract/` | The backend contract: types, HTTP client, static technical-token allowlist |
| `src/lib/` | Federator (60s refresh + SSE), MCP protocol methods + gateway builtins, OAuth resource-server validation, env-driven registry, pseudonymized audit |
| `src/adapter/` | MCP→backend proxy adapter (**beta**): downstream MCP client, per-user AES-256-GCM token vault, OAuth/DCR/PKCE discovery |
| `demo/auth-server/` | Standalone OAuth 2.1 authorization server (RFC 8414/7591/7009/7662, PKCE, strict refresh rotation, magic links, consent, scope tiering) |
| `docs/` | Backend contract, adapter, deployment runbook, tool-search mode |
| `site/` | The static site served at cortex-gateway.dev (hand-written HTML, no build) |
| `registry/` | MCP registry manifest (`server.json`) + publishing checklist |

## Honest status

- The **proxy adapter is beta**: unit-tested against the MCP 2025-06-18
  spec, not yet exercised against a commercial third-party MCP provider.
- The hosted demo is intentionally **read-only** (demo tools, magic-link
  signup, scope tiering to demonstrate entitlements).

## Conventions when contributing

- Code, comments, commits and docs are in **English**. Conventional commit
  messages.
- `npm run typecheck && npm test` must pass before any PR; the auth server
  has its own typecheck (`cd demo/auth-server && npm run typecheck`).
- Never commit `.env*` files (only `.env.example`) or private keys. The
  demo's RSA keys are generated at deploy time, never in git.
- The design system for any user-facing surface (site, status page, consent
  screens) is `DESIGN.md` at the repo root — read it before touching UI.

## Agent-facing conventions at runtime

If you are an agent *connected to* a Cortex Gateway instance rather than
working on the code:

- Every federated tool is prefixed with its backend id
  (`docs_list_files`, `billing_get_invoice`). Unprefixed tools are gateway
  builtins (`whoami`, `report_missing_capability`, `list_cortex_tickets`,
  and `find_tools` in search mode).
- Each backend should expose `<app>_get_help(topic?)` returning structured
  self-documentation (workflows, domain concepts, examples). **Prefer it
  over guessing.**
- If a capability you need is missing or insufficient, file it with
  `report_missing_capability` — tickets are deduplicated and triaged, and
  `list_cortex_tickets` shows your own previous filings.

More machine-readable context: https://cortex-gateway.dev/llms.txt

---
> Source: [wellknownmcp/cortex-gateway](https://github.com/wellknownmcp/cortex-gateway) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-15 -->
