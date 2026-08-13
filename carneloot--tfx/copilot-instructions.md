## tfx

> - This project is an in-progress build and has no backward-compatibility requirements yet.

# Project Instructions

## Development Status

- This project is an in-progress build and has no backward-compatibility requirements yet.
- Prefer clean designs and direct breaking changes over compatibility shims, deprecation layers, or legacy migrations.
- Do not preserve existing APIs, database schemas, migration ledgers, or persisted development data unless the user explicitly requests it.

## Testing

- `pnpm --filter <package> test -- <file>` may still run the package's entire configured suite. For exact files, use `pnpm exec vitest run path/to/test.ts`.
- PostgreSQL integration tests skip unless `TEST_DATABASE_URL` is set or `RUN_TESTCONTAINERS=true`.
- Run the full PostgreSQL suite with `RUN_TESTCONTAINERS=true pnpm test:integration`.
- Run one PostgreSQL integration file with `RUN_TESTCONTAINERS=true pnpm exec vitest run --config vitest.integration.config.ts <path>`.
- Vitest `describe.sequential` is deprecated. Plain `describe` runs tests sequentially by default.
- TFX conversation rows use `(bot_id, chat_id, user_id)` rather than a generic `id` column. Update-deduplication rows use `(bot_id, update_id)`, and the prefixed table name is `<prefix>update_deduplication`.
- When adding an application migration, update every test that asserts exact migration ledger versions, names, or checksums.
- In non-interactive environments, reinstall dependencies with `CI=true pnpm install --frozen-lockfile`.

## Conversation Tests

- `ConversationBuilder.stay({ afterCommit })` emits `afterCommit`, then re-enters the current step and emits its prompt. Assert the error output exists; the final output may be the repeated prompt.
- Active conversations receive ordinary messages and commands before command routing. `/cancelar` is special-cased; cancel the active workflow before starting another command in E2E transcripts.

## Parallel Worktrees

- Do not symlink a writable worktree's `node_modules` to the root `node_modules`; `pnpm install` may recreate or remove shared dependencies.
- Prefer independently installed worktrees, or run final checks from the main worktree.
- For long-running or intercom-coordinated subagents, prefer manually managed Git worktrees over temporary automatic worktrees, which may be cleaned up after detachment.

## Effect Service Access

- Always yield Effect services into a named variable before calling their methods.
- Prefer `const repository = yield* Repository; const value = yield* repository.get(...)` over `yield* (yield* Repository).get(...)`.
- `HttpClient` is the only exception where inline service yielding is allowed.

## Pet Food Invariants

- Accessible-pet discovery is only a snapshot. Reauthorize owner or caregiver access inside each mutation transaction.
- Food replay identity is per `(botId, updateId, petId)`, allowing one update to feed multiple pets safely.
- Food timestamps must anchor to the Telegram message time, never the processing clock.

---
> Source: [carneloot/tfx](https://github.com/carneloot/tfx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
