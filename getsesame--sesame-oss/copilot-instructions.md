## sesame-oss

> Source-available credential broker for AI agents. Agents call `sesame request ...`; the broker fetches the secret, injects the auth header server-side, forwards the request, and returns the response — the agent never sees the key. Humans approve each new hostname from the browser dashboard (web push). Licensed under the Elastic License 2.0.

# Sesame (community edition)

Source-available credential broker for AI agents. Agents call `sesame request ...`; the broker fetches the secret, injects the auth header server-side, forwards the request, and returns the response — the agent never sees the key. Humans approve each new hostname from the browser dashboard (web push). Licensed under the Elastic License 2.0.

This file orients AI coding assistants and new contributors. See `CONTRIBUTING.md` for the full workflow and `docs/ARCHITECTURE.md` for how the pieces fit together.

## Repo layout

Hybrid Python + TypeScript monorepo. UV workspace for Python, pnpm for web.

- `packages/sesame-core` — Python. Shared Pydantic models, crypto (Fernet/Ed25519), JWT, constants. Imported by broker + ctl.
- `packages/sesame-broker` — Python/FastAPI. REST API on `:8000`. Talks to Postgres (metadata) and AWS Secrets Manager (secret values); web push via VAPID.
- `packages/sesame-ctl` — Python/Typer CLI + the `sesame deploy aws` command. The same published `sesame` binary works against any broker.
- `packages/sesame-web` — React 19 + Vite + Tailwind dashboard (SPA).
- `packages/sesame-ui` — shared React components, imported by `sesame-web` via the `@sesame/ui` Vite alias.
- `deploy/self-host/` — docker-compose self-host stack. `deploy/aws-self-host/` — notes for `sesame deploy aws`.
- `migrations/` — SQL migrations (`self_host/` variant for the Postgres self-host path; applied automatically on broker boot).
- `tests/` — pytest suite. `scripts/e2e_selfhost_smoke.py` — end-to-end harness.

## Commands

Always go through the Makefile — it wires up UV + pnpm correctly.

```bash
make install        # UV workspace + pnpm web deps
make run-broker     # uvicorn on :8000  (needs a Postgres + .env; see deploy/self-host/README.md)
make run-web        # Vite dev server, proxies /v1 to the broker
make lint           # ruff check
make format         # ruff format + fix
make test           # pytest tests/
```

## Conventions

- **Python**: 3.12+, Pydantic models, Ruff for lint/format, pytest. Add new shared types to `sesame-core`, not the broker.
- **Endpoints**: extend existing broker routes rather than adding sibling ones.
- **TS**: one component per file; keep a proper directory structure. Don't annotate `unknown` — narrow at the runtime guard.
- **Comments**: default to none. Only justify non-obvious *why*.
- **No `setTimeout`/`setInterval`** unless genuinely unavoidable (prefer event-driven / lifecycle hooks); note it in the PR if used.
- **Secrets**: `.env` is gitignored. Never commit `.env*` or any key material.
- **Non-interactive shell**: use `cp -f`, `mv -f`, `rm -f` (system aliases may add `-i`).

---
> Source: [getsesame/sesame-oss](https://github.com/getsesame/sesame-oss) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-24 -->
