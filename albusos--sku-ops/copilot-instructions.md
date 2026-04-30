## sku-ops

> Modular monolith in production. Python/FastAPI backend, React frontend. Architecture remediation complete. These rules encode what was learned — follow them precisely.

# SKU-Ops — Agent Rules

Modular monolith in production. Python/FastAPI backend, React frontend. Architecture remediation complete. These rules encode what was learned — follow them precisely.

Consult `.cursor/rules/known-good-patterns.mdc` before any structural backend change.
Consult `.cursor/rules/deployment-hardening.mdc` before any Docker, CORS, or auth change.

---

## 1. Structural law

### Layer dependencies (strict, not aspirational)

```
shared/kernel/              → zero deps (pure domain primitives)
shared/infrastructure/      → shared/kernel only
{context}/domain/           → shared/kernel + stdlib + pydantic only
{context}/ports/            → domain + shared/kernel only
{context}/infrastructure/   → domain + shared/infrastructure
{context}/application/      → own domain + own infra + other contexts' application facades
{context}/api/              → own application + shared/api only
```

Violations of this graph are bugs, not tech debt.

### Cross-context ownership

Each bounded context owns its tables, state transitions, and invariants. Cross-context mutation goes through the owning context's application facade.

```python
# BUG — finance mutating operations' table directly
await conn.execute("UPDATE withdrawals SET payment_status = 'invoiced' WHERE id = ?", (wid,))

# CORRECT — operations owns the mutation
from operations.application.withdrawal_facade import link_to_invoice
await link_to_invoice(withdrawal_ids, invoice_id, org_id)
```

### API routes are thin

A route handler may only: parse request, call one application function, map result to response, log audit. No business logic, no orchestration, no raw SQL, no direct repo imports.

### Repos are pure persistence

Repos do CRUD and queries. No business validation, no orchestration, no cross-context access, no state machine logic.

---

## 2. Forbidden patterns

| Pattern | Why forbidden | What to do instead |
|---------|--------------|-------------------|
| `_wiring = {}` / `set_*_getter()` | Mutable global state hiding circular deps | Fix the boundary design; use module-level imports from application facades |
| `from x.infrastructure.y import z` inside function body | Service reaching through to infra details | Import collaborators at module level |
| Giant service file (15+ functions, mixed concerns) | Dumping ground; untestable | One module per important use case under `application/use_cases/` |
| `dict` as primary contract for mutation flows | No type safety, silent schema drift | Typed command/result dataclasses |
| `conn=None` on port signatures | Infrastructure leaking through abstraction | Repos call `get_connection()` directly; no `conn` parameters |
| Raw SQL in API routes | Persistence logic in transport layer | Delegate to application → infrastructure |
| Cross-context raw SQL | Invisible coupling, ownership violation | Application facade on the owning context |
| Inline JWT claim path-sniffing | Provider-specific logic scattered everywhere | Use `resolve_claims()` from `shared/api/auth_provider.py` |
| Claiming idempotency without defining mechanism | False safety guarantee | Define dedupe key, state guard, failure modes before implementation |

---

## 3. Required patterns for important flows

Any mutation that crosses context boundaries or interacts with external systems must have:

1. **Typed input** — command dataclass or explicit typed parameters
2. **Typed result** — result dataclass, not dict
3. **Module-level collaborators** — no runtime wiring, no function-body imports
4. **Explicit transaction boundary** — `async with transaction():` owned by the use case
5. **Explicit cross-context contract** — typed read models from the owning context's facade
6. **Defined idempotency** — dedupe key, DB constraint or state guard, failure recovery

---

## 4. Infrastructure facts (must know before writing backend code)

### Database layer

- **Postgres everywhere** — dev, test, and production. No SQLite. Local dev uses the local Supabase stack.
- **All SQL is native Postgres** — `$1`/`$2` placeholders, `NOW()`, `ON CONFLICT`, `::date` casts. No dialect abstraction.
- **Pool proxy:** `get_connection()` inside `transaction()` returns the dedicated transactional connection (not the pool proxy). These are the same object only because of the contextvar Unit of Work — do not call `get_connection()` outside a `transaction()` block in application code.
- **Pool config:** `PG_POOL_MIN`, `PG_POOL_MAX`, `PG_ACQUIRE_TIMEOUT`, `PG_COMMAND_TIMEOUT` — all in `config.py` and `docker-compose.yml`. Always set these explicitly in production.

### Event system

`shared/infrastructure/domain_events.py` — handler registry. Register with `@on(EventType)`, dispatch with `await dispatch(event)`. Import handler modules at startup to trigger registration. `event_hub.emit()` is only called from `shared/infrastructure/ws_bridge.py` — never call it directly from application code.

### Auth provider

`shared/api/auth_provider.py` — single source of truth for JWT claim extraction. Provider is auto-selected: supabase in production, internal in dev/test. All three auth entry points call `resolve_claims(payload)`. Do not add claim path-sniffing anywhere else.

### Adapter pattern

Only one true port-and-adapter pair: `InvoicingGateway` (Xero). Reference implementation for the pattern. Do not create new ports unless all three criteria are met (see `backend-conventions.mdc`).

---

## 5. Decision rules

### Should I create a port?

Only if ALL THREE: (1) multiple implementations exist or are plausible, (2) consumers need test isolation, (3) the boundary is stable. Otherwise keep it concrete.

### Should I move logic to domain?

Only if it's pure computation on entity state with no IO. If it coordinates aggregates, requires IO, or is use-case-specific, it stays in application.

### Should I split a file?

Split by dependency graph (different imports = different files), not by endpoint or alphabetical grouping.

### Should I create a typed model?

Yes for mutation flows and cross-context data transfer. No for simple pass-through read queries that map directly to API responses.

---

## 6. What not to do

- Do not relocate code and call it architecture. Moving a function from file A to file B without changing its dependency structure is not a fix.
- Do not preserve bad structure because it exists. If current code violates these rules, redesign, don't work around.
- Do not rush. One correctly structured module is worth more than ten relocated functions.
- Do not add abstraction layers that have one caller and one implementation. That is ceremony, not architecture.
- Do not create generic patterns (Repository[T], Service[T]) in a codebase with 5-15 entities per context.
- Do not decrease test coverage. If new abstractions are harder to test than the concrete code they replace, the abstraction is wrong.
- Do not touch Docker, CORS, or auth without reading `deployment-hardening.mdc` first.

---
> Source: [albusOS/sku-ops](https://github.com/albusOS/sku-ops) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
