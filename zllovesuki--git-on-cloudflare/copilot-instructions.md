## git-on-cloudflare

> 1. When accessing or modifying the SQLite database within Durable Objects, do not access the database directly. Instead, add your function in `dal.ts`


1. When accessing or modifying the SQLite database within Durable Objects, do not access the database directly. Instead, add your function in `dal.ts`
2. Tests should use DAL functions whenever possible, unless it is absolutely necessary to have direct access (for example, checking for constraint violation enforcement).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zllovesuki) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
