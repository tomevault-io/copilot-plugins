## trace-it

> Configurable decision processes: a process pack (JSON) declares decision types, symbols and

# trace-it: guide for assistants and newcomers

Configurable decision processes: a process pack (JSON) declares decision types, symbols and
rules in plain language; agents compile the rules to Python; a deterministic engine runs them
in a sandbox and every instance ends with a decision and its evidence. The invoice-payment
process of the "500 Sombras de Alberto" challenge is the first pack. No LLM ever decides.

## Where things are

| Path | What |
|---|---|
| `backend/app/features/{users,processes,rules,decisions,ingestion,sources,agents,traces}` | One folder per feature: `model.py`, `schemas.py`, `service.py`, `router.py`, `tests/` |
| `backend/app/core/` | settings (`TRACE_*`), database, `events.py` (spans: the audit trail, ADR 0018) |
| `backend/app/features/*/prompts/*.md` | Every instruction a model receives, one file each: `agents/prompts/` for the rule and decision agents, `ingestion/prompts/` for the readers. Never inline in Python (`app/common/prompts.py`) |
| `backend/app/cli.py` | `load <pack> [--compile] [--activate]`, `sources sync <pack>` |
| `processes/` | Process packs; `invoice-payment.json` + `invoice-payment/sources.json` + `rules-v3/*.py` |
| `tools/` | Production API demo driver (`make demo`) and historical text-layer benchmark helpers |
| `frontend/` | The manager's console (Vite + React). Its API, screen by screen: `docs/api.md` |
| `docs/adr/` | Decisions. ADR 0001 wins over any other document |
| `.artifacts/` | Archived plans and analyses. History, not guidance |

## Rules of the house

- English everywhere: code, docs, commits, API. Domain names from `docs/CONVENTIONS.md`.
- Don't overcomplicate: this is an MVP built in a weekend. One implementation, no
  abstraction for a second case that does not exist yet.
- The engine is pure: no clock, network or database inside `decisions/engine.py`. A cut-off
  date is a row in a source, never `today()`.
- Rule code contract: `evaluate(instance, sources, others) -> {"fires": bool, "reason": str}`,
  run only through `agents/sandbox.py`.
- History is append-only: decisions, sources and files are never edited. A rule change is
  checked against past decisions before it is adopted.
- Tests beside their feature; `make check` must be green (unit + golden e2e) before merging.
  No test may need an LLM key: script models with `tests/support/models.py`.
- Branch from `dev`, PR into `dev`, squash merge. `main` is what we show.

## Running

```bash
make up         # ERP bridge + setup + sources: all a local run needs
make setup      # Postgres + API in Docker, pack loaded        make test       # unit tests
make erp        # the challenge ERP bridge (other terminal)    make test-e2e   # golden + API flow
make sources    # master workbook + ERP snapshot, no invoices (needs setup + erp)
make demo       # upload the 500 invoices through the API (including OCR), then decide/export
make reset-db   # only for a database older than migration 0001
```

Details: `docs/team-guide.md`. Why things are the way they are: `docs/adr/README.md`.

## Production database access

The production database admin API is available through the Trace-it API. Keep the bearer
token out of Git and shared logs. Read it into an environment variable when needed:

```bash
export TRACE_API_BASE='https://gex-dashboard.hopto.org/nexia/trace-it/api'
read -rs TRACE_API_TOKEN
export TRACE_API_TOKEN

curl --fail-with-body "$TRACE_API_BASE/db/tables" \
  -H "Authorization: Bearer $TRACE_API_TOKEN"

# Inspect the active chat/assistant model.
curl --fail-with-body --get "$TRACE_API_BASE/db/tables/agent_configs/rows" \
  -H "Authorization: Bearer $TRACE_API_TOKEN" \
  --data-urlencode 'filters={"role":"assistant","active":true}' \
  --data-urlencode 'limit=50'
```

The database API supports exact JSON equality filters. Prefer the business API for normal
changes. Direct database writes are administrative overrides and require an audit reason.

---
> Source: [Martinhdeez/trace-it](https://github.com/Martinhdeez/trace-it) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-26 -->
