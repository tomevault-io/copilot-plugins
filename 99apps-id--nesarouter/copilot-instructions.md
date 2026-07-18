## nesarouter

> Production **cannot** start with the development bootstrap password `nesa123456` (or `change-me`). You must set `NESA_ADMIN_PASSWORD` in `.env`.

# Agent / installer notes for NesaRouter

## Admin password on VPS installs

Production **cannot** start with the development bootstrap password `nesa123456` (or `change-me`). You must set `NESA_ADMIN_PASSWORD` in `.env`.

Do **not** silently pick a random password and leave the operator guessing why the login page no longer shows `nesa123456`.

Preferred options:

1. **Ask the operator** what bootstrap password to put in `.env`, then write that value.
2. If you generate a password, **print it once clearly** in the install summary and tell them to change it under **Routing → Password** after first login.
3. Never claim the app “auto-changed” the password — only `.env` / dashboard password change does.

Local development (`npm run dev`) may omit `NESA_ADMIN_PASSWORD` and use `nesa123456` with an on-screen hint.

## DATA_DIR / SQLite

Default database is `./data/nesa-router.sqlite`.

Do **not** leave `DATA_DIR` set to smoke/test folders (e.g. `data-test-version`, `data/smoke-*`) in the agent shell when starting the app. That makes login/session look broken (wrong DB, must-change loops, false 401s) while real data stays in `./data`.

Before diagnosing auth or middleware issues:

1. Confirm `DATA_DIR` is unset or points at the real data folder.
2. Confirm login is not locked (`npm run unlock-admin` should print a path under `.../data`).
3. Only then dig into middleware / session HMAC.

---
> Source: [99apps-id/nesarouter](https://github.com/99apps-id/nesarouter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-18 -->
