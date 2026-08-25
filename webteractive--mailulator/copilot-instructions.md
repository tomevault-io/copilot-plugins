## mailulator

> Guidance for AI coding agents when working in this repository.

# AGENTS.md

Guidance for AI coding agents when working in this repository.

## Product

`webteractive/mailulator` — self-hosted, Laravel-native email testing. One Composer package ships both sides:

- **Receiver** — ingest API + Vue 3 SPA (`/mailulator`) + isolated DB.
- **Driver** — Symfony Mailer transport registered as `mailulator`.

Either side is independently disable-able via `MAILULATOR_RECEIVER_ENABLED` / `MAILULATOR_DRIVER_ENABLED`. Two deployment shapes:
- **In-app** — receiver and driver both enabled in the same host app, no `MAILULATOR_URL`. Driver bypasses HTTP and writes directly to the `Default` inbox via `StoreIncomingEmail`.
- **Standalone** — dedicated receiver app; one or more sender apps post to it over HTTP using `MAILULATOR_URL` + per-inbox `MAILULATOR_TOKEN`.

## Architecture

Horizon/Telescope-style:

- `MailulatorServiceProvider` (package) registers routes, config, migrations, transport, broadcast channels.
- `MailulatorApplicationServiceProvider` (abstract) — host app extends it in `app/Providers/MailulatorServiceProvider.php` to define the gate, `canViewInbox`, and `manage` closures.
- Static `Mailulator::` configurator. Default gate: local environment only.
- `mailulator:install` publishes the stub provider + config + assets and seeds a protected `Default` inbox.

Frontend: Vue 3 + Pinia + Vue Router + shadcn-vue (reka-ui) + Tailwind (HSL tokens). Built with Vite 8; compiled `dist/` is committed and published to `public/vendor/mailulator`.

## Hard constraints

- **No SMTP.** Cross-app ingest is HTTP only (`POST /api/emails`); in-app mode bypasses HTTP and persists in-process via `StoreIncomingEmail`.
- **Token → inbox.** The bearer token alone routes mail. Stored as `sha256` hash. One token, one inbox.
- **Isolated `mailulator` DB connection.** Never touches the host app's primary DB. Registered in `register()` (not `boot()`) so migrations resolve. All migrations pin `protected $connection = 'mailulator'`.
- **Default inbox is protected.** Cannot be renamed or deleted. Last remaining inbox cannot be deleted.
- **Polling default, broadcast opt-in.** `MAILULATOR_REALTIME=polling` is dependency-free. `broadcast` requires Reverb/Pusher in the host app.
- **Attachments on a configurable disk** (`MAILULATOR_ATTACHMENTS_DISK`), streamed via `Storage::download` — never a public URL.

## Data model

- `inboxes` — name, hashed `api_key`, `retention_days`, `is_default`, `settings` (JSON), `last_used_at`.
- `emails` — `inbox_id`, from/to/cc/bcc, subject, html/text bodies, `headers` JSON, `read_at`.
- `attachments` — `email_id`, filename, mime_type, size, disk, path.

## Conventions

- Pint runs on every commit (`vendor/bin/pint --dirty`). PHPStan level 4.
- No FQCN inline — `use` imports at the top.
- No comments on obvious code; docblocks only for non-obvious params or array shapes.
- `Inbox::COLOR_REGEX` is the single source of truth for color validation.
- Mass-assignment: `$fillable = ['name', 'retention_days', 'settings']` only. `api_key`, `is_default`, `last_used_at` require `forceCreate` / `forceFill`.
- Tests: Pest 3 + Testbench. `:memory:` SQLite for both `testing` and `mailulator` connections.

## Common workflows

- `composer dev` — concurrent Vite + Testbench server (HMR).
- `composer serve` — Testbench server only (uses built assets).
- `composer fresh` — drops the workbench SQLite + rebuilds.
- `composer test` / `composer analyse` / `composer format`.
- `npm run build` — required before tagging if `resources/js/**` or `resources/css/**` changed; CI guards against stale `dist/`.

## Watch-outs

- `Mail::extend('mailulator', ...)` must run in `boot()`, not `register()`.
- `app.blade.php` reads `public_path('vendor/mailulator/.vite/manifest.json')`. Never use `@vite()` — that resolves to the host app's manifest.
- The published gate defaults to "local env only." Non-local installs without a customized gate → 403 for everyone.

---
> Source: [webteractive/mailulator](https://github.com/webteractive/mailulator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
