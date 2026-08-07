## agentzt

> Guidance for Claude Code (and other agents) working in this repository.

# CLAUDE.md

Guidance for Claude Code (and other agents) working in this repository.

## What this is

`agentzt` — **Zero Trust for AI Agents**. An AI agent reaches enterprise **model APIs** and
**tools** only through a zero-trust proxy, the same way a human reaches enterprise apps
through a ZTNA client + gateway. It does **not** replace ZTNA for humans; it secures the
agent→model and agent→tool path.

- **`agentzt-client`** (`src/client`) — local proxy + per-agent identity. The agent points
  its model SDK at it (`ANTHROPIC_BASE_URL=http://localhost:8787`); the client attaches a
  short-lived token to every request and forwards to the gateway.
- **`agentzt-gateway`** (`src/gateway`) — PDP+PEP. Authenticates the agent, issues scoped
  short-lived tokens, enforces deny-by-default RBAC + ABAC + JIT, runs input/output
  guardrails, audits everything, then forwards to the real model API / tools.

Open source, Apache 2.0, repo: https://github.com/openguardrails/agentzt

## Hard constraints — read before editing

- **Zero runtime dependencies. No build step.** Everything runs on **Node ≥ 22.6** via
  native TypeScript type-stripping (`node src/...ts`). Do **not** add npm dependencies or a
  bundler. `package.json` deps stay empty (only `@types/node` + `typescript` as devDeps for
  editor/typecheck).
- Because of type-stripping, write **erasable** TypeScript only: **no `enum`, no
  `namespace`, no constructor parameter properties, no decorators.** Use `import type` for
  type-only imports and **`.ts` extensions** on every relative import (ESM).
- The only allowed external process is **`openssl`**, and only in `src/cli/tls.ts` for
  opt-in mTLS PKI setup. The runtime never shells out.
- Use Node built-ins only: `node:http`, `node:https`, `node:crypto`, `node:tls`, global
  `fetch`. Note global `fetch` can't carry client certs — use `src/client/transport.ts`
  (which falls back to `node:https`) for outbound client calls.

## Commands

```bash
npm run demo        # end-to-end: enroll, start gateway+client, drive 8 control checks, audit
npm run demo:mtls   # same, over mutual TLS (needs openssl)
npm test            # 28 checks (auth, RBAC, guardrails, ABAC, JIT, audit chain, mTLS)
npm run typecheck   # tsc --noEmit (needs `npm i` once for typescript)

npm run gateway                                  # start gateway (PDP+PEP)
AGENTZT_AGENT_ID=<id> npm run client             # start client proxy for an agent
node src/cli/index.ts enroll --agent <id> --role <role> [--mtls]
node src/cli/index.ts roles | agents | audit [--verify] | tls init | tls issue --agent <id>
```

Always run `npm test` after changes; it boots real gateways (HTTP then HTTPS) and exercises
the full request path.

## Layout

```
src/shared/    types, crypto (Ed25519 + compact JWS), audit (hash chain), config, http, paths
src/gateway/   server (routing+authz), token-service, policy-engine (RBAC/ABAC/JIT),
               identity-store, guardrails + guardrail-providers, tool-registry, upstream,
               rate-limiter, gateway-key
src/client/    index (entry), identity, token-client, proxy, transport (mTLS)
src/cli/       index (enroll/agents/roles/audit/tls), tls (openssl PKI)
config/        policy.json (roles), gateway.json, agents.json (registry, public keys only)
examples/      demo-agent
scripts/       demo.ts, demo-mtls.ts, test.ts
.agentzt/      runtime state — private keys, gateway key, TLS material, audit logs (gitignored)
```

## Security model (what must stay true)

- The agent holds **no** enterprise credentials and **no** access token — only its Ed25519
  private key. The enterprise model API key lives **only** on the gateway (`upstream.ts`).
- Authorization is layered; any layer can deny: **mTLS → token authn → scope|JIT → rate
  limit → input guardrail → ABAC → execute → output guardrail.** When adding a control,
  slot it into this order in `gateway/server.ts` and audit the decision.
- **Deny-by-default**: never grant a resource that isn't explicitly allow-listed. High-blast
  resources (e.g. `email.send`) stay out of standing scope and require JIT elevation.
- Every decision must be written to the audit log (`AuditLogger.record`) with the request id
  so a task reconstructs as one chain. Don't add a code path that bypasses audit.
- Guardrails: prefer **OpenGuardrails** (`guardrail-providers.ts`) for detection; keep the
  local regex provider as the offline fallback so demo/tests run without network.

## Conventions

- Two-space indent, ESM, `import type` for types. Match the existing terse comment style:
  explain *why* (which Zero Trust control), not *what*.
- Keep the demo offline-capable (mock upstream, local guardrails) so `npm run demo` and
  `npm test` need no network or API keys.
- Before committing: reset `config/agents.json` to an empty `agents` array and `rm -rf
  .agentzt` (runtime state must not be committed). Never commit the eBook PDF (gitignored).
- Commit/push only when asked. The repo is public; treat anything pushed as published.

## Framework reference

The design follows Anthropic's *Zero Trust for AI Agents* framework (Foundation → Enterprise
→ Advanced tiers). `docs/ARCHITECTURE.md` maps each implemented control to its tier and
lists the remaining roadmap (immutable audit sink/SIEM, HSM/KMS-bound gateway key).

---
> Source: [openafw/agentzt](https://github.com/openafw/agentzt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-06 -->
