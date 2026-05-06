## backend-conventions

> Backend Python conventions — architecture patterns, anti-patterns, decision rules


# Backend Conventions

## Layer structure

```
shared/kernel/         ← pure primitives, zero deps
shared/infrastructure/ ← db, redis, config, middleware
{ctx}/domain/          ← shared/kernel + stdlib + pydantic only
{ctx}/infrastructure/  ← domain + shared/infrastructure
{ctx}/application/     ← orchestrates via own infra + other contexts' facades
{ctx}/api/             ← thin HTTP transport, own application + shared/api only
```

Violations are bugs. Cross-context mutation goes through the owning context's application facade.

## Repos: persistence only

No business validation, no orchestration, no cross-context mutations, no state transitions. Analytics queries go in `queries.py` or `application/ledger_queries.py`.

## Application layer shape

Every application function must have:
- **Named input contract** — typed parameters or a command dataclass. The caller must know exactly what they're passing.
- **Named output contract** — a named typed model (dataclass or Pydantic). The caller must know exactly what they're getting. An anonymous `dict` is not a contract — it has no name, no schema, no OpenAPI visibility, and drifts silently.
- **Module-level collaborators** — no imports inside function bodies
- **Transaction boundary** — `async with transaction():` for all writes
- **Structured log** — `logger.info("action", extra={...})` on every mutation

```python
# NO CONTRACT — caller can't introspect, OpenAPI can't document, type checker can't verify
async def sales_report(...) -> dict: ...

# EXPLICIT CONTRACT — named, typed, versioned, visible
@dataclass(frozen=True)
class SalesReport:
    gross_revenue: Decimal
    gross_profit: Decimal
    gross_margin_pct: float
    total_transactions: int

async def sales_report(...) -> SalesReport: ...
```

`dict` is fine as an internal intermediate. It is not acceptable as a public output contract of an application function.

## API layer

Route handlers only: parse request → call one application function → map to response → log audit. No SQL, no repos, no orchestration.

## Auditability

Every mutation that changes state visible to users must either dispatch a typed domain event via `dispatch(event)` or write an audit record in the same transaction.

## Error handling

Do not swallow exceptions. Raise a typed `DomainError` subclass with a user-readable message, or let infrastructure errors propagate for the API layer to translate.

## Forbidden patterns

| Pattern | Fix |
|---|---|
| `_wiring = {}` / `set_*_getter()` | Fix the boundary; module-level import from facade |
| Import inside function body | Module-level collaborator |
| `conn` on repo/port signatures | Repos call `get_connection()` directly |
| Raw SQL in API routes | Delegate to application → infra |
| Cross-context infra imports | Application facade on owning context |
| Calling a flow "idempotent" without defining dedupe key + guard | Define it explicitly |

## Decision rules

- **Port?** Only if: multiple implementations plausible AND consumers need test isolation AND boundary is stable. Otherwise concrete.
- **Domain vs application?** Domain if pure computation on entity state. Application if IO, multi-aggregate, or use-case-specific.
- **Split file?** By dependency graph, not by endpoint or alphabet. Different imports → different files.
- **File too large?** Split into `{ctx}/application/use_cases/` — one function per module.

---
> Source: [albusOS/sku-ops](https://github.com/albusOS/sku-ops) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
