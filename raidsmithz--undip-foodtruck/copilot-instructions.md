## undip-foodtruck

> Private automation that grabs Undip Foodtruck coupons via SSO and delivers them via WhatsApp. Three runtimes share one MariaDB: a Node WhatsApp bot, a Python coupon-taker, and a Next.js admin (currently boilerplate).

# Claude project instructions

Private automation that grabs Undip Foodtruck coupons via SSO and delivers them via WhatsApp. Three runtimes share one MariaDB: a Node WhatsApp bot, a Python coupon-taker, and a Next.js admin (currently boilerplate).

## Architecture you should know before editing

The Node bot was 1418 lines in `src/chat_bot/bot.js` until commit `508d92b`. Now it's a routing layer:

```
src/chat_bot/
  bot.js              ← whatsapp-web.js client + stealth shim + cron.start()
  router.js           ← single route(msg, client, deps) entry point
  state.js            ← pending_action FSM helpers
  helpers.js          ← location/status names, time predicates, sanitization
  views.js            ← every reply string lives here, as functions returning Indonesian text
  cron.js             ← schedule.scheduleJob registrations + sendCoupons / doLoginAccounts / etc
  commands/
    index.js          ← ordered registry of text commands + admin + image + pending handlers
    ufood.js          ← ufood / commands (single canonical orientation; help/alur/aturan all redirect)
    daftar.js         ← ufood daftar — auto-grants 2x trial on first registration + spawns auto-login
    akun_list.js      ← ufood akun / ufood akun N
    akun_lokasi.js    ← snapshot edits, no `ya` prompt
    akun_submit.js    ← snapshot edits, capacity-checked
    akun_beli.js      ← QRIS image + sets pay_sso_id
    akun_ganti.js     ← snapshot edits, status_login reset to 0
    akun_hapus.js     ← only command that still uses `ya` (besides ping)
    status.js         ← ufood status
    subscribe.js      ← actually toggles wa_messages.subscribed
    ping.js           ← admin handoff, 3h auto-expire
    admin.js          ← !login, !kupon, !kirim, payment confirm (ya N / tidak), ping resolution (sudah)
    image.js          ← refuses uploads without prior `ufood akun N beli` (no admin spam)
```

Each command exports `{ name, match(body, msg), handle({msg, params, client, deps}) }`. Commands that put pending state also export `resolveConfirm({msg, pending, client, deps})`.

`router.js` runs in this order:
1. Block status@broadcast / group messages
2. New user (`waMsgIsBlocked` returns -1) → welcome + create row
3. Currently blocked → drop (admin handles directly)
4. Block just expired (3h passed) → notify + drop this message
5. Image/document → `commands.image.handle`
6. Sender is admin → `commands.admin.handle` first, fall through if it returns null
7. Pending action exists → matching `commands.pending[prefix].resolveConfirm`
8. Text command match (first hit in `commands.text[]`)
9. Fallback → `views.unknownCommand()`
10. Errors anywhere → `errorLogAdd` + `views.commandError()` reply with ❌

## Conventions

- **Replies are always Indonesian, WhatsApp-style.** Use `*bold*`, `_italic_`, `> quote`, `⏳ ✅ ❌` emoji where it fits.
- **Every reply string lives in `views.js`** as a named function. Don't inline strings in handlers.
- **No `ya` confirmation for non-destructive actions.** Snapshot the change directly with `oldVal → newVal`. Only `hapus` and `ping` keep `ya`/`batal`.
- **Pending state format:** `prefix` or `prefix:payload` (e.g. `delete:42`, `ping`). Stored as VARCHAR(64) in `wa_messages.pending_action` with a 5-min TTL via `pending_action_at`.
- **WhatsApp Web version: never override.** Letting `whatsapp-web.js` use its bundled default is the difference between login working and "Try Again" — see [memory: WhatsApp Try Again fix](../../../Users/nanda/.claude/projects/c--Projects-undip-foodtruck-mallocation-com/memory/feedback_whatsapp_web_stealth.md).
- **Stealth shim must stay at the top of `bot.js`** — `puppeteer-extra` + `puppeteer-extra-plugin-stealth` injected into `require.cache["puppeteer"]` before whatsapp-web.js is required.
- **Snap chromium on aarch64.** `CHROME_EXECUTABLE_PATH` and `CHROMIUM_EXECUTABLE_PATH` both point at `/usr/bin/chromium-browser`. The puppeteer-bundled `linux_arm-*` binary is actually x86-64 and won't run on the Oracle Cloud ARM host.
- **Headless mode is `false` on the server,** wrapped by `xvfb-run` via `scripts/start.sh`. WhatsApp Web's frame detached errors go away with a virtual display.

## Database (`sql_undip_foodtruck` on MariaDB 10.11)

Schema lives in **two** places — keep them in sync:
- Node: `src/models/tables.js` (Sequelize)
- Python: `python/database.py` (SQLAlchemy) — only knows `registereds`, `sso_accounts`, `taken_coupons`. The Python side does NOT touch `wa_messages` or `error_logs`.

Tables:
- `registereds` — wa_number → comma-separated sso_ids (max 3) + `pay_sso_id`
- `sso_accounts` — encrypted email/pwd, location, quota, submit toggle, `status_login`
- `wa_messages` — per-user FSM: `subscribed`, `pending_action`, `pending_action_at`, `blocked`, `blocked_at`, `free_trial`, `last_messages`, plus legacy `confirmation`/`rules_accepted` no longer read by new code
- `taken_coupons` — daily attempt log
- `error_logs` — append-only handler errors (chunk-4 telemetry)

`wa_messages.wa_number` has a UNIQUE index — use `WAMessages.findOrCreate` not raw create-after-find. AES-256-CBC keys (`ENCRYPTION_KEY`/`ENCRYPTION_IV`) MUST match between Node and Python or stored emails/passwords become unreadable across runtimes.

## Testing

```
node scripts/test-router.js
```

25 scenarios. Mocks the whatsapp-web.js Client, runs against the live DB with `_test_router_smoke@c.us`, cleans up at the end. `deps.skipAutoLogin = true` keeps puppeteer from firing during tests.

When changing handler behavior, add or update an assertion here. Exit 0 on all-pass.

## Deploy loop

1. Edit code locally on Windows.
2. `git add` specific files (NOT `git add -A` — see [purge incident](#deployments-and-secrets)) → `git commit` → `git push origin main`.
3. SSH afit: `cd ~/undip-foodtruck && git pull --ff-only && pm2 restart ufood-bot`.
4. `pm2 logs ufood-bot --lines 30 --nostream | grep -iE "ready|error"` to confirm.

afit (Oracle Cloud aarch64 prod) details: see memory `reference_afit_server.md`. PM2 systemd unit auto-starts on boot.

## Deployments and secrets

- **Never `git add -A`** — sweeps in untracked files. Use specific paths or `git add <dir>/`.
- The repo has been cleaned of two leaked files via `filter-branch` + force-push (commit history rewritten on `2026-05-03`). Don't be surprised by the unusual commit hashes around `99edc93` / `43280af`.
- `.env` and `python/config.py` are gitignored. `.env.example` documents every key.
- Production secrets (DB password, IPRoyal proxy) live only in `~/undip-foodtruck/.env` on afit. Mode 600.

## What NOT to change without thought

- **`webVersion` / `webVersionCache` in `bot.js` Client config** — leaving them out is correct.
- **`registeredCountSSOIDS` return type** — must return `0` (not `[]`) when no row exists; `daftarFirstAccountWithTrial` does strict `=== 0` check.
- **`pay_qris_me.jpg` path** at `./src/chat_bot/pay_qris_me.jpg` — sent literally by `akun_beli.js`. Don't rename without updating `QRIS_PATH`.
- **Cron schedule timing** — coupon delivery runs every minute 10:05–11:05 weekdays because the Python script writes coupons at varying times during 10:00–11:00.
- **The `ya N` (free-trial=0) branch in `admin.js`** — admin can still grant free trial via WA reply, even though the new flow auto-grants on first registration. Keep this as an admin override path.

## Style for new code

- Indonesian-style comments are fine alongside English.
- No JSDoc; types are clear from naming.
- Prefer many small `commands/*.js` files over fewer fat ones.
- Add to `views.js` first, then call from the handler — never inline strings.
- New columns: add to `tables.js` Sequelize model; `sync({ alter: true })` at boot picks them up. For Python-relevant columns, mirror in `python/database.py`.

---
> Source: [raidsmithz/undip-foodtruck](https://github.com/raidsmithz/undip-foodtruck) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-21 -->
