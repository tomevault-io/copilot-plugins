## thommf-portfolio

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Development

```bash
# Start development server with all micro-frontends
pnpm dev

# Build all applications and libraries
pnpm build

# Build for production (excludes e2e tests)
pnpm build:prod

# Run all tests
pnpm test

# Run end-to-end tests
pnpm e2e

# Run e2e tests with UI
pnpm e2e:dev

# Lint all projects
pnpm lint

# Format code
pnpm format

# Type check
nx run-many --target=typecheck --all
```

### Nx Commands for Specific Projects

```bash
# Run specific target for a project
nx run <project>:<target>

# Serve specific micro-frontend
nx serve headerMfe
nx serve projectListMfe
nx serve container --dev-remotes="headerMfe,projectListMfe"

# Test specific project
nx test <project-name>

# Run affected commands (based on git changes)
nx affected -t lint
nx affected -t test
nx affected -t build
```

### Deployment

```bash
# Full deployment (build + infrastructure + upload)
pnpm run deploy

# Deploy infrastructure only
pnpm run deploy:infra

# Build only
pnpm run deploy:build

# Upload to S3 only
pnpm run deploy:upload
```

### Terraform Operations

```bash
# Initialize Terraform
pnpm run tf:init

# Plan infrastructure changes
pnpm run tf:plan

# Apply infrastructure changes
pnpm run tf:apply
```

## Architecture

This is a **micro-frontend portfolio application** built with:

- **Nx monorepo** for workspace management
- **Module Federation** for runtime composition of micro-frontends
- **React 19** with TypeScript

### Applications Structure

- **`apps/container/`** - Shell application that orchestrates micro-frontends

  - Loads and composes `headerMfe` and `projectListMfe` at runtime
  - Contains routing and overall layout
  - Module federation host configuration in `module-federation.config.ts`

- **`apps/headerMfe/`** - Header micro-frontend

  - Exposed as remote module via Module Federation
  - Self-contained with own webpack config

- **`apps/projectListMfe/`** - Project list micro-frontend
  - Displays portfolio projects
  - Exposed as remote module

### Shared Libraries

- **`libs/shared/`** - Common utilities and configurations

  - Shared between all micro-frontends
  - Contains shared TypeScript configs and utilities

- **`libs/store/`** - Redux store configuration

  - Shared state management using Redux Toolkit

- **`libs/portfolio-store/`** - Portfolio-specific Redux slices
  - Domain-specific state management

### Module Federation Setup

Each micro-frontend has:

- `module-federation.config.ts` - Defines exposed modules and shared dependencies
- `webpack.config.ts` - Development webpack configuration
- `webpack.config.prod.ts` - Production webpack configuration

The container orchestrates remotes defined in its `module-federation.config.ts`:

```typescript
remotes: ['headerMfe', 'projectListMfe'];
```

Shared dependencies are managed via `createSharedConfig()` to ensure singleton instances.

### Infrastructure

AWS deployment using Terraform:

- **S3** for static hosting
- **CloudFront** CDN with OAC (Origin Access Control)
- **Route 53** for DNS (optional)
- Tag-based access control (`Environment=frontend`)
- Least-privilege IAM policies

Key infrastructure modules in `infra/terraform/modules/`:

- `s3/` - S3 bucket with versioning and encryption
- `cloudfront/` - CDN configuration with OAC
- `iam/` - IAM roles and policies

### Testing Strategy

- **Unit Tests**: Jest with React Testing Library
- **E2E Tests**: Playwright for each micro-frontend
- Test projects follow pattern: `<app-name>-e2e/`

### Development Workflow

1. The development script (`scripts/dev-with-mfe-tags.sh`) automatically detects all micro-frontends
2. Nx serves the container with development remotes for hot reloading
3. Each micro-frontend can be developed independently
4. Changes are composed at runtime through Module Federation

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tigredonorte) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
