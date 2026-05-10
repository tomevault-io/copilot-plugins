## wfcs-auction

> NEVER `beginTransaction()` or `commit()` — autocommit only.

# WFCS Auction

---

# GLOBAL RULES — apply to every project

## GALVANI — HARD RULES (break these = silent bugs or crashes)

NEVER `beginTransaction()` or `commit()` — autocommit only.
NEVER `new Database()` — always `Database::getInstance()`.
NEVER `LIMIT ?` — always `'LIMIT ' . (int)$n`.
NEVER `true`/`false` in SQL — use `1`/`0`.
NEVER `PDO::ATTR_EMULATE_PREPARES false` — emulated prepares must stay on or DATE/TIME values corrupt.
NEVER auth checks in constructors — check in each method.
NEVER multi-step writes across requests — do dependent writes in one handler.
DELETE request bodies are stripped — pass params via `$_GET` / query string only.
CSRF token via `csrfUrl()` → `_csrf_token` query param (headers dropped on multipart/DELETE).
CLI scripts: raw mysqli only — copy db-init.php pattern. No framework classes, no Dotenv.
NEVER use system `php` to generate password hashes — Galvani's PHP build is incompatible with system PHP bcrypt; hashes will silently fail `password_verify()`. Always generate hashes via Galvani: `./galvani auction/hash.php` where `hash.php` runs `echo password_hash('secret', PASSWORD_DEFAULT);`.
mysql CLI direct SQL: always `--skip-ssl` — `--ssl-mode=DISABLED` does not exist on this build; omitting it causes an SSL error. Pattern: `echo "SQL;" | mysql --socket=../data/mysql.sock -u root --skip-ssl [dbname]`

## PHP — HARD RULES

ALWAYS `ob_start()` as first line of `index.php` (after `declare`) — any output before `setcookie()`/`header()` silently drops them. Standard PHP buffering.
NEVER `rowCount()` to verify UPDATE success — trust `execute()` bool. With emulated prepares, `rowCount()` returns 0 when rows matched but weren't changed. Standard PDO behaviour.

## ARCHITECTURE — HARD RULES

All SQL in Repositories only — never `$this->db->` in a Service or Controller.
Services call Repositories — no direct DB access.
Controllers orchestrate Services — return views or JSON only.
NEVER write a controller that also does SQL.
NEVER write a service that also does SQL.
All controllers extend base `Controller` — always call `parent::__construct()`.

## DRY — HARD RULES

BEFORE writing any function or component — search the codebase for an existing implementation first.
NEVER duplicate logic that already exists. Find it, use it, extend it.
BEFORE writing any PHP function — check `app/Helpers/`, `app/Services/`, `app/Repositories/`.
BEFORE writing any JS function — check existing scripts in the current view and shared partials.
BEFORE writing any CSS — check if a Tailwind utility already does it.
BEFORE writing any HTML component — check `app/Views/atoms/` and `app/Views/partials/`.
NEVER use underscore-prefixed filenames (`_foo.php`) — those signal dead/backup code.

## JS — HARD RULES

NEVER `innerHTML`, `outerHTML`, `insertAdjacentHTML` — use `createElement` / `textContent` / `appendChild` / `replaceChildren`.
NEVER `alert()` / `confirm()` / `prompt()` — use Popover API.
NEVER JS fetch for server-side work — use HTML `<form method="POST">`.

## UI — HARD RULES

NEVER `style=""` attributes — Tailwind classes or `<style>` blocks only.
NEVER uppercase labels — uppercase is nav only.
NEVER numeric IDs in URLs — slugs only.
NEVER AI-generated icons — SVG only (Heroicons or Feather, or none).
NEVER `$` or any currency symbol other than `£` — we're in the UK.
Mobile-first always.
Numeric columns in sortable tables: show `0` not `—` (breaks sort).

---

# PROJECT RULES — WFCS Auction specific

## Stack
PHP MVC · MariaDB · TailwindCSS · Vanilla JS · Galvani runtime
Run: `./galvani` from `/Users/waseem/Sites/www/` · App at `/auction/` · http://localhost:8080/auction/
Core classes in `core/` (Controller, Database, Router, JWT) — not `app/`.

## Socket Paths
NEVER use `getcwd()` for socket paths — Galvani CLI sets CWD to the script's directory, not git root.
Always use `__DIR__`-relative paths — resolved from the file's location, not CWD.

`config/database.php` (and any file in `auction/config/`): `dirname(__DIR__, 2) . '/data/mysql.sock'`
CLI scripts in `auction/scripts/`: `dirname(__DIR__, 2) . '/data/mysql.sock'`
`index.php` (web entry, in `auction/`): `dirname(__DIR__) . '/data/mysql.sock'`

## Routes
All routes registered via `core/Router.php`.
NEVER hardcode `"auction"` in paths — use `$basePath`.

## Layouts — use the right one
- `layouts/public.php` — all public-facing pages
- `layouts/admin.php` — all admin pages
- `layouts/auth.php` — login, register, password reset
- `layouts/auctioneer.php` — auctioneer live panel
- `layouts/projector.php` — projector display

## Atoms — check before writing
These exist in `app/Views/atoms/` — use them, don't recreate them:
- `popover-shell.php` — Popover API wrapper with standard header/footer — use for ALL dialogs
- `button.php` — all button variants
- `input.php` / `label.php` / `select.php` / `textarea.php` / `toggle.php` — all form elements
- `badge.php` — status, category, role badges
- `stat-card.php` — icon + label + value card
- `item-card.php` — auction item card (home / my-bids)
- `event-card.php` — auction event card
- `page-header.php` — page title + subtitle + action buttons
- `breadcrumb.php` — breadcrumb trail
- `alert.php` — inline info/success/warning/error box
- `empty-state.php` — empty list placeholder
- `table-wrapper.php` — admin table chrome (border, rounded, overflow)
- `file-upload.php` — drag-and-drop file zone

## Commands
```bash
# From /Users/waseem/Sites/www/
./galvani                                             # start server
./galvani auction/db-init.php                        # wipe + reimport schema + seeds
./galvani auction/run-tests.php                      # run tests (shebang-free wrapper — vendor/bin/phpunit has shebang Galvani can't strip)
```

## Reference files (read on demand)
- `docs/developer/README.md` — developer guide
- `docs/plans/` — feature plans and design docs
- `docs/plans/view-partials-spec.md` — full atom/partial component specification

---
> Source: [waseemsadiq/wfcs-auction](https://github.com/waseemsadiq/wfcs-auction) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-27 -->
