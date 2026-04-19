## markflow

> This is a typescript project using fastify, next-app with drizzle.

# Project Context

This is a typescript project using fastify, next-app with drizzle.
It is a monorepo with workspaces: @markflow/db (packages/db), @markflow/editor (packages/editor), @markflow/api (apps/api), @markflow/demo (apps/demo), @markflow/web (apps/web), markflow-r2-uploader (apps/worker).

The API has 64 routes. See .codesight/routes.md for the full route map with methods, paths, and tags.
The database has 15 models. See .codesight/schema.md for the full schema with fields, types, and relations.
The UI has 82 components. See .codesight/components.md for the full list with props.
Middleware includes: auth, cors, custom.

High-impact files (most imported, changes here affect many other files):
- apps/web/lib/api.ts (imported by 40 files)
- apps/api/src/utils/errors.ts (imported by 32 files)
- apps/api/tests/helpers/setup.ts (imported by 31 files)
- apps/api/tests/helpers/factory.ts (imported by 30 files)
- apps/web/stores/toast-store.ts (imported by 21 files)
- apps/web/stores/workspace-store.ts (imported by 19 files)
- apps/api/src/middleware/auth.ts (imported by 17 files)
- apps/web/lib/types.ts (imported by 17 files)

Required environment variables (no defaults):
- CI (apps/web/playwright.config.ts)
- E2E_BASE_URL (apps/web/playwright.config.ts)
- E2E_USER_EMAIL (apps/web/tests/e2e/document-management.spec.ts)
- E2E_USER_PASSWORD (apps/web/tests/e2e/document-management.spec.ts)
- E2E_WORKSPACE_NAME (apps/web/tests/e2e/team-management.spec.ts)
- E2E_WORKSPACE_SLUG (apps/web/tests/e2e/document-management.spec.ts)
- EMAIL_FROM (apps/api/src/utils/email.ts)
- FRONTEND_URL (apps/api/src/utils/email.ts)
- NEXT_PUBLIC_API_URL (apps/web/app/(app)/[workspaceSlug]/doc/[docId]/layout.tsx)
- NEXT_PUBLIC_R2_WORKER_URL (apps/web/lib/image-upload.ts)
- NEXT_PUBLIC_SITE_URL (apps/web/app/layout.tsx)
- NODE_ENV (apps/api/src/routes/auth.ts)
- R2_UPLOAD_SECRET (apps/api/src/routes/upload-token.ts)
- VITEST (apps/api/src/index.ts)

Read .codesight/wiki/index.md for orientation (WHERE things live). Then read actual source files before implementing. Wiki articles are navigation aids, not implementation guides.
Read .codesight/CODESIGHT.md for the complete AI context map including all routes, schema, components, libraries, config, middleware, and dependency graph.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/claude-code-expert) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-14 -->
