## buzz

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

At the start of every session, read `.claude/memory/memory.md` to load project context.
After completing significant work (new patterns, architectural decisions, solved problems),
update `.claude/memory/memory.md`. Keep it under 300 lines — summarize when it grows.

---

## This defines the best practices to write backend code in the Frappe Framework

* Frappe Framework is a full-stack web application framework that contains all the necessary components for building modern web applications.
* It provides background workers using Redis, real-time updates using sockets, and a database layer using MariaDB.
* Bench is the official command-line tool for managing Frappe applications.

## Backend Development

### JSON & Request Handling

* Always use built-in functions for parsing JSON:

  * `frappe.parse_json` (handles dicts, lists, and JSON strings safely)

* Never use `json.loads` directly on request data.

* For outbound HTTP requests (calling external APIs), use:

  * `frappe.integration.utils.make_get_request`
  * `frappe.integration.utils.make_post_request`
  * `frappe.integration.utils.make_put_request`
  * `frappe.integration.utils.make_patch_request`

### Datatype Conversion & Utilities

* For converting datatypes (e.g. str → int, str → float, etc.) use built-in helpers:

  * `frappe.utils.data.cint`
  * `frappe.utils.data.cstr`
  * `frappe.utils.data.flt`
  * `frappe.utils.data.getdate`
  * `frappe.utils.data.get_datetime`

* `frappe.utils.data` contains most conversion and formatting helpers you will ever need:

  * date / datetime parsing
  * currency formatting
  * number formatting

* Do NOT create custom utility functions for these conversions.

* If unsure, ask before implementing.

### DocType Access Patterns

* When fetching an existing DocType, prefer:

  * `frappe.get_cached_doc`

* Use `frappe.get_doc` when:

  * creating a new document

  * To create a new doc go to bench console via bench --site sitename console and use frappe.new_doc("DocType") and then create the doc, don't create the doc via json as the validations doesn't run


### Optimization

* Don't use get_doc or get_cached_doc inside for loop it creates n+1 db problem use frappe.get_all with all the params required and then loop over that list

### Database Access

* Prefer ORM methods:

  * `frappe.get_all`
  * `frappe.get_list`
  * `frappe.db.get_value`

* Avoid raw SQL absolutely.


### Permissions & Security

* Always respect user permissions.
* Use `ignore_permissions=True` only when absolutely required and justified.

### Background Jobs & Performance

* For long-running or heavy operations, always use:

  * `frappe.enqueue`

* Never block request-response cycles with heavy business logic.

### Error Handling & Logging

* Use `frappe.throw` or specific exceptions like `frappe.ValidationError` for user-facing errors.
* Use `frappe.log_error` for unexpected or system-level exceptions.
* Avoid bare `except:` blocks.

### General Guidelines

* Prefer framework conventions over custom implementations.
* Keep business logic out of controllers where possible.
* Write readable, predictable, and maintainable code.



## Frontend Development

1. Always use async/await; avoid callback-based patterns and nested promises.

2. Use Frappe-provided APIs for server calls: `frappe.call` with `async: true`. Prefer Promise-based usage over callbacks.

3. Use Frappe's global JS helpers instead of native JS equivalents:
   * `cstr()` instead of `String()`
   * `cint()` instead of `parseInt()`
   * `flt()` instead of `parseFloat()`
   * `is_null()` instead of manual null/undefined/empty checks
   * `format_currency()` for currency formatting


## Crawling

Always use gemini as much as possible for getting the context, to get the help use gemini --help

For checking if the site works you can use the agent-browser use agent-browser --help to get the context for it


## Commands

### Frontend (Dashboard)
```bash
# dev server
yarn dev  # or: cd dashboard && yarn dev

# build for production
yarn build  # outputs to buzz/public/dashboard + buzz/www/dashboard.html

# lint/format frontend
cd dashboard && yarn lint
```

### Backend (Python)

Always run bench migrate after doctype schema changes.

```bash
# linting/formatting (via pre-commit)
pre-commit run --all-files

# run ruff directly
ruff check buzz/
ruff format buzz/

# install app to site
bench --site [site-name] install-app buzz
```

Use bench --help to see how to work with frappe bench, e.g. bench execute, bench console, etc. are very useful

### Testing

There are unit tests, run using bench run-tests. Site name is buzz.localhost, but if not found, ask user for it. The credentials are Administrator/admin.

* To test in UI, use agent-browser.
* For frontend changes use :8080 since yarn dev server is running.
* Use in headed mode unless specified

## Architecture

**Three-tier stack:**
1. **Backend**: Frappe Framework (Python) - DocTypes, API, permissions, scheduler
2. **Dashboard**: Vue 3 + FrappeUI + Vite - attendee/sponsor/checkin UI

**Core entity**: `Buzz Event` DocType drives everything (tickets, sponsors, schedule, payments).

**Main modules** (inside `buzz/`):
- `events/` - Event, Venue, Category, Talks, Sponsors, Check-ins
- `ticketing/` - Bookings, Tickets, Add-ons, Cancellations, Coupons
- `proposals/` - Talk Proposals, Sponsorship Enquiries
- `buzz/` - Settings, Custom Fields
- `api.py` - whitelisted API methods for dashboard
- `payments.py` - integration with frappe/payments app

**Frontend structure** (inside `dashboard/`):
- `src/pages/` - route components (BookTickets, TicketDetails, CheckInScanner, etc)
- `src/components/` - BookingForm, dialogs, shared UI
- `src/composables/` - reusable logic (useTicketValidation, usePaymentSuccess, etc)
- `src/data/` - frappe-ui resources for API calls
- Vite builds to `buzz/public/dashboard/`, router base is `/dashboard`

**Key flows:**
- Booking: load event data → fill form → create booking → generate payment link → on payment auth → submit booking → generate tickets + QR + email
- Ticket actions: transfer, cancel, change add-on (window checks from Buzz Settings)
- Sponsorship: enquiry → approval → payment link → payment auth → create sponsor record
- Check-in: scan QR → validate → create check-in record (requires Frontdesk Manager role)

**Integrations:**
- `frappe/payments` required for payment gateways
- `buildwithhussain/zoom_integration` optional for webinar creation/registration

## Key Paths for Common Tasks

**Booking changes**: `buzz/api.py`, `buzz/ticketing/doctype/event_booking/`, `dashboard/src/components/BookingForm.vue`

**Ticket lifecycle**: `buzz/ticketing/doctype/event_ticket/`, `dashboard/src/pages/TicketDetails.vue`

**Sponsorships**: `buzz/proposals/doctype/sponsorship_enquiry/`, `dashboard/src/pages/SponsorshipDetails.vue`

**Check-in**: `buzz/api.py` (validate_ticket_for_checkin, checkin_ticket), `dashboard/src/pages/CheckInScanner.vue`

**Event config**: `buzz/events/doctype/buzz_event/`

**Reports**: `buzz/events/report/` and `buzz/ticketing/report/`


## Joining or creating report
 "Never write `frappe.db.sql` again"
===========================================================

1.  **Ban `frappe.db.sql` in new code**
    *   Add a pre-commit rule or CI step that greps for `\.db\.sql` and fails the build.
    *   Legacy code => wrap in `frappe.db.sql("...", as_dict=1)` and add a `# TODO-QB` comment so the next refactor is trackable.

2.  **Use the typed entry point**
    ```python
    from frappe.query_builder import DocType, Field
    from frappe.query_builder.functions import Count, Sum, Coalesce, Date
    ```
    Never `import pypika` directly; the `frappe.qb` namespace already returns the correct `MariaDB/PostgreSQL` dialect.

3.  **Parameterise, never interpolate**
    ```python
    # Bad
    frappe.db.sql(f"... {user_input}")  # injection bomb
    # Good
    frappe.qb.from_(...).where(table.field == user_input)  # auto-escaped
    ```

4.  **Prefer joins over N+1**
    ```python
    so = DocType("Sales Order")
    si = DocType("Sales Invoice")
    query = (
        frappe.qb.from_(so)
        .left_join(si)
        .on(so.name == si.sales_order)
        .select(so.name, si.name)
        .where(so.customer == customer)
    )
    ```
    One round-trip, no loops.

5.  **Sub-queries > raw SQL strings**
    Need *"latest row per group"*?
    ```python
    latest = (
        frappe.qb.from_(si)
        .select(si.name)
        .where(si.sales_order == so.name)
        .orderby(si.creation, order=Order.desc)
        .limit(1)
    )
    query = frappe.qb.from_(so).where(so.name == latest)
    ```
    Keeps everything composable and dialect-agnostic.

6.  **Use `case` for conditional aggregates**
    ```python
    from frappe.query_builder.functions import Case
    paid_amt = Sum(
        Case()
        .when(si.status == "Paid", si.grand_total)
        .else_(0)
    )
    ```

7.  **Respect Frappe field casing**
    *   SQL column: `grand_total`
    *   Frappe field: `grand_total`
    *   No back-ticks needed; QB adds the correct quotes per DB.

8.  **Use `as_dict=True` or ORM objects**
    ```python
    rows = query.run(as_dict=True)        # list[dict]
    docs = query.run(as_dict=False)       # list[tuple]
    obj  = frappe.get_doc("Doctype", pk)  # when you need the full DocType hooks
    ```

9.  **Pagination with `limit_page_length` and `limit_start`**
    ```python
    query = query.limit(limit_page_length).offset(limit_start)
    ```
    Same pattern the REST API uses.

10. **Index-friendly WHERE order**
    Put indexed columns first (`company`, `customer`, `status`) so MariaDB/PostgreSQL can use composite indexes.

11. **Avoid `SELECT *` in reports**
    Explicit list of fields keeps wire-size small and prevents breaking changes when new fields are added.

13. **Cache heavy aggregations**
    ```python
    @frappe.whitelist()
    @redis_cache(ttl=300)
    def get_dashboard_stats(company):
        inv = DocType("Sales Invoice")
        total = frappe.qb.from_(inv).select(Sum(inv.grand_total)).where(inv.company == company).run()
        return total[0][0] or 0
    ```


Quick migration template
----------------------

Legacy:
```python
rows = frappe.db.sql("""
    select name, grand_total
    from `tabSales Invoice`
    where customer = %s
    and docstatus = 1
""", customer, as_dict=1)
```

QB equivalent:
```python
si = DocType("Sales Invoice")
rows = (
    frappe.qb.from_(si)
    .select(si.name, si.grand_total)
    .where((si.customer == customer) & (si.docstatus == 1))
    .run(as_dict=True)
)
```


### Report Patterns
- Entry: `def execute(filters): return get_columns(), get_data(filters)`
- QB imports: `from frappe.query_builder import DocType` + `functions.Sum, Case, Count`
- Build lookup maps first, then loop + merge (avoid N+1)
- Caching: `@redis_cache(ttl=seconds)` for conversion factors

### Token-Saving Workflow
- Default to `/model haiku` for routine edits, `/model sonnet` for moderate tasks
- Use `/model opus` ONLY for architecture, debugging complex issues
- Use `/compact` after completing each subtask
- Use `gemini -p "prompt"` via stdin to read/summarize files without burning Claude tokens
- Scope tasks narrowly: one feature/fix per session
- Dump progress to `.claude/memory/scratch.md` before session ends
- At session START: read `.claude/memory/scratch.md` — if it has content, resume from there
- At session END (or when user says "dump progress"): fill in scratch.md with current task state
- After resuming from scratch.md, clear it once the task is complete


### Variable and Function naming convention

- always use full names for variables don't use abbreviations for ex: use
"for row in rows" instead of "for r in rows"
proxy_sku = DocType("Proxy SKU") instead of ps = DocType("Proxy SKU")
- avoid starting python functions with underscore "_" unless it's a private version behind a whitelisted function (e.g. `get_dial_codes` (whitelisted) -> `_get_dial_codes` (cached logic))
- use camelCase in JS and follow the surrounding code style in the project
- always put imports at the top of the file, never inside functions

## Notes

- Read `ARCHITECTURE.md` for comprehensive details on data model, API surface, flows

---
> Source: [bwhtech/buzz](https://github.com/bwhtech/buzz) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-01 -->
