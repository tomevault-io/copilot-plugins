## phos

> Full-stack interactive project generator CLI that scaffolds modern web applications with configurable backends and frontends.

# AGENTS Guidelines for This Repository

## Project Name: Phos CLI

Full-stack interactive project generator CLI that scaffolds modern web applications with configurable backends and frontends.

**Version:** 1.3.0 (Production Release)

### Core Features:

1. **Interactive CLI** - Beautiful prompts using @clack/prompts
2. **Multiple Backends** - Elysia (Bun), FastAPI (Python), NestJS (Node.js)
3. **Multiple Frontends** - Astro, Svelte, Next.js, Vue
4. **Monorepo Support** - Workspace configuration for scalable projects
5. **Configurable Tooling** - TypeScript, ESLint, Prettier options
6. **CSS Frameworks** - Tailwind, SCSS, CSS Modules
7. **UI Components** - Framework-specific libraries (see below)
8. **Testing Support** - Vitest, Playwright

### Technology Stack:

- **CLI**: Node.js + TypeScript
- **CLI Framework**: Commander.js
- **Interactive Prompts**: @clack/prompts
- **File Operations**: fs-extra
- **Template Engine**: Handlebars
- **Terminal Colors**: picocolors
- **Backend**: Elysia (Bun), FastAPI (Python), NestJS (Node.js)
- **Frontend**: Astro, SvelteKit, Next.js, Vue
- **Package Managers**: npm, yarn, pnpm, bun

## 1. Development Workflow

### CLI Tool (Phos)

- Use Commander.js for CLI argument parsing
- Use @clack/prompts for beautiful interactive UI
- Use fs-extra for file system operations
- Use Handlebars for template rendering
- Use picocolors for terminal colors

## 2. Architecture

### Core Components

```
src/
├── cli.ts                  # Main CLI entry point
├── generators/
│   ├── monorepo.ts         # Monorepo scaffold generator
│   ├── single.ts           # Single project generator
│   ├── backends/
│   │   ├── elysia.ts       # Elysia (Bun) backend generator
│   │   ├── fastapi.ts      # FastAPI (Python) backend generator
│   │   └── nestjs.ts      # NestJS (Node.js) backend generator
│   └── frontends/
│       ├── astro.ts        # Astro frontend generator
│       ├── svelte.ts       # Svelte frontend generator
│       ├── nextjs.ts       # Next.js frontend generator
│       └── vue.ts         # Vue frontend generator
├── templates/              # Project templates
└── utils/
    └── helpers.ts          # Helper functions
```

### File Naming Conventions

| Layer        | Pattern          | Example                 |
| ------------ | ---------------- | ----------------------- |
| CLI Commands | `*.ts`           | `cli.ts`, `monorepo.ts` |
| Generators   | `{framework}.ts` | `elysia.ts`, `astro.ts` |
| Utils        | `*.ts`           | `helpers.ts`            |

**Rules:**

- Use **PascalCase** for generator files
- Use **camelCase** for utility functions
- Export generators as named exports

### Backend API Architecture

Backend projects follow a clean layered architecture:

```
src/
├── api/                   # API routers (Elysia) or endpoints (FastAPI)
│   └── user_api.ts       # User API router with validation
├── service/               # Business logic layer
│   └── user_service.ts   # User service with business rules
├── sql/                   # Database query layer
│   └── user_sql.ts       # User SQL query functions
├── types/                 # Type definitions
│   └── user_type.ts       # User entity and DTOs
└── function/              # Helper functions
    └── helper.ts         # Validation helpers
```

**Data Flow:**
```
API Layer (validation) → Service Layer (business logic) → SQL Layer (queries) → Database
```

**Responsibilities:**
- **API Layer**: Request validation, error handling, routing
- **Service Layer**: Business logic, uniqueness checks, data mapping
- **SQL Layer**: Database queries, password hashing, CRUD operations
- **Types Layer**: Entity definitions, DTOs, response models
- **Function Layer**: Reusable validation and helper functions

## 3. Keep Dependencies in Sync

When adding/updating packages:

```bash
bun install <package>
# OR
npm install <package>
```

## 4. Coding Conventions

### TypeScript

- Use modern TypeScript with strict mode enabled
- Import using `@/` alias for internal modules
- Use ES modules with `.js` extensions

### Project Templates

- Use Handlebars for dynamic content
- Include TypeScript/ESLint/Prettier options as needed
- Generate complete project structures

## 5. Key Technologies

### CLI Framework

- **Commander.js** - Command-line interface framework
- **@clack/prompts** - Interactive prompts
- **fs-extra** - Enhanced file system operations
- **Handlebars** - Template engine
- **picocolors** - Terminal colors

### Supported Backends

- **Elysia** - Fast and elegant Bun web framework
- **FastAPI** - Modern Python web framework
- **NestJS** - Progressive Node.js framework with TypeScript

### Supported Frontends

- **Astro** - Modern static site generator
- **SvelteKit** - Full-stack Svelte framework
- **Next.js** - React framework with server components
- **Vue** - Progressive JavaScript framework

## 6. UI Component Libraries by Framework

### Next.js UI Libraries
| Library | Description | Recommended |
|----------|-------------|-------------|
| **Ant Design** | Enterprise-class design system with 50+ components | ✅ Yes |
| **shadcn/ui** | Customizable, Radix UI + Tailwind CSS | No |
| **Radix UI** | Unstyled, accessible primitives | No |
| **Material UI (MUI)** | Google Material Design, extensive ecosystem | No |
| **HeroUI** | Beautiful, fast, Next.js App Router compatible | No |

### Vue UI Libraries
| Library | Description | Recommended |
|----------|-------------|-------------|
| **Vuetify** | Material Design, 80+ components, Vue 3 native | ✅ Yes |
| **Element Plus** | Enterprise-focused, successor to Element UI | No |
| **Quasar** | Full framework, 70+ components, cross-platform | No |
| **PrimeVue** | 80+ components, data tables, calendars | No |
| **Naive UI** | Vue 3 focused, great developer experience | No |

### Astro UI Libraries
| Library | Description | Recommended |
|----------|-------------|-------------|
| **daisyUI** | Tailwind CSS plugin, zero JS, 35+ themes | ✅ Yes |
| **fulldev/ui** | Vanilla Astro, shadcn/ui compatible | No |
| **shadcn/ui** | Works via React integration | No |

### Svelte UI Libraries
| Library | Description | Recommended |
|----------|-------------|-------------|
| **shadcn-svelte** | 1:1 port of shadcn/ui, highly customizable | ✅ Yes |
| **Svelte Material UI (SMUI)** | Material Design, 40+ components | No |
| **Svelte Headless UI** | Unstyled, accessible primitives | No |
| **Agnostic UI** | Cross-framework, works with React, Vue, Angular | No |
  
  ## 7. Configuration Options

### Backend Options

- **TypeScript** - Enable/disable TypeScript
- **ESLint** - Add ESLint configuration
- **Prettier** - Add Prettier configuration

### Frontend Options
- **TypeScript** - Enable/disable TypeScript
- **ESLint** - Add ESLint configuration
- **Prettier** - Add Prettier configuration
- **CSS Framework** - Tailwind, SCSS, CSS Modules
- **UI Components** - Framework-specific libraries (see section 6)
- **Testing** - Vitest, Playwright, or both

### NestJS Backend Options

- **TypeORM** - PostgreSQL database with TypeORM
- **Swagger** - API documentation with Swagger UI
- **JWT Auth** - JWT authentication with passport-jwt
- **Validation** - Request validation with class-validator
- **Password Hashing** - bcrypt for secure password storage

## 8. Available Commands

| Command                   | Purpose                        |
| ------------------------- | ------------------------------ |
| `bun run dev create`      | Run CLI in development mode    |
| `bun run build`           | Build TypeScript to JavaScript |
| `bun run start`           | Run built CLI                  |
| `node dist/cli.js create` | Run CLI directly               |
| `bun link`                | Link CLI globally              |
| `phos create`             | Run CLI (after linking)        |

## 9. Versioning

Phos CLI follows semantic versioning:

- **x.y.z (Patch/Bugfix)**: `0.0.x`, `1.0.x`, `2.0.x`
  - Bug fixes
  - Documentation updates
  - Small improvements

- **x.y.0 (Minor/Feature)**: `0.x.0`, `1.x.0`, `2.x.0`
  - New features
  - Breaking changes (when < 1.0.0)
  - Significant updates

- **x.0.0 (Major/Milestone)**: `1.0.0`, `2.0.0`, `3.0.0`
  - Major architectural changes
  - Complete rewrites
  - Production-ready releases

**Current Version: 1.3.1** - Production Release

See [CHANGELOG.md](CHANGELOG.md) for detailed version history.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DotJumpDot)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/DotJumpDot)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
