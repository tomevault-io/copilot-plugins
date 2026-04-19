## bee-mvp

> - Integration tests expect a vanilla Postgres 15 with user **postgres**.

## rule_type: auto

## trigger_dirs: [tests, scripts]

# Local Test Database 🐘

- Integration tests expect a vanilla Postgres 15 with user **postgres**.
- If `DB_HOST/DB_PORT` are unset the helper `scripts/start_test_db.sh`
  auto-starts a throw-away container `bee_test_pg` on a free port 55433-55633.
- You can start it manually with `make db-start`.
- Standard env vars: • `DB_HOST` (default `localhost`) • `DB_PORT` (assigned by
  script) • `DB_NAME` (default `test`) • super-user password = `postgres`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gmtfrombc) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
