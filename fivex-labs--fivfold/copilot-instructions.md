## fivfold

> **CRITICAL SYSTEM DIRECTIVE FOR AI AGENTS**:

# FivFold CLI Refactoring & Architectural Rule Set (AGENTS.md)

**CRITICAL SYSTEM DIRECTIVE FOR AI AGENTS**:
You are tasked with refactoring and extending the FivFold CLI. FivFold exists as two CLIs (`@fivfold/ui` and `@fivfold/api`) built on a shared engine (`@fivfold/core`). The core already implements VFS, StrategyPipeline, manifests, and AST (ts-morph). Your objective is to extend this into a highly advanced, extensible, multi-stack scaffolding engine as defined below.

You must strictly adhere to these rules. Any code generated that violates these constraints is considered a failure.

## 0. Strict Tech Stack Constraints

Do not use legacy versions of these tools. The project requires:

- **Node.js:** v20 or later.
- **Frontend:** React 18+ or Next.js 14+ (App Router).
- **Styling:** Tailwind CSS **v4 exclusively** (using `@import "tailwindcss"`; NO `tailwind.config.js` is permitted).
- **UI Foundation:** shadcn/ui.

## 1. File Modification Rules (The "Hybrid" Strategy)

**NEVER** use Regular Expressions or string `.replace()` functions to mutate existing source code files (like `app.module.ts` or `server.ts`). This is strictly forbidden due to fragility.

- **For EXISTING files:** You MUST use Abstract Syntax Tree (AST) manipulation via `ts-morph`. You must parse the AST, safely inject the node, and serialize it back to ensure surgical precision and avoid destroying user modifications.
- **For NEW, standalone files:** You may use string-based templating (e.g., Handlebars).

## 2. State & Transaction Rules (Virtual File System)

**NEVER** write directly to the physical disk (using `fs.writeFileSync`) in the middle of a generation process.

- **VFS Mandate:** You must implement a Virtual File System (VFS). All file creations, AST modifications, and deletions must be staged in memory first.
- **Atomic Commits:** Only after all generators and AST mutations complete successfully should the VFS flush changes to the disk in a single, atomic transaction.
- **Side Effects:** Actions like `npm install` or running formatting tools must only occur *after* the VFS successfully commits to the disk.
- **Dry Run:** The system must support a `--dry-run` flag to output intended changes without executing them.

**Current implementation:** `@fivfold/core` provides `VirtualFileSystem` with `stageCreate`, `stageModify`, `stageDelete`, `preview`, and `commit`.

## 3. Anti-Combinatorial Explosion Rules

**NEVER** create hardcoded directories for every permutation of a stack (e.g., no `express-typeorm-firebase` folders).

- **Strategy Pattern:** Implement interchangeable classes for code generation (e.g., `NestJsGeneratorStrategy`, `MongooseOrmStrategy`) instead of massive `if/else` statements.
- **Manifests over Hardcoding:** Kits must be defined by declarative JSON/YAML schemas (Manifests) specifying dependencies, remote template URLs, and AST mutation targets. The CLI must act as an agnostic orchestrator that reads these manifests.
- **ORM-Agnostic Framework Layer:** The `framework` manifest layer (Express/NestJS files and deps) must **never** hardcode ORM-specific dependencies (e.g., `typeorm`, `pg`, `reflect-metadata`). Those belong exclusively in the `orm` manifest layer, supplied by the active ORM strategy.
- **Handlebars Conditionals for ORM Imports:** Framework templates (`*.service.ts.hbs`, `*.module.ts.hbs`) must use `{{#if (eq orm "typeorm")}}` / `{{#if (eq orm "mongoose")}}` / etc. conditionals for ORM-specific imports, decorators, and dependency injection patterns.

**Current implementation:** `@fivfold/core` provides `StrategyPipeline`, `IGeneratorStrategy`, `IRealtimeStrategy`, and registry. API package uses `DomainStrategy`, `TypeOrmOrmStrategy`, `PrismaOrmStrategy`, `MongooseOrmStrategy`, `CosmosOrmStrategy`, `DynamoDbOrmStrategy`, `NestJsFrameworkStrategy`, `ExpressFrameworkStrategy`, `PushProviderStrategy`, `RealtimeProviderStrategy`. Manifests live in `ui/manifests/` and `api/manifests/`.

## 4. Output Architecture Rules (Hexagonal/Ports & Adapters)

When scaffolding backend code (like an Auth Kit), the outputted code **MUST** adhere to Hexagonal Architecture to prevent vendor lock-in.

- **Domain (Core):** Must generate framework-agnostic Interfaces (Ports) (e.g., `IAuthService`, `IChatService`).
- **Infrastructure (Adapters):** Must generate separate files implementing the Interfaces using vendor SDKs (e.g., `FirebaseAuthAdapter`, `MongooseChatAdapter`).
- **Delivery:** HTTP transport (Express Routes or NestJS Controllers) must be isolated from the core logic.

### Auth Provider Independence

Auth provider selection (Firebase Auth, Cognito, Auth0, JWT) is **completely independent** of database/ORM selection. A user may choose Firebase Authentication as their auth provider while storing application data in PostgreSQL with TypeORM, MongoDB with Mongoose, or any other supported combination. The auth provider handles identity verification only; all application data (user profiles, sessions, kit-specific data) lives in the user's chosen database.

**Never assume that selecting a Firebase service (Auth, FCM) implies Firebase as the database. Each concern is orthogonal.** The `GeneratorContext` separates `authProvider` from `orm` and `database`. The prompt flow must never auto-switch or constrain the database/ORM choice based on auth provider selection.

## 5. Plugin Architecture Rules

Currently, FivFold uses two separate packages (`@fivfold/ui` and `@fivfold/api`). You must prepare the architecture for a unified, language-agnostic future (supporting .NET Core).

- **Orchestrator:** The core CLI must handle terminal input, VFS, and manifest resolution.
- **Plugins:** All language-specific logic (like `ts-morph` AST parsing) must be encapsulated in isolated plugins (e.g., `@fivfold/plugin-node`).

## 6. Developer Experience & Automation Rules

- **Sequential Deduction:** When prompting the user, deduce the stack sequentially — never present incompatible options together:
  1. Runtime (Node.js — currently the only supported runtime)
  2. Framework (Express or NestJS)
  3. Database Category (RDS / NoSQL)
  4. Database (filtered by category)
  5. ORM (auto-filtered by database; some are auto-selected with no prompt)
  6. Kit-specific providers: Auth Provider, Push Provider (only shown when the kit uses them)
- **Auto-Detection:** The CLI must attempt to parse `package.json` to detect existing frameworks (e.g., `@nestjs/core`, `mongoose`) and silently skip relevant prompts.
- **CI/CD Ready:** The CLI **MUST NOT** force interactive prompts. Every prompt must map to a CLI flag, and a `--yes` or `-y` flag must instantly bypass all questions, utilizing smart defaults (e.g., defaulting to Tailwind/shadcn for UI, TypeScript for NestJS, RDS/PostgreSQL/TypeORM for API).

**Current implementation:** `@fivfold/core` provides `detectStack`, `parseGlobalFlags`, `getSmartDefaults`, and interactive prompts (`selectFramework`, `selectDatabaseCategory`, `selectDatabase`, `selectOrm`, etc.). `selectRealtimeProvider` is kept for multi-provider kits but Socket.IO is auto-selected when it is the only option.

## 7. Monorepo Context

The FivFold monorepo has four packages:

| Package | Purpose |
|---------|---------|
| `@fivfold/core` | Shared engine: VFS, StrategyPipeline, manifests, TemplateEngine, TsMorphEngine, detection, prompts, workspace |
| `@fivfold/ui` | Frontend Kits CLI: init, add, list, agents, setup |
| `@fivfold/api` | Backend scaffolding CLI: init, add, list |
| `fivfold-site` | Next.js docs site |

**Workspace dependencies:** `ui` and `api` depend on `core` via `workspace:*`. `site` depends on `ui` (devDependency) for demos.

**Discovering Kits:** Do not treat any static doc table as authoritative. Use each CLI’s `list` command and inspect `ui/manifests/*.kit.json` and `api/manifests/*.kit.json` for what is shipped in this revision.

**Where to make changes:**

- **VFS, strategy, manifest, template, AST, detection, prompt, workspace:** `core/src/`
- **UI CLI commands, manifests, templates:** `ui/src/`, `ui/manifests/`, `ui/templates/`
- **API CLI commands, strategies, manifests, templates:** `api/src/`, `api/manifests/`, `api/templates/`
- **Docs site:** `site/app/`

**Config:** `fivfold.json` at project root; shared by ui and api.

## 8. Documentation Conventions

- **Getting Started docs** (`site/app/docs/getting-started/`): Keep language generic. Do not name specific Kits, ORMs, or frameworks except in short examples.
- **Kit docs** (`site/app/docs/kits/`): Per-Kit pages may be detailed and stack-specific where needed.
- **API docs** (`site/app/docs/api/`): Framework-specific content (e.g. Express vs NestJS) is appropriate.

### Kit documentation page structure (mandatory)

Kit pages under `site/app/docs/kits/<kit>/` use **UI** and **API** tabs (or a single tab for backend-only kits). **Always** present sections in this order. Use numbered headings (`KitDocStepHeading` in the site) so readers can scan consistently. **Skip** a step only when it truly does not apply (e.g. no third-party section).

**UI tab**

1. **Installation** — CLI command (`npx @fivfold/ui add …`), `init` prerequisite, output path.
2. **Generated file structure** — tree of scaffolded files with short comments.
3. **Import and usage** — minimal working example (import + render).
4. **Props reference** — tables or grouped lists for main components.
5. **Integration with backend** — disclaimers (wiring is yours), proxy/rewrites, env vars, CORS hints, user/session alignment; include `KitFeBeConnectionGuide` / full-stack helpers where used.
6. **Third-party integrations** — only if the kit uses external APIs (e.g. Tenor, maps). Omit entirely if none.
7. **shadcn/ui primitives** — table of components the kit installs or expects.
8. **Additional** — extra npm deps, drag-and-drop libs, notes that do not fit above.

**API tab**

1. **Installation / scaffold** — `npx @fivfold/api add …` and flags.
2. **Generated file structure** — tree per stack variant (domain, adapters, delivery).
3. **Wire into the app** — register entities, mount routes/modules, middleware order (stack-specific substeps OK).
4. **API reference** — endpoints table, DTOs, or key routes (analogous to “props” on the UI side).
5. **Integration with frontend** — disclaimers, CORS, env vars, how the browser reaches the API, linking generated entities to your user model (`KitApiFeBePlaybook`, `KitUserModelIntegration` as appropriate).
6. **Third-party** — provider SDKs (e.g. FCM, Firebase Admin) when not covered in §5.
7. **shadcn** — omit on API tabs (not applicable).
8. **Additional** — migrations-only notes, security checklist, error codes, etc.

When editing kit docs, **reorder existing content** to match this sequence rather than appending new sections at the end.

## 9. Supported Databases, ORMs & Services

### Database Categories

FivFold organizes databases into two categories. The CLI prompt flow selects category first, then filters all subsequent options accordingly.

| Category | Databases |
|----------|-----------|
| **RDS (SQL)** | PostgreSQL, MySQL, MS-SQL, MariaDB |
| **NoSQL** | MongoDB, Azure Cosmos DB, AWS DynamoDB |

> **Note:** Firebase Cloud Firestore and Firebase Realtime Database are **not** supported as databases. Firebase Authentication and Firebase Cloud Messaging (FCM) remain supported as service providers (auth and push respectively) and are completely orthogonal to the database choice.

### ORM / Data Access Layer

ORM selection is **always dependent on the database choice**. Never present ORM options that are incompatible with the selected database.

| Database | Compatible ORMs / SDKs | Auto-selected? |
|----------|------------------------|----------------|
| PostgreSQL | TypeORM, Prisma | No (prompt) |
| MySQL | TypeORM, Prisma | No (prompt) |
| MS-SQL | TypeORM, Prisma | No (prompt) |
| MariaDB | TypeORM, Prisma | No (prompt) |
| MongoDB | Mongoose, Prisma (MongoDB connector) | No (prompt) |
| Azure Cosmos DB | `@azure/cosmos` SDK | Yes (auto) |
| AWS DynamoDB | `@aws-sdk/client-dynamodb` SDK | Yes (auto) |

When the ORM is auto-selected, skip the ORM prompt entirely and inform the user of the selection.

### Realtime (Kit-specific)

Kits that require real-time communication use **Socket.IO exclusively**. It is auto-selected with no prompt. There is no user-facing realtime provider choice.

| Provider | Transport | Notes |
|----------|-----------|-------|
| Socket.IO | WebSocket | Works with any database; sole supported real-time provider |

### Service Providers

Service providers are orthogonal to database/ORM and are prompted per-kit when relevant:

| Category | Providers |
|----------|-----------|
| **Auth** | Firebase Auth, AWS Cognito, Auth0, Custom JWT |
| **Push Notifications** | Firebase Cloud Messaging (FCM), OneSignal, AWS SNS, Pushy, Pusher Beams |
| **GIF** | Tenor (default), extensible via provider interface |
| **Maps/Location** | Browser Geolocation API (frontend), configurable static map provider |

### Dependency Ownership Rules

ORM-specific npm packages must only appear in the `orm` manifest layer dependencies, never in `framework` layer dependencies:

| Belongs in `orm` layer | Belongs in `framework` layer |
|------------------------|------------------------------|
| `typeorm`, `@nestjs/typeorm` | `express`, `@nestjs/common`, `@nestjs/core` |
| `pg`, `mysql2`, `mssql`, `mariadb` | `class-validator`, `class-transformer` |
| `mongoose`, `@nestjs/mongoose` | `uuid`, `@types/uuid` |
| `@prisma/client`, `prisma` | `@types/express`, `@types/node` |
| `@azure/cosmos` | `reflect-metadata` (only when using TypeORM) |
| `@aws-sdk/client-dynamodb` | |

## 10. Agent Layout Rules

When generating or modifying code, agents MUST follow the established layout:

- **Backend layout:** Domain (ports) → Infrastructure (adapters) → Delivery (controllers/routes). Never mix ORM-specific code in the framework layer.
- **ORM conditionals:** Use Handlebars `{{#if (eq orm "typeorm")}}` / `{{#if (eq orm "mongoose")}}` etc. in framework templates; ORM dependencies belong exclusively in the `orm` manifest layer.
- **File structure:** UI Kit templates under `ui/templates/` (and manifests in `ui/manifests/`); API output under the user’s configured path; entities, DTOs, services, controllers follow Hexagonal layout.
- **Manifest-driven:** All kit/module definitions come from manifests; no hardcoded stack permutations.

## 11. Skills Registry Maintenance (MANDATORY)

When developing FivFold, agents and AI assistants MUST update `skills.md` whenever:

- A new kit is added (UI or API)
- A kit is removed or renamed
- Component structure changes (e.g. new exports, new paths)
- A new installable skill is created
- Existing skill descriptions or paths change

Update the relevant table in `skills.md` and ensure the `skills` CLI command can discover and install the new or changed skill.

---
> Source: [Fivex-Labs/fivfold](https://github.com/Fivex-Labs/fivfold) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
