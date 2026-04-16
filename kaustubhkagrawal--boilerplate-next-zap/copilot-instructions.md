## boilerplate-next-zap

> Use when adding a new feature to ensure consistent structure and integration.

### New Feature Setup
- Create subfolders for feature (e.g., `chat`):
  - `src/zap/components/chat/` for components.
  - `src/zap/hooks/chat/` for hooks.
  - `src/zap/actions/chat.action.ts` for actions.
  - `src/zap/schemas/chat.schema.ts` for Zod schemas.
  - `src/zap/db/schema/chat.sql.ts` for Drizzle schemas.
  - `src/zap/rpc/procedures/chat.rpc.ts` for oRPC.
  - `src/zap/stores/chat.store.ts` for Zustand.
- Add pages/routes in `src/app/(pages|api)/(public|auth-only)/chat/`.
- Update `src/zap/rpc/router.ts` with new procedure.
- Add feature flag in `src/data/settings.ts` (e.g., `FEATURE_FLAGS.enableChat = false`).
- Run `bun run format` and `bun run lint`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kaustubhkagrawal) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
