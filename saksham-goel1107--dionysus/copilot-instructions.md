## prisma

> - Keep schema changes minimal and additive; avoid destructive changes without migrations

- Keep schema changes minimal and additive; avoid destructive changes without migrations
- Prefer explicit selects; avoid `include` of large relations unnecessarily
- Use transactions for multi-step writes; ensure idempotency for webhooks
- Use the vector extension where relevant; store embeddings as `Unsupported("vector")` per schema
- Validate inputs with `zod`; never trust client payloads
- Consider read replicas (`src/lib/read-replica*.ts`) for heavy read paths

---
> Source: [Saksham-Goel1107/Dionysus](https://github.com/Saksham-Goel1107/Dionysus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
