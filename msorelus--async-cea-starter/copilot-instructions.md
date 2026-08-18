## async-cea-starter

> Instructions for AI coding agents working in this repository. Humans should read [`README.md`](README.md) first; this file gives an agent the context and guardrails to make safe, correct changes.

# AGENTS.md

Instructions for AI coding agents working in this repository. Humans should read [`README.md`](README.md) first; this file gives an agent the context and guardrails to make safe, correct changes.

## What this repo is

The **Async CEA Starter** is a reference kit for an async custom-engine agent (CEA) that answers 1:1 on **Microsoft Teams** and **Microsoft 365 Copilot** from the same Azure Bot Service registration, using the M365 Agents SDK for Python. Its whole reason to exist is one pattern: **acknowledge fast, run the long job out-of-band, deliver the final answer proactively.**

## The golden rule

**Never make the message handler await the long-running job.** The Teams turn must return within ~15 seconds.

The flow in `python/async_reply.py` is, in order:

1. `await context.send_activity(ASYNC_ACK_TEXT)` — fast ack inside the turn.
2. `await app.proactive.store_conversation(context)` — persist the conversation reference.
3. `asyncio.create_task(_run_worker(...))` — run the backend **detached** (not awaited).
4. In the worker: call the backend, then `app.proactive.continue_conversation(...)` to deliver the answer back into the same thread.

Do not collapse these into a single awaited call, do not block the handler on the backend, and do not remove the dedupe (`_is_duplicate`) or the strong-ref worker set (`_bg_workers`). Breaking this sequence breaks the entire kit.

## Project map

| Path | What it is |
|------|------------|
| `python/app.py` | Composition root: builds auth, adapter, storage, agent; aiohttp host; `/api/messages` + `/healthz` |
| `python/agent.py` | Teams / M365 Copilot surface wiring (backend-agnostic) |
| `python/async_reply.py` | The async/proactive pattern — the core value; change with extreme care |
| `python/backends.py` | `AIBackend` protocol + `StubBackend` + `create_backend()` — the swap seam |
| `python/config.py` | Typed, frozen `AppConfig`; reads env (two credential spellings) |
| `python/obs.py` | No-op observability seam (active only when `OBS_HUB_URL` is set) |
| `infra/` | Parameterized Bicep (6 modules); `scripts/deploy.sh` deploys it |
| `appPackage/manifest.json` | Dual-surface Teams v1.21 manifest (`customEngineAgents`) |
| `docs/` | Architecture, operations, security, backend-swap, manual-steps, runbook |
| `tests/` | Pytest suite (repo root — **excluded** from the deploy zip) |
| `examples/http_backend.py` | Ready-to-copy real backend (repo root — excluded from deploy) |

## The one seam you will most often touch

To replace the stub with a real agent, change **only** `python/backends.py`:

```python
def create_backend(cfg: AppConfig) -> AIBackend:
    return MyBackend(...)   # must implement: async def respond(self, conversation_key, user_text) -> str
```

`agent.py` imports only the `AIBackend` protocol, so nothing else changes. A complete worked example is in `examples/http_backend.py`; see [`docs/backend-swap.md`](docs/backend-swap.md).

## Build, test, validate

Run these from the repo root before committing — they mirror CI (`.github/workflows/ci.yml`):

```bash
# Unit tests (use the kit venv, or any Python 3.12 with deps installed)
python/.venv/bin/python -m pytest

# Lint
python/.venv/bin/python -m ruff check .

# Infrastructure compiles (warnings BCP037 on M365Extensions and the storage
# endpoint linter note are known and non-blocking)
az bicep build --file infra/main.bicep

# Manifest is still dual-surface
python scripts/validate_manifest.py

# No live/customer identifiers leaked back in
bash scripts/scrub_gate.sh
```

Tests use lightweight fakes and do not require Azure or network access.

## Hard rules

- **No hardcoded secrets, tenant IDs, or customer names.** Secrets come from Azure Key Vault via references. Use placeholders like `__BOT_APP_ID__`. The scrub gate must stay green.
- **Keep it dual-surface.** Dual-surface = the v1.21 manifest's `copilotAgents.customEngineAgents` entry **and** the `M365Extensions` channel in `bot.bicep`. Do not remove either. In v1.21 each `customEngineAgents` item is `{id, type: "bot"}` only.
- **One Python process, target runtime 3.12.** App Service runs `PYTHON|3.12`. Do not introduce a second service or a Node/TypeScript surface.
- **`/healthz` stays anonymous and trivial.** It must bypass `jwt_authorization_middleware` (its middleware is ordered first) and return only a static status — never application data, and never widen the bypass to other routes.
- **Keep test/dev artifacts out of the deployed app.** `scripts/deploy.sh` zips only `python/*.py` + `python/requirements.txt`. Put tests, `pyproject.toml`, `requirements-dev.txt`, `.github/`, and `examples/` at the **repo root**.
- **Do not modify the source proof-of-concept** or reach outside this repo.

## Conventions

- Backend contract: `async def respond(self, conversation_key: str, user_text: str) -> str`.
- Credentials are read as camelCase `clientId` / `tenantId` / `clientSecret` with `CLIENT_ID` / `TENANT_ID` / `CLIENT_SECRET` fallbacks — **not** the legacy `MicrosoftAppId` / `MicrosoftAppPassword`.
- New runtime dependencies go in `python/requirements.txt`; dev/test-only ones in `requirements-dev.txt`.
- Ship runtime code in `python/`; keep everything test/CI/example at the repo root.

## Accepted deviations

- **D1 — network hardening kept.** Key Vault and Storage start with public access enabled, then `deploy.sh` locks them to private endpoints. The brief public window is intentional.
- **D2 — Agent 365 admin-center visibility is entitlement-gated.** Registry visibility requires the appropriate Microsoft 365 entitlement; the Microsoft Entra admin center is the guaranteed surface.

## Where to look first

- Understand the pattern: [`docs/architecture.md`](docs/architecture.md), then `python/async_reply.py`.
- Swap the backend: `python/backends.py` + [`docs/backend-swap.md`](docs/backend-swap.md).
- Run / operate it: [`README.md`](README.md) and [`docs/operations.md`](docs/operations.md).
- Security review: [`docs/security.md`](docs/security.md).

---
> Source: [msorelus/async-cea-starter](https://github.com/msorelus/async-cea-starter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-17 -->
