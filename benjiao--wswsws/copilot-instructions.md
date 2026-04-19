## wswsws

> Never run migrations unless user explicitly requests


# Migration Execution Policy

- Never run `makemigrations`, `migrate`, or any migration-generation command automatically.
- Never apply database migrations automatically.
- If schema/model changes require migrations, stop and ask the user to run them manually.
- You may mention the exact command(s) the user should run, but do not execute them unless explicitly requested.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/benjiao) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
