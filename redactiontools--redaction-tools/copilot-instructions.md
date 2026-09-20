## redaction-tools

> Catalog/directory of redaction tools with benchmarks and a leaderboard.

# redaction-tools

Catalog/directory of redaction tools with benchmarks and a leaderboard.
`backend/` is Django + django-ninja; `frontend/` is Next.js on bun.

`DEVELOPMENT.md` covers setup, the auth flow and the schema pipeline.

## Use `make`, not the underlying tools

`make help` lists every target. Targets are namespaced per component
(`backend-*`, `frontend-*`) and the unprefixed aggregates run the task
everywhere: `make test`, `lint`, `fmt`, `check`, `schema`, `install`.

Run `make backend` / `make frontend` rather than `uv run manage.py runserver` or
`bun run dev`, so the ports stay consistent.

| Service | URL |
| --- | --- |
| Backend | http://localhost:8007 (`/admin/`, `/api/v1/docs`, `/api/v1/health`) |
| Frontend | http://localhost:3007 |
| Postgres | 5432 by default; **5433 on this machine** (5432 is taken) |

## The API contract is generated, in two steps

The chain is: Python routers → `backend/openapi.json` → `frontend/src/lib/api/generated/`.
Both generated artifacts are **committed**, and CI fails if either is stale.

After changing any endpoint:

```bash
make schema   # regenerates openapi.json AND the TypeScript client
```

- **Never hand-edit** `backend/openapi.json` or anything under
  `frontend/src/lib/api/generated/`.
- Operation ids come from the **view function name** (`RedactionAPI` in
  `config/api.py` overrides ninja's dotted-path default). `def get_me` becomes a
  `useGetMe()` hook, so view names must be unique across the whole API —
  `backend/tests/test_openapi_schema.py` enforces that.
- The spec declares no 4xx responses, so generated hooks are typed with
  `ApiError` via the `Register` augmentation in `src/types/react-query.d.ts`.

## Auth

The frontend never uses session cookies against `/api/`. It gets a Google ID
token, trades it at `/_allauth/app/v1/auth/provider/token` for our own JWT pair,
and sends the access token as a bearer. Refresh tokens **rotate** — the new one
must replace the old. Access tokens live 15 minutes.

Session cookies still exist, but only for the Django admin and the
server-rendered `/accounts/` allauth views.

Google credentials come from environment variables, **not** a `SocialApp` row in
the admin — configuring both raises `MultipleObjectsReturned`.

## The catalog lives in `apps/catalog/`

The parts that bite if you miss them:

- **`Tool.is_listable()` is the public gate.** It reads related rows, so it runs in Python, not
  SQL - which is why `ToolQuerySet.listable()` returns a list. Every public route filters
  through it; a tool that fails it is a row, not a page.
- **Filtering is OR within a dimension, AND across.** That means one chained `.filter()` per
  dimension in `apps/catalog/filters.py`; a single combined `__in` would OR everything and turn
  each added filter into a widening one.
- **`PlanPrice.save()` pins anything non-crawler.** Staff and vendor figures are authoritative
  by default, so the crawler (phase 2) can never overwrite one.
- **Owners never write a price.** `PriceProposal` is the only path, and it publishes through the
  admin as `source=VENDOR`, pinned and badged.
- **Seed migrations are idempotent** and guarded by an existence check. They put 7 tools in every
  test database, so a test asserting "the catalog is empty" will not hold.
- Catalog routes and `sitemap.ts` are `force-dynamic`: `next build` runs with no backend (CI
  builds the image), so anything prerendered would bake an empty catalog into the bundle.
- `orval.config.ts` sets neither `useQuery` nor `useMutation` on purpose - each applies to every
  operation, so setting either gives POSTs query hooks or GETs mutation hooks.

## The staff MCP server lives at `/mcp`

`DEVELOPMENT.md` has the full picture. What bites:

- **It is outside the generated contract.** `make backend-schema` exports `config.api.api`
  only, so an MCP change must leave `openapi.json` and the Orval client untouched -
  `make schema && git diff --exit-code` is the check.
- **Writes go through `apps/catalog/staff.py`, never straight to the ORM.** That module
  imports neither ninja nor MCP and takes `user` explicitly, which is what keeps the price
  rules in one place and every write attributable. Refusals are `StaffError`; the MCP layer
  turns them into an in-band `isError` a model can correct from.
- **`Tool.is_listable()` delegates to `listability_blockers()`** in `models.py`. Change the
  bar there. It is a generator so `is_listable()` still short-circuits, which matters
  because the public list runs it over every published tool on every request.
- **`set_plan_price` closes only the matching (currency, billing period, overage) slot.**
  `PriceProposalAdmin.approve_proposals` closes every current row, which retires a metered
  plan's overage rate along with its monthly fee - copy the service, not the admin.
- Optional MCP parameters are `msgspec.UNSET`, and a numeric constraint goes on the inner
  type: `Annotated[int, Meta(ge=0)] | UnsetType`.
- **One MCP tool per table.** A plan, its caps and each of its prices are written by
  separate tools, because a single call that creates all four can half-succeed and because
  a price carries provenance a plan does not. `create_plan` reports
  `has_pricing_position` so a caller can tell it has not finished.

## Conventions

- Tests come first; the repo carries tdd-guard's rulebook in
  `.claude/tdd-guard/data/`. One failing test at a time, then minimal code.
- Backend: ruff, line length 100, tests in `backend/tests/`, pytest fixtures in
  `backend/conftest.py`.
- Frontend: see `frontend/CLAUDE.md`.
- `.env` files are gitignored; `.env.example` files are the documented contract
  and must stay in sync when a variable is added.

---
> Source: [RedactionTools/redaction-tools](https://github.com/RedactionTools/redaction-tools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-20 -->
