## v7-shop-repositories

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

请使用中文回答

## Repository layout

This is a polyglot monorepo for **V7 Shop** — a multi-tenant, multi-website e-commerce platform. Each top-level directory is an independently built/deployed project:

| Directory          | Stack                                             | Role                                                                                               |
| ------------------ | ------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| `v7-shop-services` | Java 17 + Spring Boot 3.3 (Gradle, Kotlin DSL)    | Backend API. Multi-module: account, admin, common, dao, entrance (the runnable app), core.         |
| `v7-shop-admin`    | Vue 3 + Vite 6 + TypeScript + Element Plus (pnpm) | Admin / total-backend SPA (`shop-vite`, a `admin-plus`-derived framework).                         |
| `v7-shop-mall`     | Nuxt 4 + Nitro + UnoCSS (pnpm)                    | Storefront. SSR with its own Nitro server layer that talks directly to MySQL/Redis.                |
| `v7-shop-docs`     | Docusaurus 3 (React 19)                           | Documentation site.                                                                                |
| `turboflow-bridge` | Chrome MV3 extension (vanilla JS)                 | Standalone browser extension for image translation via Google Flow; unrelated to the shop runtime. |

Each subproject has its own dependency install and build — there is **no** root-level workspace orchestration. `start-dev.bat mall|admin` launches a frontend dev server on Windows with a local HTTP proxy (`127.0.0.1:8800`) pre-set.

## Common commands

### Backend (`v7-shop-services`, run from that directory)

```bash
./gradlew :v7-shop-entrance:bootRun        # run the app (entrance is the only bootable module)
./gradlew build                            # build all modules
./gradlew :v7-shop-admin:test              # run one module's tests
./gradlew test --tests "GeminiTranslateServiceTest"   # run a single test class
```

- `v7-shop-entrance` is the aggregator/main app (`V7ShopEntranceApplication`); all other modules disable `bootJar`. Most ad-hoc/integration tests live under `v7-shop-entrance/src/test/java` as plain runnable classes.
- Config is **not** committed: copy `application.yml.example` → `application.yml` and supply env vars (MySQL ×2, Redis, S3, Aliyun OSS, JWT secret, config-center/cloak URLs). `application.yml`, `application-*.yml` are gitignored.

### Admin (`v7-shop-admin`, pnpm)

```bash
pnpm dev                  # dev server (mode=development)
pnpm build                # vue-tsc typecheck + vite build
pnpm build:ht / build:xyz # build a specific white-label brand (uses vite.config.website.ts)
pnpm vue-tsc              # typecheck only
pnpm lint:eslint          # eslint --fix
pnpm lint:stylelint       # stylelint --fix
pnpm template             # scaffold via plop
```

- Env files are per-brand (`.env`, `.env.ht`, `.env.xyz`, `.env.local`); `.env` and `.env.local` are gitignored.
- Three vite configs: `vite.config.ts` (default), `vite.config.dev.ts` (https + bundle report), `vite.config.website.ts` (white-label website builds, output zipped to `dist/website.zip`).

### Mall (`v7-shop-mall`, pnpm)

```bash
pnpm dev        # nuxt dev on http://localhost:3000
pnpm build      # nuxt build → .output
pnpm preview
```

- Runtime config (DB ×2, Redis, proxy, image base URLs, allowed iframe origins) is driven by `NUXT_*` env vars; see `.env.example`. For local dev set `NUXT_DEV_DOMAIN` to simulate a storefront domain.

### Docs (`v7-shop-docs`)

```bash
pnpm start      # docusaurus dev (or `yarn start`)
pnpm build
```

## Backend architecture (`v7-shop-services`)

Gradle multi-module with a strict dependency direction (`settings.gradle.kts`):

```
v7-soft-core      ← framework primitives (base entities, Result, exceptions, sa-token/JPA configurers, validators)
v7-shop-dao       ← JPA entities, repositories, multi-tenant + multi-datasource plumbing  (bootJar disabled)
v7-shop-common    ← shared services, Forest HTTP clients
v7-shop-account-service ┐
v7-shop-admin           ┘ ← feature modules (depend on core+dao+common)
v7-shop-entrance  ← composition root: depends on everything, holds the @SpringBootApplication, actuator, controllers
```

Root `build.gradle.kts` applies Spring Boot + dependency-management to `allprojects` and declares the shared dependency stack: **sa-token** (auth/JWT, with a dedicated Redis), **Forest** (declarative HTTP clients, scanned from `cn.v7soft.common.forest`), **Hutool**, **knife4j/springdoc** (OpenAPI), JPA + Redis + MySQL.

### Multi-tenancy and data scoping — read before touching DAO/queries

This is the most important non-obvious backend concept. Three independent ThreadLocal-based mechanisms layer together:

- **Company tenant** (`TenantContext` + `TenantIdentifierResolver`): Hibernate `CurrentTenantIdentifierResolver<Long>`. Tenant `-1` or `TenantContext.isSilent()` is treated as **root** (bypasses tenant filtering). Set per request.
- **Website scope** (`WebsiteContext`): ThreadLocal current-website-id + domain + an `isWebsiteAdmin` flag distinguishing storefront-admin pages from total-backend pages.
- **Data-range filter** (`AccessDataRangeContext` + `GlobalDataRangeFilterInterceptor`): a Hibernate `StatementInspector` (wired via `spring.jpa.properties.hibernate.session_factory.statement_inspector`) that rewrites SQL for row-level access control.

Two datasources are configured (`spring.datasource.primary` = shop DB, `spring.datasource.address` = a separate read DB of country/region/address data). JPA `hbm2ddl.auto=update` — entity changes auto-migrate the schema. `DatabaseInitializer.sql` seeds the schema.

## Mall architecture (`v7-shop-mall`)

Nuxt 4 with a substantial **Nitro server layer** (`server/`) that is the storefront's real backend — it queries MySQL/Redis directly (via `server/utils/db.ts`, `address-db.ts`, `redis.ts`) rather than calling the Java API.

- **Ordered middleware pipeline** (`server/middleware/NN-*.ts`) runs per request and builds a `PageContext`: `01-domain` (resolve host → domain/company, spuId cookie) → `02-cloak` (risk control) → `03-landing` → `04-protocol` → `05-pixels` → `10-product` / `11-article` / `13-order`. Numeric prefixes define execution order; `route-patterns.ts` decides which paths each one skips. When a request is invalid it short-circuits to a "safe page" (`utils/safe-page.ts`, e.g. SHOP_NOT_FOUND).
- `server/repositories/*` are the data-access layer; `server/services/*` hold business logic (e.g. `checkoutService`); `server/api/**` are the HTTP endpoints.
- **Frontend `app/components/blocks/`** is a CMS/page-builder block system: each block is a `.vue` + a `.meta.ts` descriptor, grouped into `basic` / `business` / `layout`. Blocks are globally registered with a `Block` prefix (see `nuxt.config.ts`). The admin SPA embeds the mall's `/builder` route in an iframe for visual editing — `VITE_NUXT_BUILDER_URL` / `allowedOrigins` coordinate the two apps.

## Admin architecture (`v7-shop-admin`)

Standard `shop-vite` (admin-plus lineage) Vue 3 SPA. `src/api/*.ts` mirror backend resources one-to-one (product, sku, spu, order, domain, ssl, ai\*, company, department, etc.) — when adding a backend endpoint, add the matching `api` module here. Views are grouped by domain under `src/views/` (ai, article, domain, order, product, ssl, system, website…). Auto-import is enabled (`unplugin-auto-import`, `unplugin-vue-components`) so Vue/router/store APIs and components are available without explicit imports. The app supports **white-label brand builds** — brand-specific behavior is selected by build mode (`ht`, `xyz`) and env files, not by runtime config.

## CI / release

GitHub Actions build on **tag push only**, with per-app tag prefixes, and upload artifacts to Backblaze B2:

- `ADMIN-V*` → builds admin in `ht` and `xyz` modes → `admin/<mode>/...zip` (`.github/workflows/admin-build.yml`).
- `MALL-V*` → builds mall, zips `.output` → `mall/...zip` (`.github/workflows/mall-build.yml`).

## Conventions

- Commit messages and code comments are predominantly Chinese; match the surrounding style.
- Backend package root is `cn.v7soft`; Gradle group `cn.v7soft`, version pinned to Java 17.
- `.cursor/` is Cursor-only — do not read or edit it. `.idea/`, `.vscode/` are IDE config and ignored.

---
> Source: [Dawnbells/v7-shop-repositories](https://github.com/Dawnbells/v7-shop-repositories) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
