## known-good-patterns

> Validated patterns that must be preserved — consult before structural changes


# Known-Good Patterns

Validated in production. Do not revert without a documented reason.

## Domain purity
Domain files import only `shared/kernel/`, stdlib, pydantic. Zero infrastructure.

## Unit of Work — contextvar ambient transactions
`transaction()` stores the transactional connection in `_tx_conn` ContextVar. `get_connection()` returns it inside a transaction block; repos call it directly and automatically join the ambient transaction. Nested `transaction()` calls reuse the existing connection.

asyncpg + FastAPI + asyncio is exactly the environment contextvars were designed for — each request is an isolated asyncio task. This is correct. Do not thread `conn` through function signatures.

## Domain events
`@on(EventType)` / `await dispatch(event)` — handler registry in `shared/infrastructure/domain_events.py`. Typed event classes in `shared/kernel/`. Register handlers in `startup.py` by importing handler modules. Every mutation visible to users must dispatch a typed event — this is the auditability and real-time push mechanism.

`event_hub.emit()` is called only from `shared/infrastructure/ws_bridge.py`. Never call it directly.

## InvoicingGateway port
Only true port-and-adapter pair. Reference implementation. Do not create new ports unless all three criteria are met (see backend-conventions.mdc).

## Auth provider abstraction
`shared/api/auth_provider.py` is the only place JWT claims are extracted. Provider is auto-selected: supabase in production, internal in dev/test. All three auth paths call `resolve_claims(payload)`. Adding a provider = one new branch in that file only.

## Org isolation
Every write scoped to `org_id` from `get_org_id()` (ambient contextvar set by auth middleware). `Entity.organization_id` has no default — every domain entity must be constructed with an explicit org_id. `DEFAULT_ORG_ID = "default"` exists only for seed scripts and test fixtures.

## Structured logging
Module-level `logger = logging.getLogger(__name__)` everywhere. Mutations log at INFO with `extra={"org_id": ..., "<entity>_id": ..., "action": ...}`. No `print()`.

## External call resilience
Xero, Anthropic, OpenRouter: explicit timeouts, log failures at ERROR with `exc_info=True`, do not commit the local transaction if the external call fails.

## Cross-context reads
`catalog/application/queries.py`, `operations/application/queries.py`, `purchasing/application/queries.py`. Never import from another context's `infrastructure/`.

---
> Source: [albusOS/sku-ops](https://github.com/albusOS/sku-ops) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
