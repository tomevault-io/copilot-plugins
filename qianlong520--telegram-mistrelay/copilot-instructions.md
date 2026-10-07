## telegram-mistrelay

> This repository contains **MistRelay**, a high-concurrency Telegram media streaming and private netdisk cluster.

# MistRelay Assistant Guidelines & Safety Rules

This repository contains **MistRelay**, a high-concurrency Telegram media streaming and private netdisk cluster.

## 🚨 MANDATORY SAFETY RULES (READ BEFORE TAKING ANY ACTION)

1. **NEVER DELETE OR OVERWRITE BACKUPS**:
   - Directory `/root/MistRelay-dev/db/backups/` contains essential disaster recovery baselines (`backup_mistrelay_*.tar.gz`, `*.db`).
   - `backup_mistrelay_20260928_125911.tar.gz` contains 272 netdisk files, 31 downloads, 25 tenants, 24 protocol accounts, and 5 edge VPS nodes.
   - Deleting or truncating any backup is strictly prohibited under ALL circumstances (including "disk cleanups" or "freeing space").

2. **NEVER RUN MUTATING SQL ON PRODUCTION DATABASE**:
   - Production DB: `/root/MistRelay-dev/db/downloads.db` (or `/app/db/downloads.db`).
   - SQLite `PRAGMA foreign_keys=ON;` with `ON DELETE CASCADE` is enabled.
   - Deleting from `tg_media` cascades and deletes all records in `downloads` and `uploads`.
   - Never run `DELETE`, `DROP`, or `TRUNCATE` against the production database.

3. **MANDATORY TEST SANDBOXING**:
   - All tests in `tests/` MUST use an isolated temporary database (`tempfile.NamedTemporaryFile`) in `setUp` and clean up in `tearDown`.
   - Never direct `db.DB_PATH` to production in tests.
   - Restore all global singletons (`Var.MULTI_BOT_TOKENS`, `sys.modules`) in `tearDown`.

4. **REFERENCE SPECIFICATIONS**:
   - See `AGENTS.md` for comprehensive rules and runbooks.
   - See `SAFETY_RULES.md` for safety architecture.

## 🛠️ COMMON COMMANDS

- **Run all tests**: `/opt/jy-image-testenv/bin/pytest tests/`
- **Build frontend**: `cd /root/MistRelay-dev/web && npm run build`
- **Restart production container**: `docker restart mistrelay`
- **Check container health**: `docker ps --filter name=mistrelay`
- **Verify netdisk media count**: `curl -s http://127.0.0.1:8080/api/telegram/usage`

---
> Source: [qianlong520/Telegram_MistRelay](https://github.com/qianlong520/Telegram_MistRelay) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-07 -->
