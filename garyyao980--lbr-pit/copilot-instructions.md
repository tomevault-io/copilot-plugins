## lbr-pit

> A point-in-time market data slice: CFTC COT and FRED/ALFRED fundamentals loaded with

# lbr-pit — working rules

A point-in-time market data slice: CFTC COT and FRED/ALFRED fundamentals loaded with
two time axes, served by FastAPI, drawn by React. Postgres in Docker, app on the host.

**Stack is fixed:** Python (pandas, psycopg), SQL/Postgres, FastAPI, React + Vite + Recharts.
Do not substitute libraries or add frameworks without being asked.

---

## The one idea this repo exists to demonstrate

Every observation carries **two dates**: when the value *applies to*, and when it *became
knowable*. A query that ignores the second one is reading the future.

```sql
SELECT DISTINCT ON (valid_col, series)
       valid_col, series, value
FROM   <table>
WHERE  known_col <= :as_of
ORDER  BY valid_col, series, revision DESC;
```

`cot_obs` uses `(report_date, release_ts)`. `fundamental_obs` uses `(obs_date, vintage)`.
**One generic function serves both** — see `app/queries.py`. Keep it that way; if a change
would force the two tables apart, that is a signal the change is wrong.

The mirror query, `first_published`, is the same SQL with `ASC` instead of `DESC`. It is
what draws the "as first published" line. **Both are needed; neither is redundant.**

---

## Invariants — do not break these

**Contract months are labels, never sort keys.** `contract_month TEXT` does not sort
chronologically: `JAN27` sorts before `SEP26`. Every curve query orders by
`contract_start DATE`. **`ORDER BY contract_month` must not appear anywhere in this
repository.** Relative legs (`N1`, `N2`) resolve through `queries.resolve_leg`, by
`contract_start`, in SQL — never by comparing strings.

**Never fabricate data to make a chart look fuller.** Where a source is missing, the panel
says so on screen and the loader still exists. Synthetic values are permitted only as an
explicitly labelled fallback in exploration notebooks, never behind an API route.

**`physical_price` stays empty.** Real schema, zero rows, by design. Do not seed it.

**No scraping.** Every source here has an official API or a published bulk file. If a task
seems to need scraping, stop and ask.

**Do not synthesise a "knowable" timestamp silently.** `release_ts` is derived as the Friday
of `report_date`'s own calendar week — not a flat `+3 days`, which is wrong for the rows
where CFTC shifts the as-of date back for a holiday. It still assumes that Friday's release
lands at 15:30 ET, which is wrong when that Friday is itself a US holiday. **Both the
derivation and the residual assumption must stay stated in the README *and* in a comment on
the line itself.** Same rule for the LTD approximation in `continuous(roll="ltd")`.

**Zero-volume months are exchange-computed settlements, not trades.** `no_trade_flag` must
survive from CSV to chart. Render them hollow/dashed. Use `type="linear"` on curves — a
monotone spline invents curvature between contracts that the data does not contain.

**Continuous series are a function of a convention.** `adjust` (`none`/`ratio`) and `roll`
(`volume`/`ltd`) are parameters, never hard-coded dates. When a rule cannot be applied —
no volume crossover in the window — return a `note` saying so rather than falling back
silently.

**ETL is idempotent.** Natural key plus `ON CONFLICT DO NOTHING`. Loaders print row counts
before and after; running twice must produce identical numbers.

---

## FastAPI conventions used here

- SQL lives in `app/queries.py`, never inline in a route.
- Every route declares `response_model=`; models in `app/models.py`.
- DB session via `Depends(get_db)` in `app/deps.py`.
- Handlers are plain `def`, not `async def` — psycopg is blocking, and an `async` handler
  making a blocking call stalls the event loop.
- Table and column names interpolated into SQL are hardcoded by the caller and never
  derived from request input.

## Running it

```bash
docker compose up db          # Postgres only; the app runs on the host
psql "$DATABASE_URL" -f db/schema.sql
python etl/load_futures.py && python etl/load_cot.py && python etl/load_fundamentals.py
uvicorn app.main:app --reload # docs at /docs
cd web && npm install && npm run dev
```

`etl/load_fundamentals.py` needs a free `FRED_API_KEY`. Without it, skip it — the COT and
futures panels still work, and the fundamentals panel should say the key is missing rather
than showing an empty chart.

## Definition of done for any change

- [ ] `/docs` still lists every route and each returns real data when clicked
- [ ] Term structure renders in contract order, not label order
- [ ] Moving the as-of slider visibly changes what is on screen
- [ ] "As first published" and "as revised" lines visibly diverge on HOUST
- [ ] Loaders re-run without changing row counts
- [ ] No new dependency, no scraped source, no seeded physical row

---
> Source: [GaryYao980/lbr-pit](https://github.com/GaryYao980/lbr-pit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-29 -->
