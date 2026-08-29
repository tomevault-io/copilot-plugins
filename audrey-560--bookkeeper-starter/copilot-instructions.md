## bookkeeper-starter

> A productized bookkeeping system: per-client Google Sheets double-entry ledgers +

# Bookkeeper — AI Bookkeeping OS (multi-client)

A productized bookkeeping system: per-client Google Sheets double-entry ledgers +
Drive PDF archives, operated by deterministic tools behind human approval gates.
Every command takes `--client <slug>`; with no flag it acts on `DEFAULT_CLIENT`
(`sandbox-demo`, the bundled example tenant). New here? Start with
[README.md](README.md). Docs: [docs/PRODUCT.md](docs/PRODUCT.md) (what/who),
[docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) (how), [docs/CONTROLS.md](docs/CONTROLS.md)
(trust model), [docs/LIMITATIONS.md](docs/LIMITATIONS.md) (honest boundaries).

## Layout

| Path | What |
|---|---|
| `clients/<slug>/` | Tenant: `profile.json` (entity, money map, tax, invoicing), `.env` (secrets, 600), `customers.json`, `inbox/` (statement drop), `state/` (import ledger, posted index, learned rules, exceptions, close states, audit log) |
| `tools/` | Deterministic Python. All take `--client` (default `sandbox-demo`) via `client_context.py` |
| `tools/importers/` | Statement adapters: rbc_csv, wise_csv, stripe_csv, generic_csv (+ stubs) |
| `workflows/` | SOPs: monthly_close, import_statements, process_expenses, exception_review, corrections, onboarding, generate_invoice |
| `jurisdictions/` | Tax packs: **ca-on (production)**, us-federal + generic-vat (stubs) |
| `institutions/` | Registry + per-bank download instruction cards |
| `docs/` | Product documentation |
| `tests/` | `contract_test.sh` (run after ANY change), personas, fake sheets client |

## Key invariants (do not break)

1. **One Journal writer**: every JE goes through `post_je()` — COA check, Dr=Cr,
   period lock, dedupe key, audit log. Never append to Journal any other way.
2. **Additive-only sheet schema**: Invoices A–R frozen; new columns only to the right.
   Journal Q = Dedupe Key, Expenses Q = Txn ID.
3. **Contract surface** (the skills depend on it): `mark_invoice_paid.py` flags,
   `clients/<slug>/customers.json` via `ctx.customers_path`, `sheets_client.py` path.
   Run `./tests/contract_test.sh` after any change.
4. **No silent guessing**: uncertainty → `exceptions_queue`, never the books.
5. **Confirm every cloud write**: dry-run → user OK → `--commit`.
6. **Locked periods are immutable**: corrections per [workflows/corrections.md](workflows/corrections.md).
7. Judgment lives in workflows/packs/profiles — never hardcoded in tools.

## Skills (user-invoked)

`/bk-onboard` (new client interview → workspace + sheet) · `/bk-import` (ingest
inbox + coverage report) · `/bk-status` (cross-client dashboard + deadlines) ·
`/invoice [customer]` (end-to-end invoice) · `/monthly-close [client] [YYYY-MM]`
(full close → period lock).

They live in `.claude/skills/` and load automatically when you open this project
in Claude Code.

## Per-client facts

Entity details — fiscal year end, tax IDs, filing deadlines, accountant, bank
accounts, agencies — live in that client's `clients/<slug>/profile.json`, not here.
`/bk-onboard` collects them. Live open items belong at the bottom of
[workflows/monthly_close.md](workflows/monthly_close.md) — update there, don't duplicate.

---

# Agent Instructions (WAT framework)

**Workflows** (markdown SOPs in `workflows/`) hold the judgment. **Tools**
(Python in `tools/`) hold deterministic execution. **You** (the agent) orchestrate:
read the workflow, run the tools in order, surface decisions to the user, recover
from errors, and improve the system as you learn.

Operating rules:
- Check `tools/` before building anything new; reuse beats rewriting.
- When something fails: read the full error, fix the tool, verify, then record the
  lesson in the relevant workflow. Ask before re-running paid API calls.
- Don't create or overwrite workflows without asking.
- Ask before every external write (Sheets, Drive, email, APIs) — one dry-run
  preview per write, then commit on explicit OK.
- Deliverables live in cloud services; local files are processing scratch.
  `.tmp/` is disposable; `clients/*/state/` and `clients/*/inbox/processed/` are NOT.
- Every failure is a chance to make the system stronger: identify, fix, verify,
  document, move on.

---
> Source: [audrey-560/bookkeeper-starter](https://github.com/audrey-560/bookkeeper-starter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
