## lobe-chat-pro

> Project directory structure overview


# LobeChat Project Structure

## Complete Project Structure

This project uses common monorepo structure. The workspace packages name use `@lobechat/` namespace.

**note**: some not very important files are not shown for simplicity.

```plaintext
lobe-chat/
├── apps/
│   └── desktop/
├── docs/
├── locales/
│   ├── en-US/
│   └── zh-CN/
├── packages/
│   ├── const/
│   ├── context-engine/
│   ├── database/
│   │   ├── src/
│   │   │   ├── models/
│   │   │   ├── schemas/
│   │   │   └── repositories/
│   ├── model-bank/
│   │   └── src/
│   │       └── aiModels/
│   ├── model-runtime/
│   │   └── src/
│   │       ├── core/
│   │       └── providers/
│   ├── types/
│   │   └── src/
│   │       ├── message/
│   │       └── user/
│   └── utils/
├── public/
├── scripts/
├── src/
│   ├── app/
│   │   ├── (backend)/
│   │   │   ├── api/
│   │   │   │   ├── auth/
│   │   │   │   └── webhooks/
│   │   │   ├── middleware/
│   │   │   ├── oidc/
│   │   │   ├── trpc/
│   │   │   └── webapi/
│   │   │       ├── chat/
│   │   │       └── tts/
│   │   ├── [variants]/
│   │   │   ├── (main)/
│   │   │   │   ├── chat/
│   │   │   │   └── settings/
│   │   │   └── @modal/
│   │   └── manifest.ts
│   ├── components/
│   ├── config/
│   ├── features/
│   │   └── ChatInput/
│   ├── hooks/
│   ├── layout/
│   │   ├── AuthProvider/
│   │   └── GlobalProvider/
│   ├── libs/
│   │   └── oidc-provider/
│   ├── locales/
│   │   └── default/
│   ├── server/
│   │   ├── modules/
│   │   ├── routers/
│   │   │   ├── async/
│   │   │   ├── desktop/
│   │   │   ├── edge/
│   │   │   └── lambda/
│   │   └── services/
│   ├── services/
│   │   ├── user/
│   │   │   ├── client.ts
│   │   │   └── server.ts
│   │   └── message/
│   ├── store/
│   │   ├── agent/
│   │   ├── chat/
│   │   └── user/
│   ├── styles/
│   └── utils/
└── package.json
```

## Architecture Map

- UI Components: `src/components`, `src/features`
- Global providers: `src/layout`
- Zustand stores: `src/store`
- Client Services: `src/services/` cross-platform services
  - clientDB: `src/services/<domain>/client.ts`
  - serverDB: `src/services/<domain>/server.ts`
- API Routers:
  - `src/app/(backend)/webapi` (REST)
  - `src/server/routers/{edge|lambda|async|desktop|tools}` (tRPC)
- Server:
  - Services(can access serverDB): `src/server/services` server-used-only services
  - Modules(can't access db): `src/server/modules` (Server only Third-party Service Module)
- Database:
  - Schema (Drizzle): `packages/database/src/schemas`
  - Model (CRUD): `packages/database/src/models`
  - Repository (bff-queries): `packages/database/src/repositories`
- Third-party Integrations: `src/libs` — analytics, oidc etc.

## Data Flow Architecture

- **Web with ClientDB**: React UI → Client Service → Direct Model Access → PGLite (Web WASM)
- **Web with ServerDB**: React UI → Client Service → tRPC Lambda → Server Services → PostgreSQL (Remote)
- **Desktop**:
  - Cloud sync disabled: Electron UI → Client Service → tRPC Lambda → Local Server Services → PGLite (Node WASM)
  - Cloud sync enabled: Electron UI → Client Service → tRPC Lambda → Cloud Server Services → PostgreSQL (Remote)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/vual)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/vual)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
